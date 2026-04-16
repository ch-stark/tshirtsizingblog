# Resolving Node Exporter Port Conflicts on AKS with ACM Observability (working in ACM 2.17)

## The Problem

ACM's observability addon deploys a Node Exporter DaemonSet on managed clusters. To avoid clashing with the standard Node Exporter port (9100), ACM already defaults to host port **19100**. This works on most non-OpenShift providers.

However, the Node Exporter pod runs a kube-rbac-proxy sidecar, and both containers were configured to use the **same port number internally**. Some container runtimes -- notably the one used by **AKS** -- reject this and return a port clash error, preventing the pods from starting.

## The Fix

PR [stolostron/multicluster-observability-addon#439](https://github.com/stolostron/multicluster-observability-addon/pull/439) separates the Node Exporter listen port from the kube-rbac-proxy port, so each container now uses a different port. This fixes the AKS problem out of the box.

## Configuring a Custom Host Port

If the default host port 19100 is already reserved in your environment, you can change it via the `AddOnDeploymentConfig` on the hub cluster using the `nodeExporterHostPort` key:

```yaml
apiVersion: addon.open-cluster-management.io/v1alpha1
kind: AddOnDeploymentConfig
metadata:
  name: observability-addon-config
  namespace: open-cluster-management
spec:
  customizedVariables:
    - name: nodeExporterHostPort
      value: "19200"
```

This updates the Node Exporter DaemonSet on managed clusters to bind to host port 19200 instead of 19100.

### Key Details

- **No action needed for most users** -- the port separation fix works automatically
- **Default host port remains 19100** (not 9100) to avoid conflicts with standard Node Exporter
- Port validation with strict bounds checking (1-65535)
- The fix ships as part of the multicluster-observability-addon, no changes needed on managed clusters beyond the hub config
