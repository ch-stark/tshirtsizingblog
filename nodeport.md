# Resolving Node Exporter Port Conflicts on EKS with ACM Observability (working in ACM 2.17)

## The Problem

When deploying RHACM Observability on EKS clusters, the ACM-managed Node Exporter fails to start because port **9100** is already in use. EKS clusters commonly run their own Node Exporter instance on the default port, and ACM's observability addon deploys a second one that binds to the same port -- causing pod scheduling failures and `CrashLoopBackOff` errors.

```
Error: listen tcp :9100: bind: address already in use
```

This affects any non-OpenShift managed cluster where Node Exporter is already deployed as part of the existing monitoring stack (EKS, AKS, GKE, or vanilla Kubernetes clusters with Prometheus pre-installed).

## The Solution

PR [stolostron/multicluster-observability-addon#439](https://github.com/stolostron/multicluster-observability-addon/pull/439) introduces a new `nodeExporterPort` configuration variable that allows you to change the internal Node Exporter listen port independently from the kube-rbac-proxy host port.

### How to Configure

Set the custom port via the `AddOnDeploymentConfig` on the hub cluster:

```yaml
apiVersion: addon.open-cluster-management.io/v1alpha1
kind: AddOnDeploymentConfig
metadata:
  name: observability-addon-config
  namespace: open-cluster-management
spec:
  customizedVariables:
    - name: nodeExporterPort
      value: "9101"
```

This updates the Node Exporter DaemonSet on the managed cluster to listen on port 9101 instead of 9100, avoiding the conflict with the existing Node Exporter.

### What Changed

- New `nodeExporterPort` custom variable in the observability addon config
- The Node Exporter DaemonSet `--web.listen-address` and upstream flags now use the configurable port
- Port validation with strict bounds checking (1-65535)
- The kube-rbac-proxy host port remains independent, so only the internal listener is affected

### Key Details

- **Default behavior is unchanged** -- if you don't set `nodeExporterPort`, it defaults to 9100 as before
- **Validation** ensures the port value is within the valid range
- The fix ships as part of the multicluster-observability-addon, no changes needed on managed clusters beyond the hub config
