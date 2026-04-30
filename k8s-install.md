![banner](https://imgur.com/H3ZyZw2.png)

# 🧩 **Kubernetes Installation Guide (2025 Update)**

By **[H A R S H H A A](https://www.github.com/NotHarshhaa)**

---

Master + Worker Nodes
## **Step 1: Update Your System**

Update the package list and upgrade installed packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

---

Master + Worker Nodes
## **Step 2: Install Dependencies**

Install required tools and packages:

```bash
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

---

Master + Worker Nodes
## **Step 3: Disable Swap**

Kubernetes requires swap to be disabled:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

---

Master + Worker Nodes
## **Step 4: Install Docker or containerd**

Kubernetes needs a container runtime. We'll use **containerd** (recommended).
Master + Worker Nodes
### **4.1: Install containerd**

```bash
sudo apt install -y containerd
```
Master + Worker Nodes
### **4.2: Configure containerd**

Generate the default configuration file:

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Restart containerd:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---
Master + Worker Nodes
## **Step 5: Install Kubernetes Components**

Install **kubeadm**, **kubelet**, and **kubectl**.

### ⚠️ Note

The old repository (`https://apt.kubernetes.io kubernetes-xenial`) is **deprecated**.
Use the new `pkgs.k8s.io` repository instead.
Master + Worker Nodes
### **5.1: Add the Kubernetes APT Repository (New Method)**

```bash
# Create a keyring directory
sudo mkdir -p /etc/apt/keyrings

# Download and add the Kubernetes repository GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update package list
sudo apt update
```

> 💡 *If a newer version (like v1.31) is available, replace `v1.30` in the URL above.*
Master + Worker Nodes
### **5.2: Install kubeadm, kubelet, and kubectl**

```bash
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

If you see errors about Snap versions, remove them first:

```bash
sudo snap remove kubelet kubectl kubeadm
```

---
Master Nodes
## **Step 6: Initialize the Kubernetes Cluster**

Run the following command on the **control-plane (master) node**:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

### **6.1: Configure kubectl for the Current User**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### **6.2: Save the Join Command**

The output of `kubeadm init` will include a `kubeadm join` command for worker nodes.
Copy and save this for later.

---

Master Node

## **Step 7: Install a Pod Network Add-On**

Choose a CNI (Container Network Interface). We'll use **Calico**.

### **7.1: Apply the Calico Manifest**

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---
Master Node
## **Step 8: Add Worker Nodes**

Run the `kubeadm join` command from Step 6.2 on each worker node:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

---
Master Node
## **Step 9: Verify the Cluster**

### **9.1: Check Nodes**

On the master node, check the status of nodes:

```bash
kubectl get nodes
```

### **9.2: Check Pods**

Verify that the cluster is working by checking pods in the `kube-system` namespace:

```bash
kubectl get pods -n kube-system
```

---

## **Optional: Enable Autocompletion for kubectl**

```bash
sudo apt install -y bash-completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
source ~/.bashrc
```

---

## **Troubleshooting Tips**

### 1. **Reset the Cluster**

If the installation fails, reset kubeadm and try again:

```bash
sudo kubeadm reset -f
sudo rm -rf $HOME/.kube
```

### 2. **Check Logs**

Use the following commands to troubleshoot issues:

```bash
journalctl -xeu kubelet
kubectl describe pod <pod-name> -n kube-system
```

---

✅ **This guide is now compatible with modern Ubuntu (20.04, 22.04, 24.04) and Kubernetes v1.30+ repositories.**
