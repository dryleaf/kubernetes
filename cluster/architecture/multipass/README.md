## Search an image
```
multipass find
```

## Launch an instance
```
multipass launch jammy -n cp --cloud-init cloud-init-cp.yaml
```

```
multipass launch jammy -n worker01 --cloud-init cloud-init-worker01.yaml
```

## Setting cpus|disk|memory
**CP**
```
multipass set local.cp.cpus=2
multipass set local.cp.memory=2G
```

check:
```
multipass get local.cp.cpus
multipass get local.cp.memory
```


**Worker**
```
multipass set local.worker01.cpus=3
multipass set local.worker01.memory=4G
```

check:
```
multipass get local.worker01.cpus
multipass get local.worker01.memory
```

### Confirm cpus and memory
**CPU**
```
lscpu
sudo lshw -class processor | grep "core"
```

**MEMORY**
```
free -h
```

## Kubernetes cluster
```
sudo swapoff -a  
sudo sed -i '/ swap / s/^/#/' /etc/fstab

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

lsmod | grep br_netfilter
lsmod | grep overlay

sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

mkdir Software
cd Software/

wget https://github.com/containerd/containerd/releases/download/v1.7.8/containerd-1.7.8-linux-amd64.tar.gz

ls -l 
sudo tar Cxzvf /usr/local containerd-1.7.8-linux-amd64.tar.gz 

sudo mkdir -p /etc/containerd
sudo sh -c '/usr/local/bin/containerd config default > /etc/containerd/config.toml'

ls -l /etc/containerd/config.toml 

sudo mkdir -p /usr/local/lib/systemd/system

wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

sudo mv containerd.service /usr/local/lib/systemd/system/
sudo systemctl daemon-reload 
sudo systemctl enable --now containerd

wget https://github.com/opencontainers/runc/releases/download/v1.1.10/runc.amd64

sudo install -m 755 runc.amd64 /usr/local/sbin/runc
ls -l /usr/local/sbin/runc

wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz

sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.3.0.tgz 

ps -ef | grep containerd

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl stop kubelet
sudo systemctl enable --now kubelet
sudo systemctl daemon-reload
sudo systemctl status kubelet

sudo systemctl restart containerd.service 
sudo systemctl restart kubelet.service 
sudo systemctl status kubelet.service 
```

## CP node
```
sudo kubeadm init --cri-socket unix:///var/run/containerd/containerd.sock --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get pods -A
- In case of error like 'E1116 18:38:16.987160   37494 memcache.go:265] couldn't get current server API group list: Get "https://10.196.37.226:6443/api?timeout=32s": dial tcp 10.196.37.226:6443: connect: connection refused' just restart containerd service.
```
**Deply a pod network to the cluster**
You should now deploy a pod network to the cluster.
Run `kubectl apply -f [podnetwork].yaml` with one of the options listed at:
- https://kubernetes.io/docs/concepts/cluster-administration/addons/

Install Flannel:
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

**Get the nodes**
```
kubectl get nodes
kubectl get nodes -o wide
```

After joining the worker nodes, 
**Assign a role to a worker node**
```
kubectl label node worker01 node-role.kubernetes.io/worker=worker
```

**kubectl autocomplete function**
```
source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
```

## Worker nodes
```
sudo kubeadm join 10.196.37.226:6443 --token rzyiml.t1vtn03kzzbhwsxk --discovery-token-ca-cert-hash sha256:a3be2bc0d5cf82cd72d50c70cb6400637a8cc6fa567726697149c93f446b7f04 --cri-socket unix:///var/run/containerd/containerd.sock
```

## Local kubectl context
```
cat .kube/config

echo "<base64-encoded-data>" | base64 -d

kubectl config set-cluster multipass --server=https://10.196.37.226:6443 --certificate-authority=/home/dryleaf/.config/multipass/ca.crt

kubectl config set-credentials multipass --client-certificate=/home/dryleaf/.config/multipass/client.crt --client-key=/home/dryleaf/.config/multipass/client.key

kubectl config set contexts.multipass.namespace dev --set-raw-bytes=true

kubectl config set-context multipass --cluster=multipass --namespace=dev

○─▪ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/dryleaf/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Mon, 18 Sep 2023 12:30:00 JST
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
- cluster:
    certificate-authority: /home/dryleaf/.config/multipass/ca.crt
    server: https://10.196.37.226:6443
  name: multipass
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Mon, 18 Sep 2023 12:30:00 JST
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
- context:
    cluster: multipass
    namespace: dev
    user: multipass
  name: multipass
current-context: multipass
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/dryleaf/.minikube/profiles/minikube/client.crt
    client-key: /home/dryleaf/.minikube/profiles/minikube/client.key
- name: multipass
  user:
    client-certificate: /home/dryleaf/.config/multipass/client.crt
    client-key: /home/dryleaf/.config/multipass/client.key
```

## CP Issues
**CrashLoopBackOff - After update to containerd**
https://github.com/containerd/containerd/issues/6009#issuecomment-1121672553

**Expired tokens**
```
sudo kubeadm token create --print-join-command
```

