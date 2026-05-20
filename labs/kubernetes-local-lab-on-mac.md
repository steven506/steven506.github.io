# Kubernetes Local Lab on Mac Using kind

This lab documents how to build and troubleshoot a local Kubernetes cluster on macOS using `kind`, Docker, and `kubectl`.

The purpose of this lab is to practice Kubernetes administration, application deployment, service exposure, log analysis, troubleshooting, and operational workflows in a local environment.

---

## Table of Contents

- [1. Lab Objective](#lab-objective)
- [2. Lab Architecture](#lab-architecture)
- [3. Technologies Used](#technologies-used)
- [4. Prerequisites](#prerequisites)
- [5. Install Required Tools](#install-required-tools)
- [6. Create a Local Kubernetes Cluster](#create-a-local-kubernetes-cluster)
- [7. Validate the Cluster](#validate-the-cluster)
- [8. Configure Daily kubectl Shortcuts](#configure-daily-kubectl-shortcuts)
- [9. Deploy a Test Application](#deploy-a-test-application)
- [10. Expose the Application with a Service](#expose-the-application-with-a-service)
- [11. Test the Application with Port Forwarding](#test-the-application-with-port-forwarding)
- [12. Scale the Application](#scale-the-application)
- [13. Perform a Rolling Update](#perform-a-rolling-update)
- [14. Roll Back a Deployment](#roll-back-a-deployment)
- [15. Collect Logs and Events](#collect-logs-and-events)
- [16. Troubleshooting Scenario 1: ImagePullBackOff](#troubleshooting-scenario-1-imagepullbackoff)
- [17. Troubleshooting Scenario 2: CrashLoopBackOff](#troubleshooting-scenario-2-crashloopbackoff)
- [18. Troubleshooting Scenario 3: Service Has No Endpoints](#troubleshooting-scenario-3-service-has-no-endpoints)
- [19. Troubleshooting Scenario 4: Application Not Reachable](#troubleshooting-scenario-4-application-not-reachable)
- [20. Clean Up the Lab](#clean-up-the-lab)
---

<a id="lab-objective"></a>

## 1. Lab Objective

The objective of this lab is to create a local Kubernetes environment on macOS and practice the most common Kubernetes support and troubleshooting tasks.

By the end of this lab, I should be able to:

- Install and configure local Kubernetes tooling.
- Create a local Kubernetes cluster.
- Deploy applications.
- Expose applications using Services.
- Use `kubectl` to inspect cluster resources.
- Collect logs, events, and YAML output.
- Troubleshoot common pod and service issues.
- Document evidence for a technical portfolio.

---

<a id="lab-architecture"></a>

## 2. Lab Architecture

High-level architecture:

```text
macOS Host
│
├── Docker Desktop
│   │
│   └── kind Kubernetes Cluster
│       │
│       ├── Control Plane Node
│       │
│       └── Worker Node(s)
│
└── kubectl CLI
    │
    └── Manages Kubernetes resources
```

Application flow:

```text
User / Browser
│
└── localhost
    │
    └── kubectl port-forward
        │
        └── Kubernetes Service
            │
            └── Pod running containerized application
```

---

<a id="technologies-used"></a>

## 3. Technologies Used

- macOS
- Homebrew
- Docker Desktop
- kind
- kubectl
- Kubernetes
- NGINX container image
- Bash/Zsh terminal

`kind` is a tool for running local Kubernetes clusters using Docker container nodes. It is commonly used for local development and testing. Kubernetes also lists `kind` as one of the local Kubernetes options and notes that it requires Docker or Podman.  
Sources: [kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/), [Kubernetes Install Tools](https://kubernetes.io/docs/tasks/tools/)

---

<a id="prerequisites"></a>

## 4. Prerequisites

Before starting, confirm that the Mac has:

- Homebrew installed
- Docker Desktop installed and running
- Internet access
- Terminal access
- Basic command-line knowledge

Check Homebrew:

```bash
brew --version
```

Check Docker:

```bash
docker --version
docker ps
```

If `docker ps` fails, start Docker Desktop first.

---

<a id="install-required-tools"></a>

## 5. Install Required Tools

Install `kubectl`:

```bash
brew install kubectl
```

Validate:

```bash
kubectl version --client
```

Install `kind`:

```bash
brew install kind
```

Validate:

```bash
kind version
```

Optional tools:

```bash
brew install jq
brew install watch
```

Useful validation:

```bash
which kubectl
which kind
which docker
```

---

<a id="create-a-local-kubernetes-cluster"></a>

## 6. Create a Local Kubernetes Cluster

Create a basic cluster:

```bash
kind create cluster --name devops-lab
```

Expected result:

```text
Creating cluster "devops-lab" ...
Ensuring node image ...
Preparing nodes ...
Writing configuration ...
Starting control-plane ...
Installing CNI ...
Installing StorageClass ...
```

Check kind clusters:

```bash
kind get clusters
```

Check Docker containers created by kind:

```bash
docker ps
```

The cluster context should be created automatically.

Check current context:

```bash
kubectl config current-context
```

Expected context:

```text
kind-devops-lab
```

---

<a id="validate-the-cluster"></a>

## 7. Validate the Cluster

Check cluster information:

```bash
kubectl cluster-info
```

Check nodes:

```bash
kubectl get nodes
```

Check nodes with more details:

```bash
kubectl get nodes -o wide
```

Check all pods across all namespaces:

```bash
kubectl get pods -A
```

Check Kubernetes system components:

```bash
kubectl get pods -n kube-system
```

Describe the node:

```bash
kubectl describe node devops-lab-control-plane
```

Troubleshooting logic:

- If the node is not `Ready`, check Docker Desktop resources.
- If `kubectl` cannot connect, confirm the current context.
- If the cluster did not create, confirm Docker Desktop is running.

---

<a id="configure-daily-kubectl-shortcuts"></a>

## 8. Configure Daily kubectl Shortcuts

Create a short alias for `kubectl`.

For Zsh on macOS:

```bash
echo 'alias k=kubectl' >> ~/.zshrc
source ~/.zshrc
```

For Bash:

```bash
echo 'alias k=kubectl' >> ~/.bashrc
source ~/.bashrc
```

Test:

```bash
k get nodes
```

Enable autocomplete for Zsh:

```bash
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
echo 'alias k=kubectl' >> ~/.zshrc
echo 'compdef __start_kubectl k' >> ~/.zshrc
source ~/.zshrc
```

Useful daily aliases:

```bash
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
```

Check kubeconfig:

```bash
ls -l ~/.kube/config
kubectl config get-contexts
kubectl config current-context
```

Secure kubeconfig permissions:

```bash
chmod 600 ~/.kube/config
```

---

<a id="deploy-a-test-application"></a>

## 9. Deploy a Test Application

Create a namespace:

```bash
kubectl create namespace web-lab
```

Set it as the default namespace for the current context:

```bash
kubectl config set-context --current --namespace=web-lab
```

Deploy NGINX:

```bash
kubectl create deployment nginx-demo --image=nginx:latest
```

Check deployment:

```bash
kubectl get deployments
```

Check pods:

```bash
kubectl get pods -o wide
```

Describe the deployment:

```bash
kubectl describe deployment nginx-demo
```

Describe the pod:

```bash
kubectl describe pod <pod-name>
```

Check logs:

```bash
kubectl logs <pod-name>
```

Expected result:

```text
Deployment exists
Pod is Running
Container image is nginx:latest
```

---

<a id="expose-the-application-with-a-service"></a>

## 10. Expose the Application with a Service

Expose the deployment as a ClusterIP service:

```bash
kubectl expose deployment nginx-demo --port=80 --target-port=80 --name=nginx-demo-service
```

Check service:

```bash
kubectl get svc
```

Describe service:

```bash
kubectl describe svc nginx-demo-service
```

Check endpoints:

```bash
kubectl get endpoints nginx-demo-service
```

Expected result:

```text
The service should have endpoints pointing to the NGINX pod IP.
```

Troubleshooting logic:

- If endpoints are empty, check pod labels and service selectors.
- If the pod is not ready, check readiness/liveness probes.
- If the service points to the wrong port, check `targetPort`.

---

<a id="test-the-application-with-port-forwarding"></a>

## 11. Test the Application with Port Forwarding

Port forward local port `8080` to service port `80`:

```bash
kubectl port-forward svc/nginx-demo-service 8080:80
```

Open another terminal and test:

```bash
curl -I http://localhost:8080
```

Expected result:

```text
HTTP/1.1 200 OK
Server: nginx
```

Open in browser:

```text
http://localhost:8080
```

Stop port forwarding with:

```text
CTRL + C
```

Troubleshooting logic:

- If port-forward fails, check service name.
- If curl fails, check pod status.
- If service exists but has no endpoints, check selector labels.

---

<a id="scale-the-application"></a>

## 12. Scale the Application

Scale the deployment to 3 replicas:

```bash
kubectl scale deployment nginx-demo --replicas=3
```

Check pods:

```bash
kubectl get pods -o wide
```

Check deployment:

```bash
kubectl get deployment nginx-demo
```

Check endpoints:

```bash
kubectl get endpoints nginx-demo-service
```

Expected result:

```text
Three NGINX pods should be running.
The service should have three endpoints.
```

Troubleshooting logic:

- If not all pods run, describe the failed pod.
- Check events.
- Check node resources.

---

<a id="perform-a-rolling-update"></a>

## 13. Perform a Rolling Update

Update the image:

```bash
kubectl set image deployment/nginx-demo nginx=nginx:1.25
```

Check rollout status:

```bash
kubectl rollout status deployment/nginx-demo
```

Check rollout history:

```bash
kubectl rollout history deployment/nginx-demo
```

Check pods:

```bash
kubectl get pods
```

Describe deployment:

```bash
kubectl describe deployment nginx-demo
```

Troubleshooting logic:

- If rollout is stuck, check pods.
- If new pods fail, check `kubectl describe pod`.
- Check logs and events.
- Roll back if needed.

---

<a id="roll-back-a-deployment"></a>

## 14. Roll Back a Deployment

Roll back to the previous version:

```bash
kubectl rollout undo deployment/nginx-demo
```

Check rollout status:

```bash
kubectl rollout status deployment/nginx-demo
```

Check deployment image:

```bash
kubectl describe deployment nginx-demo
```

Expected result:

```text
Deployment should return to the previous image version.
```

---

<a id="collect-logs-and-events"></a>

## 15. Collect Logs and Events

Create an evidence folder locally:

```bash
mkdir -p ~/k8s-lab-evidence
```

Save pod list:

```bash
kubectl get pods -o wide > ~/k8s-lab-evidence/pods.txt
```

Save services:

```bash
kubectl get svc -o wide > ~/k8s-lab-evidence/services.txt
```

Save events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp > ~/k8s-lab-evidence/events.txt
```

Save deployment YAML:

```bash
kubectl get deployment nginx-demo -o yaml > ~/k8s-lab-evidence/nginx-demo-deployment.yaml
```

Save service YAML:

```bash
kubectl get svc nginx-demo-service -o yaml > ~/k8s-lab-evidence/nginx-demo-service.yaml
```

Save pod logs:

```bash
POD_NAME=$(kubectl get pods -l app=nginx-demo -o jsonpath='{.items[0].metadata.name}')
kubectl logs "$POD_NAME" > ~/k8s-lab-evidence/nginx-demo-pod.log
```

List evidence:

```bash
ls -lah ~/k8s-lab-evidence
```

Important: Review files before committing them to GitHub. Do not commit secrets, tokens, private keys, or sensitive kubeconfig files.

---

<a id="troubleshooting-scenario-1-imagepullbackoff"></a>

## 16. Troubleshooting Scenario 1: ImagePullBackOff

Create a broken deployment with a fake image:

```bash
kubectl create deployment broken-image --image=nginx:this-tag-does-not-exist
```

Check pod status:

```bash
kubectl get pods
```

Expected issue:

```text
ImagePullBackOff
```

Investigate:

```bash
kubectl describe pod <broken-pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

What to look for:

```text
Failed to pull image
Image not found
ErrImagePull
Back-off pulling image
```

Fix it:

```bash
kubectl set image deployment/broken-image nginx=nginx:latest
```

Validate:

```bash
kubectl rollout status deployment/broken-image
kubectl get pods
```

Clean up:

```bash
kubectl delete deployment broken-image
```

Documented root cause:

```text
The pod failed because the container image tag did not exist. Kubernetes could not pull the image from the container registry.
```

---

<a id="troubleshooting-scenario-2-crashloopbackoff"></a>

## 17. Troubleshooting Scenario 2: CrashLoopBackOff

Create a pod that exits immediately:

```bash
kubectl run crash-demo --image=busybox --restart=Never -- /bin/sh -c "exit 1"
```

Check pod:

```bash
kubectl get pods
```

Describe pod:

```bash
kubectl describe pod crash-demo
```

Check logs:

```bash
kubectl logs crash-demo
```

Note: Because this is a standalone pod with `restart=Never`, it may show `Error` instead of `CrashLoopBackOff`.

To create a real CrashLoopBackOff using a Deployment:

```bash
kubectl create deployment crashloop-demo --image=busybox -- /bin/sh -c "exit 1"
```

Check:

```bash
kubectl get pods
```

Investigate:

```bash
kubectl describe pod <crashloop-pod-name>
kubectl logs <crashloop-pod-name>
kubectl logs <crashloop-pod-name> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
```

Fix it by changing the command:

```bash
kubectl delete deployment crashloop-demo
kubectl create deployment crashloop-demo --image=busybox -- /bin/sh -c "sleep 3600"
```

Validate:

```bash
kubectl get pods
```

Clean up:

```bash
kubectl delete deployment crashloop-demo
kubectl delete pod crash-demo
```

Documented root cause:

```text
The container repeatedly exited with code 1, causing Kubernetes to restart it until it entered CrashLoopBackOff.
```

---

<a id="troubleshooting-scenario-3-service-has-no-endpoints"></a>

## 18. Troubleshooting Scenario 3: Service Has No Endpoints

Create a service with a selector that does not match any pod:

```bash
kubectl create service clusterip broken-service --tcp=80:80
```

Edit the service selector:

```bash
kubectl edit svc broken-service
```

Set selector to something that does not match existing pods:

```yaml
selector:
  app: does-not-exist
```

Check endpoints:

```bash
kubectl get endpoints broken-service
```

Expected result:

```text
No endpoints
```

Investigate:

```bash
kubectl describe svc broken-service
kubectl get pods --show-labels
```

Fix by matching the selector to the NGINX pod label.

Check labels:

```bash
kubectl get pods --show-labels
```

Edit service:

```bash
kubectl edit svc broken-service
```

Set selector:

```yaml
selector:
  app: nginx-demo
```

Validate:

```bash
kubectl get endpoints broken-service
```

Clean up:

```bash
kubectl delete svc broken-service
```

Documented root cause:

```text
The service had no endpoints because the service selector did not match any pod labels.
```

---

<a id="troubleshooting-scenario-4-application-not-reachable"></a>

## 19. Troubleshooting Scenario 4: Application Not Reachable

Problem:

```text
The application is expected to respond through localhost, but curl fails.
```

Commands to investigate:

```bash
kubectl get pods
kubectl get svc
kubectl get endpoints nginx-demo-service
kubectl describe svc nginx-demo-service
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl port-forward svc/nginx-demo-service 8080:80
curl -v http://localhost:8080
```

Troubleshooting logic:

```text
1. Confirm pod is Running.
2. Confirm service exists.
3. Confirm service has endpoints.
4. Confirm targetPort matches container port.
5. Confirm port-forward is running.
6. Test localhost with curl.
7. Check logs if HTTP response is not expected.
```

Possible root causes:

```text
Pod is not running
Service selector mismatch
Wrong service port
Wrong target port
Port-forward not running
Local port already in use
Application inside container is not responding
```

Possible fixes:

```text
Restart or fix pod
Correct service selector
Correct targetPort
Use a different local port
Check application logs
Redeploy application
```

---
<a id="clean-up-the-lab"></a>

## 20. Clean Up the Lab

Delete the namespace and all resources inside it:

```bash
kubectl delete namespace web-lab
```

Delete the kind cluster:

```bash
kind delete cluster --name devops-lab
```

Validate:

```bash
kind get clusters
docker ps
```

---
