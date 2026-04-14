# Kubernetes One-Page Cheat Sheet (Senior DevOps + Architect)

High-value kubectl commands for daily ops, incident response, and interview revision.

## 1) Context Safety First

kubectl config get-contexts                     # List all contexts
kubectl config current-context                  # Show active context
kubectl config use-context <context>            # Switch cluster context
kubectl auth can-i <verb> <resource> -n <ns>   # Validate RBAC permission

Use case:
- Prevent mistakes in wrong cluster and verify access before changes.

## 2) Fast Discovery

kubectl get ns                                  # List namespaces
kubectl get all -n <ns>                         # Quick workload inventory
kubectl get pods -o wide -n <ns>                # Pod to node/IP mapping
kubectl api-resources                            # List available resource kinds
kubectl explain deployment.spec.template.spec   # Inspect schema path

Use case:
- Understand environment and resources before troubleshooting or rollout.

## 3) Safe Deploy Flow

kubectl diff -f k8s/<env>/                      # Preview changes
kubectl apply -f k8s/<env>/                     # Apply declarative manifests
kubectl rollout status deploy/<name> -n <ns>    # Wait for rollout completion
kubectl rollout history deploy/<name> -n <ns>   # View revisions
kubectl rollout undo deploy/<name> -n <ns>      # Roll back quickly

Use case:
- Controlled release with visibility and fast recovery.

## 4) Pod Troubleshooting Core

kubectl get pods -n <ns>
kubectl describe pod <pod> -n <ns>              # Events, probes, status
kubectl logs <pod> -n <ns>                      # Current logs
kubectl logs --previous <pod> -n <ns>           # Prior crashed container logs
kubectl logs -f <pod> -n <ns>                   # Live stream
kubectl exec -it <pod> -n <ns> -- /bin/sh       # Interactive diagnostics
kubectl cp <ns>/<pod>:/tmp/app.log ./app.log    # Copy files from pod

Use case:
- Diagnose CrashLoopBackOff, config issues, network failures, and app errors.

## 5) Service and Network Debug

kubectl get svc -n <ns>
kubectl describe svc <name> -n <ns>             # Selector and endpoint checks
kubectl get endpoints -n <ns>                   # Verify backing pods
kubectl get ingress -n <ns>
kubectl describe ingress <name> -n <ns>
kubectl port-forward svc/<service> 8080:80 -n <ns>

Use case:
- Debug request routing from ingress to service to pod.

## 6) Config, Secrets, and Security

kubectl get configmap -n <ns>
kubectl get secret -n <ns>
kubectl create secret generic <name> --from-literal=token=<value> -n <ns>
kubectl describe secret <name> -n <ns>
kubectl get secret <name> -o jsonpath='{.data.token}' -n <ns> | base64 -d

Use case:
- Validate runtime configuration and credential wiring.

## 7) Scale and Capacity

kubectl top node                                # Node resource pressure
kubectl top pod -n <ns>                         # Pod-level CPU/memory
kubectl scale deploy/<name> --replicas=6 -n <ns>
kubectl autoscale deploy/<name> --cpu-percent=70 --min=2 --max=10 -n <ns>

Use case:
- Respond to traffic spikes and right-size workloads.

## 8) Node Maintenance Pattern

kubectl cordon <node>
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
# patch/reboot node
kubectl uncordon <node>

Use case:
- Perform maintenance without uncontrolled scheduling disruptions.

## 9) Incident Commands (Must Know)

kubectl get events -n <ns> --sort-by=.metadata.creationTimestamp
kubectl get events -A --field-selector type=Warning
kubectl get pod <pod> -o yaml -n <ns>
kubectl debug -it <pod> --image=busybox -n <ns>

Use case:
- Build timeline quickly and debug minimal/distroless containers.

## 10) Top Interview Differences

- apply vs create: apply is declarative update/create, create fails if object exists.
- diff vs apply: diff previews what apply will change.
- logs vs describe: logs for app output, describe for Kubernetes state/events.
- scale vs autoscale: scale is manual fixed replicas, autoscale is dynamic.
- cordon vs drain: cordon blocks new pods, drain evicts existing pods safely.

## 11) 60-Second Production Checklist

1. kubectl config current-context
2. kubectl auth can-i apply deployments -n <ns>
3. kubectl diff -f k8s/<env>/
4. kubectl apply -f k8s/<env>/
5. kubectl rollout status deploy/<critical-app> -n <ns>
6. kubectl get pods -n <ns>
7. kubectl get events -n <ns> --sort-by=.metadata.creationTimestamp

## 12) Real Command Snippets

Rolling update to new image:

kubectl set image deploy/api api=ghcr.io/org/api:v1.8.2 -n production
kubectl rollout status deploy/api -n production

Quick rollback:

kubectl rollout undo deploy/api -n production
kubectl rollout status deploy/api -n production

Debug service locally:

kubectl port-forward svc/api 8080:80 -n production

---

Designed for rapid execution, production safety, and senior-level Kubernetes operations.

## Related Guide

- Detailed handbook: [Kuberneties.md](Kuberneties.md)