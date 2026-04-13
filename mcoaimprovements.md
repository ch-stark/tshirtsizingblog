# Elevating Fleet Observability: Unmatched Network Resiliency and Health Detection

The transition to the **MultiCluster Observability Addon (MCOA)** introduces critical operational improvements for enterprise fleet monitoring, specifically targeting network reliability and closing crucial blind spots in telemetry health.


## Improved Network Resiliency

MCOA drastically improves how metrics are handled during connectivity disruptions by utilizing a standard **Prometheus remote-write implementation**. 

At the core of this resiliency is the **Prometheus Agent**, which uses a local **Write Ahead Log (WAL)** to securely buffer federated metric samples on the managed cluster's disk. This architecture ensures:

* **Data Preservation:** If network connectivity is temporarily lost, your data is preserved locally until the connection is restored.
* **Buffer Capacity:** Provides robust resiliency against network partitions between managed clusters and the centralized hub for **up to two hours** without any data loss.


## Enhanced Cluster Health Detection
### Addressing the OCM Blind Spot

Historically, cluster health detection relied heavily on the **Open Cluster Management (OCM) lease model**. This created a potential observability blind spot: a metrics collector could actively renew its lease and report as "available" while silently failing to federate or remote-write metrics due to internal crashes, authentication failures, or network issues.

To solve this, the observability stack now includes precise, real-time alerting to detect when telemetry pipelines are silently failing:

1.  **ManagedClusterMetricsMissing:** This alert immediately flags when a managed cluster's observability add-on reports as available, but no metrics have reached the hub for **15 minutes**.
2.  **MetricsCollectorNotIngestingSamples:** Firing directly on the managed cluster when the agent fails to federate any metrics locally.
3.  **MetricsCollectorRemoteWriteFailures:** Firing during high remote-write request failure rates to the hub.


### Summary of Health Monitoring Improvements

| Feature | Legacy OCM Lease Model | MCOA Observability Stack |
| :--- | :--- | :--- |
| **Detection Method** | Heartbeat-based (Lease) | Metric-flow-based (Alerts) |
| **Failure Visibility** | Often "Green" even if data is missing | Immediate "Red" on telemetry failure |
| **Resiliency** | Limited local caching | 2-hour local WAL buffer |
| **Alerting Location** | Central Hub only | Hub-side & Managed Cluster-side |

Together, these enhancements guarantee that platform administrators are immediately notified of true telemetry health, completely eliminating **silent failures** in the observability pipeline.

> **Note:** The definition of the `ManagedClusterMetricsMissing` alert is based on provided query parameters, while supplementary alerts and resiliency mechanics are sourced from official MCOA documentation.
