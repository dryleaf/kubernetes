## Servers
**dev**
```
C:\Users\esjopol>ssh -l osboxes -p 2222 -- localhost
```

**cp**
```
C:\Users\esjopol\source\repos>ssh -l student -p 2201 -- localhost
...
...
Last login: Thu Jul 13 06:28:59 2023 from 10.10.10.5
student@cp:~$
```

**worker**
```
C:\Users\esjopol\source\repos>ssh -l student -p 2202 -- localhost
...
...
Last login: Thu Jul 13 06:28:59 2023 from 10.10.10.4
student@worker:~$
```

## Two nodes cluster

## Install with calico cluster networking
[kubernetes install](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)

### cp
**1. kubeadm init**
```
$ sudo kubeadm init
...
...
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.10.10.4:6443 --token f0mkqd.dpmkili7mw3pcnxz \
        --discovery-token-ca-cert-hash sha256:c25f966e601f639ccc1d2cb3abdeaf13b3cd4eee8d815d4189be8c542e2b0664
```

**Install helm**
```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

**Install kubeview**
[kubeview](https://github.com/benc-uk/kubeview)

**dashboard**
Go to [kubernetes dashboard on github](https://github.com/kubernetes/dashboard), then install using the following commands:
```
# Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

You can access the dashboard via [kubernetes-dashboard](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)

#### Troubleshooting
**container runtime is not running**
```
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR CRI]: container runtime is not running: output: time="2023-07-21T03:56:44Z" level=fatal msg="validate service connection: CRI v1 runtime API is not implemented for endpoint \"unix:///var/run/containerd/containerd.sock\": rpc error: code = Unimplemented desc = unknown service runtime.v1.RuntimeService"
, error: exit status 1
```
**solution**
```
student@cp:~$ sudo rm /etc/containerd/config.toml
student@cp:~$ sudo systemctl restart containerd
student@cp:~$ sudo kubeadm init
```

## minikube
**dashboard**
```
minikube dashboard --url
```
↓
```
http://127.0.0.1:44875/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

**service**
```
minikube service list
minikube service hello-node
```

**addons**
```
minikube addons list
minikube addons enable ingress
minikube addons disable ingress
```

**stop**
```
minikube stop
```

**delete**
```
minikube delete
```

## kubectl

**App deployment**
[Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo)

```
kubectl port-forward services/frontend-external --address 0.0.0.0 8080:80
```

**Port forwarding**
```
kubectl port-forward service/myservice --address 0.0.0.0 8080:80
kubectl -n kubernetes-dashboard  port-forward services/kubernetes-dashboard --address 0.0.0.0 8080:80
 kubectl port-forward services/hello-node --address 0.0.0.0 8080:8080
```
↓
```
http://localhost:8080
```

**Get**
```
kubectl get pods
kubectl get pods -w
kubectl get events -w
kubectl get pod,svc -n kube-system
```

**Create**
```
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
```

**Expose**
```
 kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```

**Config**
```
kubectl config view
```

**delete**
```
kubectl delete service hello-node
kubectl delete deployment hello-node
```