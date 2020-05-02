Kubernetes
## Kubernetes multinode setup 
###  we have 2 machines, 1 master and 1 worker
## Pre-requisite 

### Disable selinux on all the nodes

```
  [root@master ~]# setenforce  0
  [root@master ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/'  /etc/selinux/config
  
 ```
 
 ### Enable the kernel bridge for every system on all the nodes
 ```
 [root@master ~]# modprobe br_netfilter
 [root@master ~]# echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
 ```
 ### Disable the swap on all the nodes
 ```
 [root@master ~]# swapoff  -a
 ```
 ## Installing kubeadm on all the nodes 
 ```
 [root@master ~]# yum  install kubeadm  -y
 ```
 ## if kubeadm is not present in your repo 
 you can browse this link [kubernetes repo](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)  <br/>
 
## yum can be configure by running this command 
```
cat  <<EOF  >/etc/yum.repos.d/kube.repo
[kube]
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
gpgcheck=0
EOF
```
 
 ## Enable kubelet service on all the nodes 
 ```
 [root@master ~]# systemctl enable --now kubelet
 ```
 ## Do this only on Kubernetes Master 
 We are here using Calico Networking so we need to pass some parameter 
 you can start [Kubernetes_networking](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) from this  <br/>
 
```
[root@master ~]# kubeadm  init --pod-network-cidr=192.168.0.0/16 --piserver-advertise-address=192.168.122.1
```

## this is optional 
### In case of cloud like aws , azure if want to bind public with certificate of kubernetes 
```
[root@master ~]# kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=0.0.0.0   --apiserver-cert-extra-sans=publicip,privateip,serviceip
```

### Use the output of above command and paste on all the worker nodes
```
[root@master ~]# kubeadm join 192.168.122.1:6443 --token 75ns2m.3p3zfeypoles00ex  --discovery-token-ca-cert-hash sha256:0b98dfae74f00384d456704f494ef92b22cafe96852e935565ca8133b674fce6
```

## Do this step in master node 
```
[root@master ~]# mkdir -p $HOME/.kube
[root@master ~]#  cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```

##  Now apply calico project 
```
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```
After this all nodes will be ready in state

## Now you can check the nodes status
```
[root@lab ~]# kubectl get nodes
NAME                 STATUS   ROLES    AGE    VERSION
lab.example.com      Ready    master   25m    v1.18.2
worker.example.com   Ready    <none>   106s   v1.18.2
```
## Now label the worker node with worker role
```
[root@lab ~]# kubectl label node worker.example.com node-role.kubernetes.io/worker=worker
node/worker.example.com labeled
```

## Now check the all the nodes

```
[root@lab ~]# kubectl get nodes
NAME                 STATUS   ROLES    AGE    VERSION
lab.example.com      Ready    master   25m    v1.18.2
worker.example.com   Ready    worker   2m9s   v1.18.2
```
