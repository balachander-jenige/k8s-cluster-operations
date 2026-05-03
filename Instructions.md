# Kubernetes Local Practice Setup (WSL2 + Ubuntu 24.04)
 
A step-by-step guide to set up a local Kubernetes practice environment on Windows using WSL2, Docker, kubectl, kind, and kubeadm — tailored for CKA exam preparation.
 
---
 
## Prerequisites
 
- Windows 10/11 with WSL2 enabled
- Ubuntu 24.04.1 LTS installed from Microsoft Store
- Internet connection
---
 
## Step 1 — Enable systemd in WSL
 
Systemd is required for Docker Engine to run properly.
 
Open your **WSL Ubuntu terminal** and run:
 
```bash
sudo nano /etc/wsl.conf
```
 
Add the following content:
 
```ini
[boot]
systemd=true
```
 
Save with `Ctrl+O` → `Enter`, then exit with `Ctrl+X`.
 
Now open **Windows PowerShell** (not WSL) and run:
 
```powershell
wsl --shutdown
```
 
Reopen Ubuntu, then verify systemd is running:
 
```bash
systemctl --version
```
 
> ✅ You should see `systemd 255` or higher.
 
---
 
## Step 2 — Update System Packages
 
```bash
sudo apt-get update && sudo apt-get upgrade -y
```
 
---
 
## Step 3 — Install Docker Engine
 
### 3a. Install prerequisites
 
```bash
sudo apt-get install -y ca-certificates curl gnupg
```
 
### 3b. Add Docker's GPG key
 
```bash
sudo install -m 0755 -d /etc/apt/keyrings
 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
 
### 3c. Add Docker repository
 
> ⚠️ Uses hardcoded `noble` to avoid variable expansion issues in WSL terminals.
 
```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list
```
 
Verify the file looks correct:
 
```bash
cat /etc/apt/sources.list.d/docker.list
```
 
Expected output:
```
deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble stable
```
 
### 3d. Install Docker
 
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```
 
### 3e. Allow running Docker without sudo
 
```bash
sudo usermod -aG docker $USER
newgrp docker
```
 
### 3f. Verify Docker works
 
```bash
docker run hello-world
```
 
> ✅ You should see `Hello from Docker!`
 
---
 
## Step 4 — Install kubectl
 
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
 
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
 
sudo apt-get update
sudo apt-get install -y kubectl
```
 
Verify:
 
```bash
kubectl version --client
```
 
> ✅ Should show `Client Version: v1.31.x`
 
---
 
## Step 5 — Install kind
 
> ⚠️ Using `v0.24.0` — more stable download than `v0.23.0`.
 
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```
 
---
 
## Step 6 — Install kubeadm
 
kubeadm is already available from the Kubernetes repo added in Step 4:
 
```bash
sudo apt-get install -y kubeadm
kubeadm version
```
 
> 📝 You need to know `kubeadm init`, `kubeadm join`, and `kubeadm token create` for the CKA exam.
 
---
 
## Step 7 — Set Up Shell Shortcuts (CKA Exam Essentials)
 
### Configure vim for YAML editing
 
```bash
cat >> ~/.vimrc << 'EOF'
set expandtab
set tabstop=2
set shiftwidth=2
set autoindent
EOF
```
 
### Add kubectl aliases to ~/.bashrc
 
```bash
cat >> ~/.bashrc << 'EOF'
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
EOF
 
source ~/.bashrc
```
 
### Test shortcuts
 
```bash
k run nginx --image=nginx $do    # generates YAML without creating pod
k get nodes                      # alias works
```
 
---
 
## Step 8 — Create a Practice Cluster
 
### Option A — Single-node cluster (quick, 30 seconds)
 
Good for most CKA topics: deployments, services, RBAC, configmaps, ingress, storage.
 
```bash
kind create cluster --name cka-practice
kubectl get nodes
```
 
### Option B — Multi-node cluster (1 control plane + 2 workers)
 
Required for: node affinity, taints/tolerations, drain/cordon, DaemonSets, pod anti-affinity.
 
```bash
cat > cka-cluster.yaml << 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF
 
kind create cluster --config cka-cluster.yaml --name cka-practice
```
 
Verify all nodes are ready (may take ~60 seconds):
 
```bash
kubectl get nodes
```
 
> ✅ All nodes should show `STATUS=Ready`
 
---
 
## Daily Practice Workflow
 
```bash
# Start a fresh cluster
kind create cluster --name cka-practice
 
# Practice your topic (networking, storage, scheduling, etc.)
kubectl run nginx --image=nginx
kubectl get pods
 
# Tear down when done
kind delete cluster --name cka-practice
```
 
> 💡 Destroying and recreating clusters is intentional — the CKA exam expects you to work fast under pressure.
 
---
 
## Quick Reference — Most-Used CKA Commands
 
```bash
# Generate YAML skeletons (never write from scratch in the exam)
kubectl run pod1 --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment dep1 --image=nginx --replicas=3 --dry-run=client -o yaml
 
# Troubleshooting
kubectl describe node <node>
kubectl logs <pod> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
 
# Context switching (multi-cluster exam scenario)
kubectl config get-contexts
kubectl config use-context <context-name>
 
# Force delete a stuck pod
kubectl delete pod <pod> --force --grace-period 0
```
 
---
 
## Managing Docker Service
 
Docker runs as a background systemd service. For CKA practice, leave it running.
 
```bash
# Check status
sudo systemctl status docker
 
# Stop Docker (free up resources)
sudo systemctl stop docker
 
# Start Docker again
sudo systemctl start docker
```
 
> 💡 Docker itself uses almost no resources when idle. Only kind clusters consume RAM/CPU — and those are wiped with `kind delete cluster`.
 
---
 
## Troubleshooting
 
### Docker packages not found during install
 
The Docker GPG key or repo file was not written correctly. Reset and redo Step 3:
 
```bash
# Remove broken files
sudo rm -f /etc/apt/sources.list.d/docker.list
sudo rm -f /etc/apt/keyrings/docker.gpg
 
# Re-run Steps 3b → 3d
```
 
### Pod stuck in Pending
 
You likely have no nodes. Create a cluster first:
 
```bash
kind create cluster --name cka-practice
kubectl get nodes
```
 
### kind cluster creation times out
 
Docker daemon may not be running:
 
```bash
sudo systemctl start docker
kind create cluster --name cka-practice
```
 
---
 
## What Each Tool Does
 
| Tool | Purpose |
|------|---------|
| **Docker** | Runs kind cluster nodes as containers |
| **kubectl** | CLI to interact with any Kubernetes cluster |
| **kind** | Creates local Kubernetes clusters using Docker |
| **kubeadm** | Used in real cluster bootstrapping — exam knowledge required |
 
---
 
## References
 
- [kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [kubectl Install Docs](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Docker Install for Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [CKA Exam Curriculum](https://github.com/cncf/curriculum)
