# Kubernetes Troubleshooting Commands for DevOps and Support

This page is a practical Kubernetes troubleshooting reference for DevOps, Cloud Support, SRE, Infrastructure, and Technical Support work.

The goal is to document common `kubectl` commands, troubleshooting workflows, productivity shortcuts, and investigation patterns used to diagnose real Kubernetes issues.

---

## Table of Contents

- [1. Cluster Information](#cluster-information)
- [2. Daily Kubectl Productivity Setup](#daily-kubectl-productivity-setup)
- [3. Nodes](#nodes)
- [4. Namespaces](#namespaces)
- [5. Pods](#pods)
- [6. Pod Logs](#pod-logs)
- [7. Events](#events)
- [8. Deployments](#deployments)
- [9. ReplicaSets](#replicasets)
- [10. Services](#services)
- [11. Endpoints](#endpoints)
- [12. Ingress](#ingress)
- [13. ConfigMaps](#configmaps)
- [14. Secrets](#secrets)
- [15. Persistent Volumes and Claims](#persistent-volumes-and-claims)
- [16. Resource Usage](#resource-usage)
- [17. Exec Into Pods](#exec-into-pods)
- [18. Port Forwarding](#port-forwarding)
- [19. Rollouts and Rollbacks](#rollouts-and-rollbacks)
- [20. Common Kubernetes Issues](#common-kubernetes-issues)
- [21. Real Troubleshooting Scenarios](#real-troubleshooting-scenarios)

---

<a id="cluster-information"></a>

## 1. Cluster Information

Use these commands to understand the Kubernetes cluster context and basic cluster health.

### Check current context

```bash
kubectl config current-context
```

### View all contexts

```bash
kubectl config get-contexts
```

### Switch context

```bash
kubectl config use-context <context-name>
```

### Cluster information

```bash
kubectl cluster-info
```

### Check Kubernetes version

```bash
kubectl version
```

### Check API resources

```bash
kubectl api-resources
```

### Troubleshooting logic

- Confirm you are connected to the correct cluster.
- Confirm you are using the correct namespace.
- Check if the API server is reachable.
- Validate your permissions if commands fail with `Forbidden`.
- Always confirm context before running commands in production.

---

<a id="daily-kubectl-productivity-setup"></a>

## 2. Daily Kubectl Productivity Setup

These commands help make daily Kubernetes work faster, safer, and easier.

This section is useful for engineers who work with Kubernetes clusters every day.

---

### Create a short alias for `kubectl`

Instead of typing `kubectl` every time, create an alias called `k`.

Temporary alias for the current terminal session:

```bash
alias k=kubectl
```

Now you can run:

```bash
k get pods -A
k get nodes
k describe pod <pod-name> -n <namespace>
```

To make it permanent, add it to your shell profile.

For Bash:

```bash
echo 'alias k=kubectl' >> ~/.bashrc
source ~/.bashrc
```

For Zsh on macOS:

```bash
echo 'alias k=kubectl' >> ~/.zshrc
source ~/.zshrc
```

Common daily examples:

```bash
k get pods -A
k get nodes -o wide
k get svc -A
k get events -A --sort-by=.metadata.creationTimestamp
k logs <pod-name> -n <namespace>
k describe pod <pod-name> -n <namespace>
```

---

### Enable kubectl autocomplete

Autocomplete helps you type Kubernetes commands faster.

For Bash:

```bash
source <(kubectl completion bash)
echo 'source <(kubectl completion bash)' >> ~/.bashrc
```

If using the `k` alias:

```bash
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc
```

For Zsh on macOS:

```bash
source <(kubectl completion zsh)
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
```

If using the `k` alias:

```bash
echo 'alias k=kubectl' >> ~/.zshrc
echo 'compdef __start_kubectl k' >> ~/.zshrc
source ~/.zshrc
```

---

### Check current Kubernetes context

The context tells you which cluster you are connected to.

```bash
kubectl config current-context
```

Using alias:

```bash
k config current-context
```

List all contexts:

```bash
kubectl config get-contexts
```

Switch context:

```bash
kubectl config use-context <context-name>
```

Example:

```bash
k config use-context dev-cluster
```

Important: Always confirm the context before running commands in production.

---

### Set a default namespace

Instead of adding `-n <namespace>` every time, set a default namespace for your current context.

```bash
kubectl config set-context --current --namespace=<namespace>
```

Example:

```bash
kubectl config set-context --current --namespace=production
```

Using alias:

```bash
k config set-context --current --namespace=production
```

Check current namespace:

```bash
kubectl config view --minify | grep namespace
```

This helps avoid mistakes when working across multiple namespaces.

---

### Create useful kubectl aliases

You can add these aliases to `~/.bashrc` or `~/.zshrc`.

```bash
alias k=kubectl
alias kgp='kubectl get pods'
alias kgpa='kubectl get pods -A'
alias kgn='kubectl get nodes -o wide'
alias kgs='kubectl get svc'
alias kgsa='kubectl get svc -A'
alias kge='kubectl get events --sort-by=.metadata.creationTimestamp'
alias kgea='kubectl get events -A --sort-by=.metadata.creationTimestamp'
alias kd='kubectl describe'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias klf='kubectl logs -f'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete -f'
```

Reload shell:

```bash
source ~/.zshrc
```

or:

```bash
source ~/.bashrc
```

Examples:

```bash
kgpa
kgn
kgea
kdp <pod-name> -n <namespace>
kl <pod-name> -n <namespace>
```

---

### Manage kubeconfig files

The kubeconfig file stores cluster connection information.

Default location:

```bash
~/.kube/config
```

Check which kubeconfig is being used:

```bash
echo $KUBECONFIG
```

Set a specific kubeconfig file:

```bash
export KUBECONFIG=~/.kube/config
```

Use a custom kubeconfig:

```bash
export KUBECONFIG=~/Downloads/dev-cluster-kubeconfig.yaml
```

Make it permanent for Zsh:

```bash
echo 'export KUBECONFIG=~/.kube/config' >> ~/.zshrc
source ~/.zshrc
```

Make it permanent for Bash:

```bash
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc
source ~/.bashrc
```

---

### Merge multiple kubeconfig files

If you have multiple cluster config files, you can merge them.

First, back up your current kubeconfig:

```bash
cp ~/.kube/config ~/.kube/config.backup
```

Merge multiple kubeconfig files:

```bash
export KUBECONFIG=~/.kube/config:~/Downloads/dev-kubeconfig.yaml:~/Downloads/prod-kubeconfig.yaml
kubectl config view --flatten > ~/.kube/merged-config
mv ~/.kube/merged-config ~/.kube/config
chmod 600 ~/.kube/config
```

Validate:

```bash
kubectl config get-contexts
```

Important: Always back up your kubeconfig before merging.

---

### Secure kubeconfig permissions

Kubeconfig files may contain sensitive authentication information.

Recommended permission:

```bash
chmod 600 ~/.kube/config
```

Check permissions:

```bash
ls -l ~/.kube/config
```

Avoid committing kubeconfig files to GitHub.

---

### View cluster credentials and users

View kubeconfig details:

```bash
kubectl config view
```

View only current context details:

```bash
kubectl config view --minify
```

View users configured in kubeconfig:

```bash
kubectl config get-users
```

View clusters:

```bash
kubectl config get-clusters
```

---

### Save a certificate from a Kubernetes Secret

TLS certificates are often stored in Kubernetes Secrets.

List secrets:

```bash
kubectl get secrets -n <namespace>
```

Check a TLS secret:

```bash
kubectl describe secret <secret-name> -n <namespace>
```

Save the TLS certificate to a file:

```bash
kubectl get secret <secret-name> -n <namespace> -o jsonpath='{.data.tls\.crt}' | base64 -d > tls.crt
```

Save the TLS private key to a file:

```bash
kubectl get secret <secret-name> -n <namespace> -o jsonpath='{.data.tls\.key}' | base64 -d > tls.key
```

Check certificate details:

```bash
openssl x509 -in tls.crt -text -noout
```

Check certificate expiration date:

```bash
openssl x509 -in tls.crt -noout -enddate
```

Important: Be careful with private keys. Do not commit them to GitHub.

---

### Create a TLS Secret

Create a Kubernetes TLS secret from a certificate and key:

```bash
kubectl create secret tls <secret-name> \
  --cert=tls.crt \
  --key=tls.key \
  -n <namespace>
```

Example:

```bash
kubectl create secret tls app-tls \
  --cert=app.crt \
  --key=app.key \
  -n production
```

---

### Create a Docker registry pull secret

Use this when Kubernetes needs credentials to pull images from a private registry.

```bash
kubectl create secret docker-registry <secret-name> \
  --docker-server=<registry-server> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email> \
  -n <namespace>
```

Example:

```bash
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=user@example.com \
  -n production
```

Reference it in a Deployment:

```yaml
imagePullSecrets:
  - name: regcred
```

Important: Avoid saving passwords in terminal history when possible.

---

### Decode a Kubernetes Secret value

Secrets are base64 encoded.

View secret keys:

```bash
kubectl get secret <secret-name> -n <namespace> -o yaml
```

Decode a specific key:

```bash
kubectl get secret <secret-name> -n <namespace> -o jsonpath='{.data.<key-name>}' | base64 -d
```

Example:

```bash
kubectl get secret app-secret -n production -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

Important: Do not paste decoded secrets into tickets, GitHub, or public chats.

---

### Save pod logs to a local file

Useful for ticket evidence and incident investigation.

```bash
kubectl logs <pod-name> -n <namespace> > pod.log
```

Save logs with timestamp:

```bash
kubectl logs <pod-name> -n <namespace> > "pod-logs-$(date +%F-%H%M%S).log"
```

Save previous crashed container logs:

```bash
kubectl logs <pod-name> -n <namespace> --previous > "pod-previous-logs-$(date +%F-%H%M%S).log"
```

For multi-container pods:

```bash
kubectl logs <pod-name> -c <container-name> -n <namespace> > container.log
```

---

### Export Kubernetes resources to YAML

Useful for documentation, backup, and troubleshooting.

Export a Deployment:

```bash
kubectl get deployment <deployment-name> -n <namespace> -o yaml > deployment.yaml
```

Export a Service:

```bash
kubectl get svc <service-name> -n <namespace> -o yaml > service.yaml
```

Export an Ingress:

```bash
kubectl get ingress <ingress-name> -n <namespace> -o yaml > ingress.yaml
```

Export all resources in a namespace:

```bash
kubectl get all -n <namespace> -o yaml > namespace-resources.yaml
```

Important: Review files before committing because YAML exports may contain sensitive data.

---

### Dry-run before creating resources

Dry-run helps validate commands without applying changes.

Generate YAML without creating the resource:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

Save generated YAML:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Validate an apply without changing the cluster:

```bash
kubectl apply -f deployment.yaml --dry-run=client
```

---

### Useful output formatting

Wide output:

```bash
kubectl get pods -o wide
```

YAML output:

```bash
kubectl get pod <pod-name> -n <namespace> -o yaml
```

JSON output:

```bash
kubectl get pod <pod-name> -n <namespace> -o json
```

Custom columns:

```bash
kubectl get pods -A -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

JSONPath example:

```bash
kubectl get pod <pod-name> -n <namespace> -o jsonpath='{.status.podIP}'
```

---

### Quick daily Kubernetes checks

```bash
k config current-context
k get nodes -o wide
k get pods -A
k get svc -A
k get ingress -A
k get events -A --sort-by=.metadata.creationTimestamp
```

Quick namespace check:

```bash
k get all -n <namespace>
```

Quick pod investigation:

```bash
k describe pod <pod-name> -n <namespace>
k logs <pod-name> -n <namespace>
k logs <pod-name> -n <namespace> --previous
```

---

<a id="nodes"></a>

## 3. Nodes

Nodes are the worker machines where Kubernetes runs workloads.

### List nodes

```bash
kubectl get nodes
```

### List nodes with more details

```bash
kubectl get nodes -o wide
```

### Describe a node

```bash
kubectl describe node <node-name>
```

### Check node labels

```bash
kubectl get nodes --show-labels
```

### Check pods running on a specific node

```bash
kubectl get pods -A -o wide | grep <node-name>
```

### Common node states

```text
Ready               Node is healthy
NotReady            Node has a problem
SchedulingDisabled  Node is cordoned
```

### Troubleshooting logic

- Check if the node is `Ready`.
- Review node conditions.
- Check CPU, memory, disk pressure, and PID pressure.
- Check if pods are failing only on one node.
- Check kubelet or container runtime issues if the node is unhealthy.

---

<a id="namespaces"></a>

## 4. Namespaces

Namespaces separate Kubernetes resources logically.

### List namespaces

```bash
kubectl get namespaces
```

Short version:

```bash
kubectl get ns
```

### List resources in a namespace

```bash
kubectl get all -n <namespace>
```

### Set default namespace for current context

```bash
kubectl config set-context --current --namespace=<namespace>
```

### Troubleshooting logic

- Confirm the correct namespace.
- Many “not found” errors happen because the wrong namespace is being used.
- Use `-A` to search across all namespaces when unsure.

---

<a id="pods"></a>

## 5. Pods

Pods are the smallest deployable units in Kubernetes.

### List pods in current namespace

```bash
kubectl get pods
```

### List pods in a specific namespace

```bash
kubectl get pods -n <namespace>
```

### List pods across all namespaces

```bash
kubectl get pods -A
```

### Show more pod details

```bash
kubectl get pods -n <namespace> -o wide
```

### Describe a pod

```bash
kubectl describe pod <pod-name> -n <namespace>
```

### Get pod YAML

```bash
kubectl get pod <pod-name> -n <namespace> -o yaml
```

### Watch pods live

```bash
kubectl get pods -n <namespace> -w
```

### Common pod statuses

```text
Running             Pod is running
Pending             Pod is waiting to be scheduled
CrashLoopBackOff    Container keeps crashing
ImagePullBackOff    Kubernetes cannot pull the image
ErrImagePull        Image pull failed
Completed           Pod completed successfully
Evicted             Pod was removed from a node
```

### Troubleshooting logic

- Check pod status.
- Run `kubectl describe pod`.
- Review events inside the pod description.
- Check logs.
- Check previous logs if the pod is restarting.
- Validate image, environment variables, ConfigMaps, Secrets, probes, and resources.

---

<a id="pod-logs"></a>

## 6. Pod Logs

Logs are critical for troubleshooting application and container issues.

### View pod logs

```bash
kubectl logs <pod-name> -n <namespace>
```

### Follow logs live

```bash
kubectl logs -f <pod-name> -n <namespace>
```

### View logs from a specific container

```bash
kubectl logs <pod-name> -c <container-name> -n <namespace>
```

### View previous container logs

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

### View last 100 lines

```bash
kubectl logs <pod-name> -n <namespace> --tail=100
```

### Logs since a specific time

```bash
kubectl logs <pod-name> -n <namespace> --since=1h
```

### Troubleshooting logic

- Use current logs for running containers.
- Use `--previous` for containers that restarted.
- Search for errors, exceptions, connection refused, timeout, permission denied, and authentication failures.
- Compare application logs with Kubernetes events.

---

<a id="events"></a>

## 7. Events

Events show what Kubernetes is doing behind the scenes.

### Get events in a namespace

```bash
kubectl get events -n <namespace>
```

### Get events across all namespaces

```bash
kubectl get events -A
```

### Sort events by time

```bash
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

### Watch events live

```bash
kubectl get events -n <namespace> -w
```

### Troubleshooting logic

- Events often explain scheduling issues, image pull failures, probe failures, and volume mount errors.
- Always check events when a pod is `Pending`, `CrashLoopBackOff`, or `ImagePullBackOff`.
- Review events around the time the issue started.

---

<a id="deployments"></a>

## 8. Deployments

Deployments manage application replicas and rolling updates.

### List deployments

```bash
kubectl get deployments -n <namespace>
```

Short version:

```bash
kubectl get deploy -n <namespace>
```

### Describe deployment

```bash
kubectl describe deployment <deployment-name> -n <namespace>
```

### Get deployment YAML

```bash
kubectl get deployment <deployment-name> -n <namespace> -o yaml
```

### Scale deployment

```bash
kubectl scale deployment <deployment-name> --replicas=3 -n <namespace>
```

### Restart deployment

```bash
kubectl rollout restart deployment <deployment-name> -n <namespace>
```

### Check rollout status

```bash
kubectl rollout status deployment <deployment-name> -n <namespace>
```

### Troubleshooting logic

- Check desired replicas vs available replicas.
- Check deployment events.
- Check ReplicaSets.
- Check if pods are being created.
- Check if rollout is stuck.
- Review image, environment variables, resources, and probes.

---

<a id="replicasets"></a>

## 9. ReplicaSets

ReplicaSets maintain the desired number of pod replicas.

### List ReplicaSets

```bash
kubectl get replicasets -n <namespace>
```

Short version:

```bash
kubectl get rs -n <namespace>
```

### Describe ReplicaSet

```bash
kubectl describe rs <replicaset-name> -n <namespace>
```

### Troubleshooting logic

- ReplicaSets are usually managed by Deployments.
- Check ReplicaSets when pods are not being created.
- Compare old and new ReplicaSets during rollouts.
- Useful when investigating failed deployments.

---

<a id="services"></a>

## 10. Services

Services expose pods internally or externally.

### List services

```bash
kubectl get services -n <namespace>
```

Short version:

```bash
kubectl get svc -n <namespace>
```

### Describe service

```bash
kubectl describe svc <service-name> -n <namespace>
```

### Get service YAML

```bash
kubectl get svc <service-name> -n <namespace> -o yaml
```

### Common service types

```text
ClusterIP      Internal service
NodePort       Exposes service on a node port
LoadBalancer   Exposes service using cloud/load balancer
ExternalName   Maps service to external DNS name
```

### Troubleshooting logic

- Confirm the service exists.
- Confirm the service selector matches pod labels.
- Confirm endpoints are populated.
- Confirm the target port matches the container port.
- Test service connectivity from inside the cluster.

---

<a id="endpoints"></a>

## 11. Endpoints

Endpoints show which pods are backing a service.

### Check endpoints

```bash
kubectl get endpoints -n <namespace>
```

For a specific service:

```bash
kubectl get endpoints <service-name> -n <namespace>
```

Newer Kubernetes versions may use EndpointSlices:

```bash
kubectl get endpointslices -n <namespace>
```

### Troubleshooting logic

- If a service has no endpoints, traffic has nowhere to go.
- Check if service selectors match pod labels.
- Check if pods are ready.
- Check readiness probes.
- Check target port and labels.

---

<a id="ingress"></a>

## 12. Ingress

Ingress exposes HTTP/HTTPS services outside the cluster.

### List ingress resources

```bash
kubectl get ingress -n <namespace>
```

Short version:

```bash
kubectl get ing -n <namespace>
```

### Describe ingress

```bash
kubectl describe ingress <ingress-name> -n <namespace>
```

### Get ingress YAML

```bash
kubectl get ingress <ingress-name> -n <namespace> -o yaml
```

### Troubleshooting logic

- Confirm ingress exists.
- Confirm host/path rules.
- Confirm backend service and port.
- Check TLS secret if HTTPS is used.
- Check ingress controller logs.
- Check DNS points to the correct load balancer.

---

<a id="configmaps"></a>

## 13. ConfigMaps

ConfigMaps store non-sensitive configuration.

### List ConfigMaps

```bash
kubectl get configmaps -n <namespace>
```

Short version:

```bash
kubectl get cm -n <namespace>
```

### View ConfigMap

```bash
kubectl describe cm <configmap-name> -n <namespace>
```

### Get ConfigMap YAML

```bash
kubectl get cm <configmap-name> -n <namespace> -o yaml
```

### Troubleshooting logic

- Check if the ConfigMap exists.
- Validate keys and values.
- Confirm the pod references the correct ConfigMap.
- Restart pods if the app does not automatically reload config changes.

---

<a id="secrets"></a>

## 14. Secrets

Secrets store sensitive data such as passwords, tokens, or certificates.

### List secrets

```bash
kubectl get secrets -n <namespace>
```

### Describe secret

```bash
kubectl describe secret <secret-name> -n <namespace>
```

### View secret YAML

```bash
kubectl get secret <secret-name> -n <namespace> -o yaml
```

### Decode a secret value

```bash
kubectl get secret <secret-name> -n <namespace> -o jsonpath="{.data.<key>}" | base64 -d
```

### Troubleshooting logic

- Check if the secret exists.
- Check if the expected key exists.
- Confirm the pod references the correct secret.
- Be careful when displaying secret values.
- Never commit secrets to GitHub.

---

<a id="persistent-volumes-and-claims"></a>

## 15. Persistent Volumes and Claims

Persistent Volumes and Persistent Volume Claims provide storage to pods.

### List PVCs

```bash
kubectl get pvc -n <namespace>
```

### List PVs

```bash
kubectl get pv
```

### Describe PVC

```bash
kubectl describe pvc <pvc-name> -n <namespace>
```

### Describe PV

```bash
kubectl describe pv <pv-name>
```

### Common PVC statuses

```text
Bound     PVC is connected to storage
Pending   PVC is waiting for storage
Lost      PVC lost its backing volume
```

### Troubleshooting logic

- Check if PVC is `Bound`.
- Check storage class.
- Check volume capacity.
- Check access modes.
- Check volume mount errors in pod events.
- Check cloud/on-prem storage backend if needed.

---

<a id="resource-usage"></a>

## 16. Resource Usage

Use these commands to check CPU and memory usage.

### Check node usage

```bash
kubectl top nodes
```

### Check pod usage

```bash
kubectl top pods -n <namespace>
```

### Check pod usage across all namespaces

```bash
kubectl top pods -A
```

Note: `kubectl top` requires metrics-server.

### Check resource requests and limits

```bash
kubectl describe pod <pod-name> -n <namespace>
```

### Troubleshooting logic

- Check if the cluster has metrics-server.
- Compare usage with requests and limits.
- Check for CPU throttling or memory pressure.
- Check if pods are being OOMKilled.
- Check if nodes are under pressure.

---

<a id="exec-into-pods"></a>

## 17. Exec Into Pods

Use `exec` to run commands inside a container.

### Open shell inside a pod

```bash
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash
```

If Bash is not available:

```bash
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh
```

### Run a single command

```bash
kubectl exec <pod-name> -n <namespace> -- env
```

### Test connectivity from inside the pod

```bash
kubectl exec -it <pod-name> -n <namespace> -- curl http://service-name:port
```

### Troubleshooting logic

- Use exec to test DNS, service connectivity, environment variables, and file paths.
- Some minimal containers do not include troubleshooting tools like `curl`, `dig`, or `bash`.
- Avoid modifying production containers manually unless approved.

---

<a id="port-forwarding"></a>

## 18. Port Forwarding

Port forwarding allows local access to a Kubernetes service or pod.

### Port forward to a pod

```bash
kubectl port-forward pod/<pod-name> 8080:8080 -n <namespace>
```

### Port forward to a service

```bash
kubectl port-forward svc/<service-name> 8080:80 -n <namespace>
```

Then test locally:

```bash
curl http://localhost:8080
```

### Troubleshooting logic

- Useful when testing internal services.
- Helps isolate ingress/load balancer issues from application issues.
- If port-forward works but ingress fails, check ingress, service, DNS, or external routing.

---

<a id="rollouts-and-rollbacks"></a>

## 19. Rollouts and Rollbacks

Use these commands to manage deployments.

### Check rollout status

```bash
kubectl rollout status deployment <deployment-name> -n <namespace>
```

### View rollout history

```bash
kubectl rollout history deployment <deployment-name> -n <namespace>
```

### Restart deployment

```bash
kubectl rollout restart deployment <deployment-name> -n <namespace>
```

### Roll back to previous version

```bash
kubectl rollout undo deployment <deployment-name> -n <namespace>
```

### Roll back to specific revision

```bash
kubectl rollout undo deployment <deployment-name> -n <namespace> --to-revision=<revision-number>
```

### Troubleshooting logic

- Check if the issue started after a deployment.
- Review rollout history.
- Check new pods vs old pods.
- Roll back only when approved or when it is the safest recovery option.

---

<a id="common-kubernetes-issues"></a>

## 20. Common Kubernetes Issues

### CrashLoopBackOff

The container starts, crashes, and Kubernetes keeps restarting it.

Commands:

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

Common causes:

```text
Application error
Bad environment variable
Missing ConfigMap
Missing Secret
Bad command or entrypoint
Failed health check
Permission issue
Dependency unavailable
```

---

### ImagePullBackOff / ErrImagePull

Kubernetes cannot pull the container image.

Commands:

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

Common causes:

```text
Wrong image name
Wrong image tag
Image does not exist
Private registry authentication issue
Network issue reaching registry
Image pull secret missing
```

---

### Pending Pod

The pod is waiting to be scheduled.

Commands:

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get nodes
kubectl describe node <node-name>
```

Common causes:

```text
Insufficient CPU
Insufficient memory
Node selector mismatch
Taints and tolerations issue
PVC not bound
No available nodes
```

---

### OOMKilled

The container was killed because it used too much memory.

Commands:

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous
kubectl top pod <pod-name> -n <namespace>
```

Common causes:

```text
Memory leak
Memory limit too low
Unexpected traffic spike
Large batch process
Inefficient application behavior
```

---

### Service Has No Endpoints

The service exists, but no pods are behind it.

Commands:

```bash
kubectl get svc <service-name> -n <namespace>
kubectl get endpoints <service-name> -n <namespace>
kubectl get pods -n <namespace> --show-labels
kubectl describe svc <service-name> -n <namespace>
```

Common causes:

```text
Service selector does not match pod labels
Pods are not ready
Readiness probe failing
Wrong namespace
Pods are not running
```

---

<a id="real-troubleshooting-scenarios"></a>

## 21. Real Troubleshooting Scenarios

### Scenario 1: Pod is in CrashLoopBackOff

Commands:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

What to check:

- Application error logs
- Environment variables
- ConfigMaps and Secrets
- Container command or entrypoint
- Health checks
- Recently deployed image

Possible fixes:

- Fix application configuration.
- Correct missing environment variables.
- Restore missing ConfigMap or Secret.
- Roll back the deployment.
- Fix image or entrypoint.

---

### Scenario 2: Application is not reachable through Service

Commands:

```bash
kubectl get svc -n <namespace>
kubectl describe svc <service-name> -n <namespace>
kubectl get endpoints <service-name> -n <namespace>
kubectl get pods -n <namespace> --show-labels
```

What to check:

- Service selector
- Pod labels
- Target port
- Pod readiness
- Namespace

Possible fixes:

- Correct service selector.
- Correct pod labels.
- Fix readiness probe.
- Correct service port or target port.

---

### Scenario 3: Ingress is not working

Commands:

```bash
kubectl get ingress -n <namespace>
kubectl describe ingress <ingress-name> -n <namespace>
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl logs -n <ingress-controller-namespace> <ingress-controller-pod>
```

What to check:

- DNS record
- Ingress host and path
- Backend service
- Backend port
- TLS secret
- Ingress controller logs

Possible fixes:

- Fix DNS.
- Correct ingress backend service.
- Correct service port.
- Fix TLS secret.
- Check ingress controller.

---

### Scenario 4: Pod is Pending

Commands:

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get nodes
kubectl describe node <node-name>
kubectl get pvc -n <namespace>
```

What to check:

- Node resources
- Taints and tolerations
- Node selectors
- PVC status
- Scheduling errors

Possible fixes:

- Reduce resource requests.
- Add capacity.
- Fix node selectors.
- Add tolerations if appropriate.
- Fix PVC/storage issue.

---

### Scenario 5: Deployment failed after a new release

Commands:

```bash
kubectl rollout status deployment <deployment-name> -n <namespace>
kubectl rollout history deployment <deployment-name> -n <namespace>
kubectl get pods -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```

What to check:

- New image version
- App logs
- Environment variables
- Config changes
- Health checks
- Dependencies

Possible fixes:

- Roll back deployment.
- Fix configuration.
- Update image tag.
- Fix health checks.
- Redeploy after correction.

---

### Scenario 6: Pod cannot pull image

Commands:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
kubectl get secrets -n <namespace>
```

What to check:

- Image name
- Image tag
- Registry URL
- ImagePullSecret
- Registry credentials
- Network access to registry

Possible fixes:

- Correct image name or tag.
- Create or update image pull secret.
- Reference `imagePullSecrets` in the Deployment.
- Validate registry credentials.
- Check network path to the registry.

---

### Scenario 7: TLS certificate issue

Commands:

```bash
kubectl get secrets -n <namespace>
kubectl describe secret <tls-secret-name> -n <namespace>
kubectl get secret <tls-secret-name> -n <namespace> -o jsonpath='{.data.tls\.crt}' | base64 -d > tls.crt
openssl x509 -in tls.crt -text -noout
openssl x509 -in tls.crt -noout -enddate
kubectl describe ingress <ingress-name> -n <namespace>
```

What to check:

- Certificate expiration date
- Certificate common name or SAN
- TLS secret name
- Ingress TLS configuration
- Hostname mismatch

Possible fixes:

- Renew certificate.
- Update TLS secret.
- Correct Ingress TLS reference.
- Correct DNS or hostname configuration.

---
