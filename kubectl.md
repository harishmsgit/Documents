# kubectl Command Reference — Production-Ready (WSL / Linux)

> **Environment:** Windows Subsystem for Linux (WSL2) + kubectl  
> **Audience:** DevOps Architect  
> **Scope:** Day-to-day operations, CI/CD pipelines, cluster management, troubleshooting, security, and scaling

---

## Prerequisites — WSL2 Setup

Install kubectl in WSL2:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

Install kubeconfig from Windows into WSL:
```bash
mkdir -p ~/.kube
cp /mnt/c/Users/<your-user>/.kube/config ~/.kube/config
chmod 600 ~/.kube/config
```

Verify cluster connectivity:
```bash
kubectl cluster-info
kubectl get nodes
```

---

## 1. Cluster & Context Management

List all available contexts (clusters):
```bash
kubectl config get-contexts
```

Show the current active context:
```bash
kubectl config current-context
```

Switch to a different cluster context:
```bash
kubectl config use-context <context-name>
```

Rename a context:
```bash
kubectl config rename-context <old-name> <new-name>
```

Set a default namespace for the current context:
```bash
kubectl config set-context --current --namespace=<namespace>
```

View full kubeconfig:
```bash
kubectl config view
```

Merge multiple kubeconfigs:
```bash
export KUBECONFIG=~/.kube/config:~/.kube/cluster2-config
kubectl config view --flatten > ~/.kube/merged-config
```

Check cluster component health:
```bash
kubectl get componentstatuses
kubectl get cs
```

Get cluster API server info:
```bash
kubectl cluster-info
kubectl cluster-info dump
```

---

## 2. Namespace Management

List all namespaces:
```bash
kubectl get namespaces
kubectl get ns
```

Create a namespace:
```bash
kubectl create namespace production
kubectl create namespace staging
kubectl create namespace monitoring
```

Delete a namespace (removes all resources inside):
```bash
kubectl delete namespace <namespace>
```

Show all resources across all namespaces:
```bash
kubectl get all --all-namespaces
kubectl get all -A
```

---

## 3. Node Management

List all nodes:
```bash
kubectl get nodes
kubectl get nodes -o wide
```

Describe a specific node:
```bash
kubectl describe node <node-name>
```

Check node resource usage:
```bash
kubectl top nodes
```

Cordon a node (prevent new pod scheduling):
```bash
kubectl cordon <node-name>
```

Drain a node (safely evict all pods before maintenance):
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

Uncordon (re-enable scheduling after maintenance):
```bash
kubectl uncordon <node-name>
```

Label a node:
```bash
kubectl label node <node-name> environment=production
```

Taint a node (restrict which pods can run on it):
```bash
kubectl taint nodes <node-name> key=value:NoSchedule
kubectl taint nodes <node-name> key=value:NoExecute
```

Remove a taint:
```bash
kubectl taint nodes <node-name> key=value:NoSchedule-
```

---

## 4. Pod Management

List all pods in the current namespace:
```bash
kubectl get pods
kubectl get pods -o wide
```

List pods in a specific namespace:
```bash
kubectl get pods -n production
```

List pods across all namespaces:
```bash
kubectl get pods -A
```

Watch pod status live:
```bash
kubectl get pods -w
kubectl get pods -n production -w
```

Describe a pod (events, resource limits, conditions):
```bash
kubectl describe pod <pod-name> -n <namespace>
```

Get pod logs:
```bash
kubectl logs <pod-name> -n <namespace>
```

Follow pod logs live:
```bash
kubectl logs -f <pod-name> -n <namespace>
```

Get logs from a specific container in a multi-container pod:
```bash
kubectl logs <pod-name> -c <container-name> -n <namespace>
```

Get previous container logs (after a crash):
```bash
kubectl logs <pod-name> --previous -n <namespace>
```

Get last N lines of logs:
```bash
kubectl logs <pod-name> --tail=100 -n <namespace>
```

Execute a command inside a pod:
```bash
kubectl exec -it <pod-name> -n <namespace> -- bash
kubectl exec -it <pod-name> -n <namespace> -- sh
```

Execute in a specific container:
```bash
kubectl exec -it <pod-name> -c <container-name> -n <namespace> -- bash
```

Copy files from a pod to host:
```bash
kubectl cp <namespace>/<pod-name>:/path/in/pod ./local-path
```

Copy files from host to a pod:
```bash
kubectl cp ./local-file <namespace>/<pod-name>:/path/in/pod
```

Delete a pod (will be recreated by Deployment):
```bash
kubectl delete pod <pod-name> -n <namespace>
```

Force delete a stuck pod:
```bash
kubectl delete pod <pod-name> -n <namespace> --force --grace-period=0
```

Get pod resource usage:
```bash
kubectl top pods -n <namespace>
kubectl top pods -A
```

---

## 5. Deployment Management

Create a deployment:
```bash
kubectl create deployment <name> --image=<image>:<tag> -n <namespace>
```

Apply a deployment from a YAML file:
```bash
kubectl apply -f deployment.yaml
```

List all deployments:
```bash
kubectl get deployments -n <namespace>
kubectl get deploy -A
```

Describe a deployment:
```bash
kubectl describe deployment <name> -n <namespace>
```

Scale a deployment:
```bash
kubectl scale deployment <name> --replicas=5 -n <namespace>
```

Update the image of a deployment (rolling update):
```bash
kubectl set image deployment/<name> <container-name>=<image>:<new-tag> -n <namespace>
```

Check rollout status:
```bash
kubectl rollout status deployment/<name> -n <namespace>
```

View rollout history:
```bash
kubectl rollout history deployment/<name> -n <namespace>
```

Rollback to the previous version:
```bash
kubectl rollout undo deployment/<name> -n <namespace>
```

Rollback to a specific revision:
```bash
kubectl rollout undo deployment/<name> --to-revision=<revision-number> -n <namespace>
```

Pause a rollout:
```bash
kubectl rollout pause deployment/<name> -n <namespace>
```

Resume a paused rollout:
```bash
kubectl rollout resume deployment/<name> -n <namespace>
```

Restart all pods in a deployment (zero-downtime):
```bash
kubectl rollout restart deployment/<name> -n <namespace>
```

Delete a deployment:
```bash
kubectl delete deployment <name> -n <namespace>
```

---

## 6. Service Management

List all services:
```bash
kubectl get services -n <namespace>
kubectl get svc -A
```

Describe a service:
```bash
kubectl describe service <service-name> -n <namespace>
```

Expose a deployment as a service:
```bash
# ClusterIP (internal only)
kubectl expose deployment <name> --port=80 --target-port=8080 --type=ClusterIP -n <namespace>

# NodePort (accessible on node IP)
kubectl expose deployment <name> --port=80 --target-port=8080 --type=NodePort -n <namespace>

# LoadBalancer (cloud provider external IP)
kubectl expose deployment <name> --port=80 --target-port=8080 --type=LoadBalancer -n <namespace>
```

Port-forward a service to localhost (dev/debug):
```bash
kubectl port-forward service/<service-name> 8080:80 -n <namespace>
```

Port-forward a specific pod:
```bash
kubectl port-forward pod/<pod-name> 8080:8080 -n <namespace>
```

Delete a service:
```bash
kubectl delete service <service-name> -n <namespace>
```

---

## 7. ConfigMap & Secret Management

Create a ConfigMap from a literal value:
```bash
kubectl create configmap <name> --from-literal=KEY=VALUE -n <namespace>
```

Create a ConfigMap from a file:
```bash
kubectl create configmap <name> --from-file=config.properties -n <namespace>
```

List all ConfigMaps:
```bash
kubectl get configmaps -n <namespace>
```

Describe a ConfigMap:
```bash
kubectl describe configmap <name> -n <namespace>
```

Create a Secret (generic):
```bash
kubectl create secret generic <name> --from-literal=username=admin --from-literal=password=secret -n <namespace>
```

Create a Secret from a file:
```bash
kubectl create secret generic <name> --from-file=./credentials.env -n <namespace>
```

Create a TLS Secret:
```bash
kubectl create secret tls <name> --cert=tls.crt --key=tls.key -n <namespace>
```

Create a Docker registry secret:
```bash
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.example.com \
  --docker-username=<user> \
  --docker-password=<password> \
  --docker-email=<email> \
  -n <namespace>
```

List all secrets:
```bash
kubectl get secrets -n <namespace>
```

Decode a secret value:
```bash
kubectl get secret <name> -n <namespace> -o jsonpath="{.data.<key>}" | base64 --decode
```

Edit a secret:
```bash
kubectl edit secret <name> -n <namespace>
```

Delete a secret:
```bash
kubectl delete secret <name> -n <namespace>
```

---

## 8. Ingress Management

List all ingresses:
```bash
kubectl get ingress -n <namespace>
kubectl get ing -A
```

Describe an ingress:
```bash
kubectl describe ingress <name> -n <namespace>
```

Apply an ingress from YAML:
```bash
kubectl apply -f ingress.yaml
```

Delete an ingress:
```bash
kubectl delete ingress <name> -n <namespace>
```

---

## 9. StatefulSet Management

List StatefulSets:
```bash
kubectl get statefulsets -n <namespace>
kubectl get sts -A
```

Scale a StatefulSet:
```bash
kubectl scale statefulset <name> --replicas=3 -n <namespace>
```

Describe a StatefulSet:
```bash
kubectl describe statefulset <name> -n <namespace>
```

Restart a StatefulSet:
```bash
kubectl rollout restart statefulset/<name> -n <namespace>
```

---

## 10. DaemonSet Management

List DaemonSets:
```bash
kubectl get daemonsets -n <namespace>
kubectl get ds -A
```

Describe a DaemonSet:
```bash
kubectl describe daemonset <name> -n <namespace>
```

Restart a DaemonSet:
```bash
kubectl rollout restart daemonset/<name> -n <namespace>
```

---

## 11. Persistent Volume & Storage

List PersistentVolumes (cluster-wide):
```bash
kubectl get pv
```

List PersistentVolumeClaims:
```bash
kubectl get pvc -n <namespace>
```

Describe a PVC:
```bash
kubectl describe pvc <name> -n <namespace>
```

Delete a PVC:
```bash
kubectl delete pvc <name> -n <namespace>
```

List StorageClasses:
```bash
kubectl get storageclass
kubectl get sc
```

---

## 12. Resource Quotas & Limits

List resource quotas in a namespace:
```bash
kubectl get resourcequota -n <namespace>
kubectl describe resourcequota -n <namespace>
```

List LimitRanges:
```bash
kubectl get limitrange -n <namespace>
kubectl describe limitrange -n <namespace>
```

---

## 13. Horizontal Pod Autoscaler (HPA)

Create an HPA for a deployment:
```bash
kubectl autoscale deployment <name> --min=2 --max=10 --cpu-percent=70 -n <namespace>
```

List all HPAs:
```bash
kubectl get hpa -n <namespace>
kubectl get hpa -A
```

Describe an HPA:
```bash
kubectl describe hpa <name> -n <namespace>
```

Delete an HPA:
```bash
kubectl delete hpa <name> -n <namespace>
```

---

## 14. RBAC — Role-Based Access Control

List ClusterRoles:
```bash
kubectl get clusterroles
```

List Roles in a namespace:
```bash
kubectl get roles -n <namespace>
```

List RoleBindings:
```bash
kubectl get rolebindings -n <namespace>
```

List ClusterRoleBindings:
```bash
kubectl get clusterrolebindings
```

Create a Role from YAML:
```bash
kubectl apply -f role.yaml
```

Bind a role to a user:
```bash
kubectl create rolebinding <binding-name> \
  --role=<role-name> \
  --user=<username> \
  -n <namespace>
```

Check what a user can do:
```bash
kubectl auth can-i create pods --as=<username> -n <namespace>
kubectl auth can-i '*' '*' --as=<username>
```

---

## 15. Network Policies

List NetworkPolicies:
```bash
kubectl get networkpolicies -n <namespace>
kubectl get netpol -A
```

Describe a NetworkPolicy:
```bash
kubectl describe networkpolicy <name> -n <namespace>
```

Apply a NetworkPolicy from YAML:
```bash
kubectl apply -f network-policy.yaml
```

---

## 16. Jobs & CronJobs

Create a one-off Job:
```bash
kubectl create job <name> --image=<image> -n <namespace>
```

List Jobs:
```bash
kubectl get jobs -n <namespace>
```

Describe a Job:
```bash
kubectl describe job <name> -n <namespace>
```

Create a CronJob:
```bash
kubectl create cronjob <name> --image=<image> --schedule="0 2 * * *" -n <namespace>
```

List CronJobs:
```bash
kubectl get cronjobs -n <namespace>
```

Manually trigger a CronJob:
```bash
kubectl create job --from=cronjob/<name> <manual-job-name> -n <namespace>
```

Delete a Job:
```bash
kubectl delete job <name> -n <namespace>
```

---

## 17. Apply, Diff & Dry-Run

Apply resources from a file or directory:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f ./manifests/
kubectl apply -f ./manifests/ -R          # recursive
```

Preview changes before applying (diff):
```bash
kubectl diff -f deployment.yaml
```

Dry run — validate without applying:
```bash
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f deployment.yaml --dry-run=server
```

Delete resources from a file:
```bash
kubectl delete -f deployment.yaml
```

---

## 18. Labels & Annotations

Add a label to a resource:
```bash
kubectl label pod <pod-name> env=production -n <namespace>
kubectl label node <node-name> role=worker
```

Remove a label:
```bash
kubectl label pod <pod-name> env- -n <namespace>
```

List pods by label selector:
```bash
kubectl get pods -l env=production -n <namespace>
kubectl get pods -l 'app in (jenkins, nginx)' -n <namespace>
```

Add an annotation:
```bash
kubectl annotate pod <pod-name> description="main app pod" -n <namespace>
```

Remove an annotation:
```bash
kubectl annotate pod <pod-name> description- -n <namespace>
```

---

## 19. Output Formats & JSONPath

Output in wide format:
```bash
kubectl get pods -o wide
```

Output as YAML:
```bash
kubectl get deployment <name> -n <namespace> -o yaml
```

Output as JSON:
```bash
kubectl get pod <pod-name> -n <namespace> -o json
```

Extract a specific field with JSONPath:
```bash
kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
```

Custom column output:
```bash
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

Sort by a field:
```bash
kubectl get pods --sort-by=.metadata.creationTimestamp -n <namespace>
kubectl get pods --sort-by=.status.phase -n <namespace>
```

---

## 20. Events & Troubleshooting

Get all events in a namespace (sorted by time):
```bash
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

Get warning events only:
```bash
kubectl get events -n <namespace> --field-selector type=Warning
```

Describe any resource for detailed events:
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl describe node <node-name>
kubectl describe deployment <name> -n <namespace>
```

Check why a pod is in CrashLoopBackOff:
```bash
kubectl logs <pod-name> --previous -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```

Check why a pod is Pending:
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

Debug with a temporary pod (alpine):
```bash
kubectl run debug-pod --image=alpine --restart=Never --rm -it -n <namespace> -- sh
```

Debug with network tools (curl, dig):
```bash
kubectl run netdebug --image=nicolaka/netshoot --restart=Never --rm -it -n <namespace> -- bash
```

Check DNS resolution inside the cluster:
```bash
kubectl exec -it <pod-name> -n <namespace> -- nslookup kubernetes.default
kubectl exec -it <pod-name> -n <namespace> -- nslookup <service-name>.<namespace>.svc.cluster.local
```

---

## 21. Helm (Package Manager for Kubernetes)

Install Helm in WSL:
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

Add a Helm repo:
```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add jenkins https://charts.jenkins.io
helm repo update
```

Search for a chart:
```bash
helm search repo jenkins
```

Install a chart:
```bash
helm install jenkins jenkins/jenkins -n jenkins --create-namespace
```

Install with custom values file:
```bash
helm install jenkins jenkins/jenkins -f values-prod.yaml -n jenkins
```

List installed releases:
```bash
helm list -n jenkins
helm list -A
```

Upgrade a release:
```bash
helm upgrade jenkins jenkins/jenkins -f values-prod.yaml -n jenkins
```

Rollback a Helm release:
```bash
helm rollback jenkins 1 -n jenkins
```

Uninstall a release:
```bash
helm uninstall jenkins -n jenkins
```

Render templates locally (dry-run):
```bash
helm template jenkins jenkins/jenkins -f values-prod.yaml
```

Check release history:
```bash
helm history jenkins -n jenkins
```

---

## 22. kubectl Plugins & Productivity (krew)

Install krew (kubectl plugin manager) in WSL:
```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/arm.*$/arm/' -e 's/aarch64$/arm64/')" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew-${OS}_${ARCH}.tar.gz" &&
  tar zxvf "krew-${OS}_${ARCH}.tar.gz" &&
  KREW=./krew-"${OS}_${ARCH}" &&
  "$KREW" install krew
)
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

Essential plugins for DevOps architects:
```bash
kubectl krew install ctx          # fast context switching
kubectl krew install ns           # fast namespace switching
kubectl krew install neat         # clean YAML output
kubectl krew install top          # resource usage
kubectl krew install tree         # show resource ownership tree
kubectl krew install stern        # multi-pod log tailing
kubectl krew install access-matrix # RBAC access matrix
```

Use kubectx / kubens:
```bash
kubectl ctx                   # list contexts
kubectl ctx production        # switch to production context
kubectl ns                    # list namespaces
kubectl ns monitoring         # switch to monitoring namespace
```

Tail logs from multiple pods (stern):
```bash
kubectl stern -n production <pod-name-prefix>
kubectl stern -n production -l app=jenkins
```

---

## 23. Cluster-Level Production Operations

Check cluster API versions:
```bash
kubectl api-resources
kubectl api-versions
```

Export all resources from a namespace (backup):
```bash
kubectl get all -n production -o yaml > production-backup.yaml
```

Apply resources from stdin:
```bash
cat deployment.yaml | kubectl apply -f -
```

Watch all resources in a namespace:
```bash
kubectl get all -n production -w
```

Set resource requests/limits on a deployment:
```bash
kubectl set resources deployment/<name> \
  --requests=cpu=250m,memory=512Mi \
  --limits=cpu=1000m,memory=2Gi \
  -n <namespace>
```

Annotate a deployment with change cause (for rollout history):
```bash
kubectl annotate deployment/<name> kubernetes.io/change-cause="Updated image to v2.0" -n <namespace>
```

Patch a resource inline:
```bash
kubectl patch deployment <name> -n <namespace> \
  -p '{"spec":{"replicas":3}}'
```

Force replace a resource (destructive — use with caution):
```bash
kubectl replace --force -f deployment.yaml
```

---

## 24. Security & Compliance

Scan for privileged containers:
```bash
kubectl get pods -A -o json | \
  jq '.items[] | select(.spec.containers[].securityContext.privileged == true) | .metadata.name'
```

List pods running as root:
```bash
kubectl get pods -A -o json | \
  jq '.items[] | select(.spec.securityContext.runAsUser == 0) | .metadata.name'
```

View admission controllers:
```bash
kubectl get ValidatingWebhookConfiguration
kubectl get MutatingWebhookConfiguration
```

Check pod disruption budgets:
```bash
kubectl get poddisruptionbudget -A
kubectl get pdb -A
```

View service accounts:
```bash
kubectl get serviceaccounts -n <namespace>
kubectl describe serviceaccount <name> -n <namespace>
```

---

## 25. Quick Reference Summary

| Task | Command |
|---|---|
| Switch context | `kubectl config use-context <name>` |
| Set default namespace | `kubectl config set-context --current --namespace=<ns>` |
| List all resources | `kubectl get all -A` |
| Watch pod changes | `kubectl get pods -w -n <ns>` |
| Live logs | `kubectl logs -f <pod> -n <ns>` |
| Exec into pod | `kubectl exec -it <pod> -n <ns> -- bash` |
| Rolling restart | `kubectl rollout restart deployment/<name> -n <ns>` |
| Rollback | `kubectl rollout undo deployment/<name> -n <ns>` |
| Scale | `kubectl scale deployment/<name> --replicas=5 -n <ns>` |
| Drain node | `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` |
| Dry-run apply | `kubectl apply -f file.yaml --dry-run=server` |
| Diff before apply | `kubectl diff -f file.yaml` |
| Decode secret | `kubectl get secret <name> -o jsonpath="{.data.<key>}" \| base64 --decode` |
| Debug temp pod | `kubectl run debug --image=alpine --rm -it --restart=Never -- sh` |
| Get events | `kubectl get events -n <ns> --sort-by='.lastTimestamp'` |
| Resource usage | `kubectl top pods -n <ns>` |
| Force delete pod | `kubectl delete pod <pod> --force --grace-period=0 -n <ns>` |
| Export namespace | `kubectl get all -n <ns> -o yaml > backup.yaml` |
