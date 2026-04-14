# Kubernetes Commands Handbook (Senior DevOps + Architect)

A practical and production-focused Kubernetes command reference.
Each section includes command purpose and common real-world use cases.

## 1) Cluster Context and Access

| Command | Description | Use Case |
|---|---|---|
| `kubectl version --short` | Shows client and server Kubernetes versions. | Validate compatibility before upgrades or deploying new API versions. |
| `kubectl cluster-info` | Displays cluster API and control plane endpoints. | Quick health check to ensure kubectl is connected to the right cluster. |
| `kubectl config get-contexts` | Lists all configured contexts. | Verify available environments like dev/stage/prod. |
| `kubectl config current-context` | Shows active context. | Prevent accidental deployment to wrong cluster. |
| `kubectl config use-context <context-name>` | Switches active context. | Move safely between clusters for operations. |
| `kubectl config view --minify` | Shows active kubeconfig details only. | Confirm current user/cluster/namespace settings before running changes. |
| `kubectl auth can-i <verb> <resource>` | Checks RBAC permission for an action. | Validate if service account/user can perform required operations. |

## 2) Namespace and Resource Discovery

| Command | Description | Use Case |
|---|---|---|
| `kubectl get ns` | Lists namespaces. | Understand workload segmentation in a cluster. |
| `kubectl create ns <name>` | Creates namespace. | Isolate application environments or teams. |
| `kubectl get all -n <namespace>` | Lists core workload resources in a namespace. | Fast inventory of running app components. |
| `kubectl get <resource> -A` | Lists a resource across all namespaces. | Global visibility for troubleshooting or audits. |
| `kubectl api-resources` | Lists supported resource kinds. | Discover available custom and built-in resources. |
| `kubectl explain <resource>` | Shows schema and field docs for a resource. | Author manifests correctly without guessing fields. |

## 3) Apply, Diff, and Delete Workloads

| Command | Description | Use Case |
|---|---|---|
| `kubectl apply -f <file-or-dir>` | Creates or updates resources declaratively. | Standard GitOps-style deployment flow. |
| `kubectl apply -k <kustomize-dir>` | Applies kustomize overlays. | Manage environment-specific config without duplication. |
| `kubectl diff -f <file-or-dir>` | Shows manifest changes before apply. | Safe pre-deployment review in production pipelines. |
| `kubectl delete -f <file-or-dir>` | Deletes resources defined in manifest. | Controlled teardown of components. |
| `kubectl delete <resource> <name> -n <ns>` | Deletes one specific resource. | Remove problematic instance quickly. |
| `kubectl replace -f <file>` | Replaces full object spec directly. | Emergency full-object update when patching is unsuitable. |
| `kubectl annotate <resource> <name> key=value -n <ns>` | Adds metadata annotation. | Mark change tickets, ownership, or rollout notes. |
| `kubectl label <resource> <name> key=value -n <ns>` | Adds or updates labels. | Drive selectors, policy, and observability filtering. |

## 4) Pod Operations and Troubleshooting

| Command | Description | Use Case |
|---|---|---|
| `kubectl get pods -n <ns>` | Lists pods in namespace. | Check app pod count and status quickly. |
| `kubectl get pods -o wide -n <ns>` | Includes node IP and pod IP details. | Investigate networking/node placement issues. |
| `kubectl describe pod <pod> -n <ns>` | Shows events, conditions, and container states. | Root cause CrashLoopBackOff/ImagePullBackOff. |
| `kubectl logs <pod> -n <ns>` | Shows container logs from a pod. | Inspect runtime errors. |
| `kubectl logs <pod> -c <container> -n <ns>` | Logs from specific container in multi-container pod. | Debug sidecar/main container separately. |
| `kubectl logs -f <pod> -n <ns>` | Streams logs live. | Monitor behavior during rollout or incident. |
| `kubectl logs --previous <pod> -n <ns>` | Fetches logs from prior crashed container instance. | Analyze startup failures after restart. |
| `kubectl exec -it <pod> -n <ns> -- /bin/sh` | Opens shell in container. | In-container diagnostics for env/network/files. |
| `kubectl cp <ns>/<pod>:<path> <local-path>` | Copies files from pod to local. | Export logs/dumps for deeper analysis. |
| `kubectl top pod -n <ns>` | Shows pod CPU/memory metrics. | Spot resource pressure and sizing gaps. |

## 5) Deployments, ReplicaSets, and Scaling

| Command | Description | Use Case |
|---|---|---|
| `kubectl get deploy -n <ns>` | Lists deployments. | Verify desired applications exist. |
| `kubectl describe deploy <name> -n <ns>` | Shows deployment strategy/events/history. | Diagnose failed rollout conditions. |
| `kubectl rollout status deploy/<name> -n <ns>` | Tracks rollout completion. | Gate CI/CD progress based on deployment success. |
| `kubectl rollout history deploy/<name> -n <ns>` | Shows revision history. | Audit deployment changes over time. |
| `kubectl rollout undo deploy/<name> -n <ns>` | Reverts deployment to previous revision. | Fast recovery from bad release. |
| `kubectl scale deploy/<name> --replicas=<n> -n <ns>` | Adjusts replica count. | Handle traffic spikes manually when needed. |
| `kubectl autoscale deploy/<name> --cpu-percent=70 --min=2 --max=10 -n <ns>` | Creates HPA based on CPU. | Enable elastic scaling for variable load. |
| `kubectl set image deploy/<name> <container>=<image>:<tag> -n <ns>` | Updates container image. | Trigger controlled rolling update. |

## 6) Stateful and Batch Workloads

| Command | Description | Use Case |
|---|---|---|
| `kubectl get sts -n <ns>` | Lists StatefulSets. | Manage stateful services like DBs/queues. |
| `kubectl rollout status sts/<name> -n <ns>` | Monitors StatefulSet rollout. | Ensure ordered updates complete safely. |
| `kubectl get job -n <ns>` | Lists Jobs. | Track one-time processing tasks. |
| `kubectl create job --from=cronjob/<cronjob> <job-name> -n <ns>` | Runs a CronJob immediately as Job. | On-demand execution for maintenance/backup. |
| `kubectl get cronjob -n <ns>` | Lists scheduled CronJobs. | Verify periodic task schedules and status. |

## 7) Services, Endpoints, and Networking

| Command | Description | Use Case |
|---|---|---|
| `kubectl get svc -n <ns>` | Lists services. | Validate service exposure types and cluster IPs. |
| `kubectl describe svc <name> -n <ns>` | Shows ports, selectors, and endpoints mapping. | Diagnose service routing issues. |
| `kubectl get endpoints -n <ns>` | Displays backend pod IPs for services. | Confirm service has healthy target pods. |
| `kubectl get ingress -n <ns>` | Lists ingress rules. | Review domain/path routing configs. |
| `kubectl describe ingress <name> -n <ns>` | Shows ingress backend and events. | Debug path/host routing failures. |
| `kubectl port-forward svc/<service> 8080:80 -n <ns>` | Forwards local port to service port. | Local debugging without external exposure. |
| `kubectl port-forward pod/<pod> 8080:8080 -n <ns>` | Forwards local port to pod port. | Debug internal admin endpoints directly. |

## 8) ConfigMaps, Secrets, and Security

| Command | Description | Use Case |
|---|---|---|
| `kubectl get configmap -n <ns>` | Lists ConfigMaps. | Verify non-secret configuration objects. |
| `kubectl create configmap <name> --from-file=<path> -n <ns>` | Builds ConfigMap from files. | Externalize app configs cleanly. |
| `kubectl get secret -n <ns>` | Lists secrets metadata. | Confirm secret resources exist. |
| `kubectl create secret generic <name> --from-literal=key=value -n <ns>` | Creates generic secret from literals. | Quick bootstrap secret for testing or migration. |
| `kubectl describe secret <name> -n <ns>` | Shows secret metadata and key names. | Validate mounted/env key structure. |
| `kubectl get secret <name> -o jsonpath='{.data.key}' -n <ns> | base64 -d` | Decodes one secret key value. | Verify secret content when debugging auth/connectivity. |
| `kubectl auth can-i list secrets -n <ns>` | Checks permission to view secret resources. | RBAC validation for least-privilege reviews. |

## 9) Nodes, Scheduling, and Capacity

| Command | Description | Use Case |
|---|---|---|
| `kubectl get nodes` | Lists cluster worker/control nodes. | Inspect capacity and readiness status. |
| `kubectl describe node <node>` | Shows node conditions, allocatable resources, taints. | Diagnose scheduling and resource exhaustion. |
| `kubectl top node` | Node CPU/memory usage. | Capacity planning and hotspot analysis. |
| `kubectl cordon <node>` | Marks node unschedulable. | Prevent new pods before maintenance. |
| `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` | Evacuates workloads safely from node. | Planned node patching/upgrades. |
| `kubectl uncordon <node>` | Re-enables scheduling on node. | Return node to service after maintenance. |
| `kubectl taint nodes <node> key=value:NoSchedule` | Adds scheduling restriction taint. | Reserve nodes for special workloads. |
| `kubectl label nodes <node> role=workload-a` | Labels node for affinity/selectors. | Control workload placement strategy. |

## 10) Events, Debugging, and Incident Response

| Command | Description | Use Case |
|---|---|---|
| `kubectl get events -n <ns> --sort-by=.metadata.creationTimestamp` | Shows event timeline oldest to newest. | Build incident chronology. |
| `kubectl get events -A --field-selector type=Warning` | Shows warnings cluster-wide. | Detect broad failures quickly. |
| `kubectl get pod <pod> -o yaml -n <ns>` | Full pod spec and status in YAML. | Deep debugging and comparison with desired state. |
| `kubectl debug node/<node> -it --image=busybox` | Starts debug container on node. | Node-level network and filesystem diagnostics. |
| `kubectl debug -it <pod> --image=busybox -n <ns>` | Creates ephemeral debug container in pod context. | Troubleshoot distroless/minimal images. |

## 11) Architecture-Grade Operations Patterns

### Controlled Production Rollout

```bash
kubectl config use-context prod-cluster
kubectl diff -f k8s/prod/
kubectl apply -f k8s/prod/
kubectl rollout status deploy/api -n production
kubectl rollout status deploy/web -n production
```

Use case:
- Safe and observable release with pre-change diff and rollout gating.

### Fast Rollback During Incident

```bash
kubectl rollout undo deploy/api -n production
kubectl rollout status deploy/api -n production
kubectl get pods -n production
kubectl logs -f deploy/api -n production
```

Use case:
- Immediate rollback to last stable revision after failed deployment.

### Node Maintenance Window

```bash
kubectl cordon worker-01
kubectl drain worker-01 --ignore-daemonsets --delete-emptydir-data
# perform node patching/reboot
kubectl uncordon worker-01
```

Use case:
- Zero-surprise node maintenance with controlled scheduling behavior.

### Live Service Debugging via Port Forward

```bash
kubectl get svc -n payments
kubectl port-forward svc/payments-api 8080:80 -n payments
```

Use case:
- Validate service endpoints from local machine without exposing service publicly.

## 12) Safety and Best Practices

- Always verify current context before apply/delete in shared environments.
- Use namespaces and labels consistently for team ownership and filtering.
- Prefer declarative apply from version-controlled manifests.
- Use `kubectl diff` in production change workflows.
- Avoid direct secret values in shell history when possible.
- Prefer `rollout undo` over manual hot edits during incidents.
- Use `drain` plus `cordon` for maintenance, not ad-hoc pod deletions.
- Track every operational change with change-ticket annotations/labels.

## 13) High-Value Quick Reference

```bash
# Context safety
kubectl config current-context
kubectl config use-context <context>

# Deploy safely
kubectl diff -f <manifest-dir>
kubectl apply -f <manifest-dir>
kubectl rollout status deploy/<name> -n <ns>

# Troubleshoot quickly
kubectl describe pod <pod> -n <ns>
kubectl logs -f <pod> -n <ns>
kubectl get events -n <ns> --sort-by=.metadata.creationTimestamp

# Recover quickly
kubectl rollout undo deploy/<name> -n <ns>
```

---

This handbook is designed for senior-level Kubernetes operations, production support, and architecture decision workflows.

## Related Guide

- Quick revision version: [Kubernetes-one-page.md](Kubernetes-one-page.md)