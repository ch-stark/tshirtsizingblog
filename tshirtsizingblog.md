# Scaling RHACM Observability Made Simple: A Guide to T-Shirt Sizing
 
Managing an observability stack across a large hybrid cloud fleet is no small feat — especially when it comes to right-sizing backend components. Red Hat Advanced Cluster Management (RHACM) 2.16 addresses this directly by graduating **T-Shirt Sizing** for hub-side observability to **General Availability (GA)**.
 
 
## What Is T-Shirt Sizing?
 
T-Shirt Sizing provides an opinionated, out-of-the-box methodology for scaling hub-side observability components using a set of predefined size profiles. Rather than manually calculating resource requests, limits, and replica counts for individual components — such as Thanos, Grafana, Alertmanager, and the Observatorium API — you simply select the size that best matches your fleet's metric volume.
 
Each size pre-tunes these core components for optimal performance and efficiency, dramatically reducing the complexity of both initial setup and future scaling operations.
 
 
## Choosing the Right Size
 
Sizing is driven primarily by the **number of active time series** in your environment. You can find your current count in the **OCP Console → Metrics** tab.
 
| Size      | Active Time Series | Total CPU | Total Memory |
|-----------|--------------------|-----------|--------------|
| Default   | < 200k             | 3         | 12 GiB       |
| Minimal   | < 1 million        | 16        | 25 GiB       |
| Small     | 1 million          | 32        | 72 GiB       |
| Medium    | 5 million          | 55        | 137 GiB      |
| Large     | 10 million         | 103       | 293 GiB      |
| XLarge    | 20 million         | 163       | 590 GiB      |
| 2XLarge   | 50 million         | 222       | 1,019 GiB    |
| 4XLarge   | 100 million        | 337       | 2,158 GiB    |
 
> **Tip:** When in doubt, size up. It is always safer to over-provision slightly than to under-resource a component handling millions of active series.
 

## How to Configure T-Shirt Sizing
 
Applying a T-Shirt size requires a single field change in your `MultiClusterObservability` custom resource (CR) on the hub cluster:
 
```yaml
apiVersion: observability.open-cluster-management.io/v1beta2
kind: MultiClusterObservability
metadata:
  name: observability
spec:
  instanceSize: large  # Replace with your desired size: minimal, small, medium, large, xlarge, 2xlarge, 4xlarge
```
 
Apply the change with:
 
```bash
oc apply -f multiclusterobservability.yaml
```
 
RHACM will reconcile the resource configuration across the relevant components automatically.
 
##  Changing Your Instance Size

A common question when adopting this feature is whether you can easily move from one instance size to another.
The system fully supports *changing* your `instanceSize` to any available value at any time, and does not block you from moving in either direction.
**Scaling Up**: Moving from a lower to a higher size (e.g., small to large) works seamlessly and is the expected path as your fleet and metric volume naturally grow.
**Scaling Down**: Moving from a higher size to a lower size is also entirely possible. However, while it is permitted, it is not recommended based on load. You should only scale down if you have actively reduced your active time series footprint; otherwise, the lower resource limits could cause observability components to crash or struggle under the existing workload.

 
## Further Important Considerations
 
Before rolling out T-Shirt Sizing, keep the following in mind:
 
**Customizations take precedence.**
Any manually defined advanced configurations — such as explicit resource requests or limits for a specific Thanos component — will override the T-Shirt size settings for that component. Review your existing `MultiClusterObservability` CR carefully before enabling a size profile to avoid unexpected conflicts.
 
**Sizing is reactive, not predictive.**
The feature scales based on your *current* workload. It cannot forecast future time series growth. You are responsible for monitoring fleet growth and adjusting `instanceSize` proactively as your active series count increases.
 
 
## Summary
 
**T-Shirt Sizing** eliminates the guesswork from RHACM Observability resource provisioning. By mapping your fleet's metric volume to a predefined size profile, platform teams can ensure their centralized observability stack remains highly available, properly tuned, and performant — without deep expertise in every underlying component's resource model.
 
As your fleet grows, revisit your active time series count regularly and adjust the `instanceSize` field accordingly. It really is that simple.
