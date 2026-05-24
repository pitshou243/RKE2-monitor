# RKE2-monitor
## Overview

The RKE2 Monitor is a lightweight Prometheus-based monitoring package designed to proactively detect instability in RKE2 clusters before it becomes a full outage.
The goal is to:

- Detect degradation early
- Surface actionable symptoms
- Reduce time-to-detection during incidents
- Provide consistent operational visibility across RKE2 clusters

This project ships primarily as a set of:

- PrometheusRule alert rules

## Why This Exists

Many RKE2 incidents follow recognizable failure patterns before the cluster becomes unavailable:
| Failure Pattern                 | Common Symptoms                          |
| ------------------------------- | ---------------------------------------- |
| kube-apiserver degradation      | Slow kubectl, 5xx errors, probe failures |
| kubelet instability             | Pods stuck Pending or ContainerCreating  |
| Node pressure                   | DiskPressure, MemoryPressure, IO wait    |
| Runtime degradation             | containerd errors, pod startup delays    |
| DNS failures                    | CoreDNS crashloops, SERVFAIL spikes      |
| CNI instability                 | Canal/Calico/Cilium pods unhealthy       |
| Certificate expiration          | kubelet auth failures                    |
| Disk exhaustion                 | etcd instability, image pull failures    |
| Control plane component failure | Scheduler/controller-manager unavailable |

Most environments already expose these metrics through Prometheus — the gap is usually alerting and operational visibility.
## What This Collects

The RKE2 monitor evaluates metrics from:
| Component               | Metrics Source                   |
| ----------------------- | -------------------------------- |
| kube-apiserver          | Kubernetes control plane metrics |
| kube-scheduler          | Kubernetes control plane metrics |
| kube-controller-manager | Kubernetes control plane metrics |
| kubelet                 | kubelet metrics                  |
| containerd runtime      | kubelet runtime metrics          |
| Node health             | node-exporter                    |
| Kubernetes objects      | kube-state-metrics               |
| DNS                     | CoreDNS metrics                  |
| CNI components          | kube-state-metrics               |
| Certificates            | kubelet certificate metrics      |

## Included Alerts
### Control Plane Health
| Alert                            | Purpose                           |
| -------------------------------- | --------------------------------- |
| `RKE2KubeAPIServerDown`          | Detect API server outages         |
| `RKE2KubeAPIServerHighErrorRate` | Detect elevated API failures      |
| `RKE2KubeAPIServerHighLatency`   | Detect API slowness               |
| `RKE2KubeControllerManagerDown`  | Detect controller-manager failure |
| `RKE2KubeSchedulerDown`          | Detect scheduler failure          |
### Node Health
| Alert                          | Purpose                    |
| ------------------------------ | -------------------------- |
| `RKE2NodeNotReady`             | Detect unavailable nodes   |
| `RKE2NodeMemoryPressure`       | Detect memory exhaustion   |
| `RKE2NodeDiskPressure`         | Detect disk pressure       |
| `RKE2NodeFilesystemAlmostFull` | Detect low disk space      |
| `RKE2NodeHighIOWait`           | Detect storage bottlenecks |
### Kubelet & Runtime Health
| Alert                        | Purpose                              |
| ---------------------------- | ------------------------------------ |
| `RKE2KubeletDown`            | Detect kubelet outages               |
| `RKE2ContainerRuntimeErrors` | Detect runtime/containerd issues     |
| `RKE2PodStartupLatencyHigh`  | Detect slow pod lifecycle operations |
### Networking & DNS
| Alert                       | Purpose                   |
| --------------------------- | ------------------------- |
| `RKE2CNIPluginPodsNotReady` | Detect unhealthy CNI pods |
| `RKE2CoreDNSNotReady`       | Detect DNS outages        |
| `RKE2CoreDNSHighErrorRate`  | Detect DNS instability    |
### Workload Stability
| Alert                        | Purpose                            |
| ---------------------------- | ---------------------------------- |
| `RKE2SystemPodsCrashLooping` | Detect repeated crashes            |
| `RKE2SystemPodPending`       | Detect scheduling/runtime stalls   |
| `RKE2SystemPodFailed`        | Detect failed critical system pods |
### Certificate Health
| Alert                                      | Purpose                                    |
| ------------------------------------------ | ------------------------------------------ |
| `RKE2KubeletClientCertificateExpiringSoon` | Detect upcoming kubelet client cert expiry |
| `RKE2KubeletServerCertificateExpiringSoon` | Detect upcoming kubelet server cert expiry |
### Requirements

The following components must already exist in the cluster:
| Requirement             | Purpose                    |
| ----------------------- | -------------------------- |
| Prometheus              | Alert evaluation           |
| kube-state-metrics      | Kubernetes object metrics  |
| node-exporter           | Node metrics               |
| CoreDNS metrics enabled | DNS alerting               |
| kubelet metrics enabled | Runtime/kubelet visibility |
Typically these are already present when using: Rancher Monitoring or any Prometheus stack

## Installation
### Apply the PrometheusRule
kubectl apply -f rke2-monitor.yaml
### Verify Rule Loading
kubectl get prometheusrules -A
