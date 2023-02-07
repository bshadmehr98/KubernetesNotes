1. Install docker
2. make sure that machines can talk to each other and ports are open
3. Install CRI(Container runtime interface) for docker - https://github.com/Mirantis/cri-dockerd
```
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest|grep tag_name | cut -d '"' -f 4|sed 's/v//g')

### For Intel 64-bit CPU ###
wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz
tar xvf cri-dockerd-${VER}.amd64.tgz

### For ARM 64-bit CPU ###
wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.arm64.tgz
cri-dockerd-${VER}.arm64.tgz

sudo mv cri-dockerd/cri-dockerd /usr/local/bin/

# Configure systemd units for cri-dockerd
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service

sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket

```

4. Installing kubeadm, kubelet and kubectl - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

5. Pulling images before start
```
kubeadm config images pull --cri-socket=unix:///var/run/cri-dockerd.sock
```

6. Initialising the manager
```
kubeadm init --pod-network-cidr=20.96.0.0/12 --cri-socket=unix:///var/run/cri-dockerd.sock
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

7. It should give you something like this
```
kubeadm join 192.168.1.237:6443 --token 2ohnrk.wzi2gazp2nf935pp \
        --discovery-token-ca-cert-hash sha256:0509a8e5c2e866a307019a34890b775e1e5a9f4282a76487bec3a78528fed451 --cri-socket=unix:///var/run/cri-dockerd.sock
```

8. Install calico (Only on master node)
```
# Pull images
docker pull docker.io/calico/node:v3.25.0
docker pull docker.io/calico/cni:v3.25.0
docker pull docker.io/calico/kube-controllers:v3.25.0
```

9. Setup bash completion
```
# Install bash completion
source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.

# Alias kubectl to k
alias k=kubectl
complete -o default -F __start_kubectl k
```

10. Set a label for nodes
```
 k label nodes {node-name} role=worker
```

Useful tips:
```
# Get the system architecture
dpkg --print-architecture

# get kubectl api versions for each object
kubectl api-resources
```

---
Questions:
- [ ]  What is apt-mark hold
- [ ] How to select value for --pod-network-cidr and what exactly is it?
- [x] What is CA
- [ ] What is PKI
- [x] What is manifest
