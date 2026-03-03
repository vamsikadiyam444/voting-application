# Voting App — CI/CD with Azure DevOps, AKS & ArgoCD

# Tech stack

```bash
- Azure Kubernetes Service (AKS)
- Azure Container Registry (ACR)
- Azure DevOps Pipelines
- ArgoCD (GitOps)
- Docker
- Kubernetes
- Node.js (Result App)
- Python (Vote App)
- PostgreSQL
- Redis
```

# Step1 Create A AKS cluster and resource Group  

```bash
# Create resource group
az group create --name azurecicd --location switzerlandnorth

# Create AKS cluster
az aks create \
  --resource-group azurecicd \
  --name azurecicd \
  --node-count 1 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get credentials
az aks get-credentials --resource-group azurecicd --name azurecicd

# Verify
kubectl get nodes
```

# Step-2 Create Azure Container Registry 

```bash
# Create ACR
az acr create \
  --resource-group azurecicd \
  --name azurecicd1 \
  --sku Basic

# Attach ACR to AKS (allows AKS to pull images)
az aks update \
  --name azurecicd \
  --resource-group azurecicd \
  --attach-acr azurecicd1
```

# Step-3 Install ArgoCD on AKS

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Expose ArgoCD server as NodePort
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "NodePort"}}'

# Get ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Check ArgoCD pods
kubectl get pods -n argocd

# Check ArgoCD services
kubectl get svc -n argocd
```

# Step-4 Useful Commands 

```bash
# Check all pods
kubectl get pods -n argocd

# Check all services
kubectl get svc -n argocd

# Check node IP
kubectl get nodes -o wide

# Describe failing pod
kubectl describe pod <pod-name> -n argocd

# Check pod logs
kubectl logs <pod-name> -n argocd

# Restart deployments (one time fix)
kubectl rollout restart deployment vote -n argocd
kubectl rollout restart deployment result -n argocd
kubectl rollout restart deployment worker -n argocd

# Check ArgoCD application status
kubectl describe application -n argocd
```

# Step-5 Access the Application

```bash
# Get node public IP
kubectl get nodes -o wide

# Access via browser
Vote App    → http://<NODE-IP>:31000
Result App  → http://<NODE-IP>:31001
ArgoCD UI   → https://<NODE-IP>:30868
```

# Step-6 To Delete the resources 

```bash
# Delete everything
az group delete -n azurecicd --yes --no-wait
```
## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

