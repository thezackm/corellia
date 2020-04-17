# Installation on Ubuntu 
*Last Edited: 04/16/2020*

## Base Installation

### Install Docker
`sudo apt install docker.io -y`

### Setup daemon 
```
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

### Enable Docker service on startup
`sudo systemctl enable docker`

### Add Key for Kubernetes Repository
`curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`

### Manually add repository by creating kubernetes.list and adding the repo
`sudo vim /etc/apt/sources.list.d/kubernetes.list`

Past the following into VIM:

`deb http://apt.kubernetes.io/ kubernetes-xenial main`

### Update Repos and Install Kubernetes
`sudo apt-get update && sudo apt install kubeadm -y`

### Disable SWAP
`swapoff -a`

## Master Node

### Initialize Kubernetes
`sudo kubeadm init --pod-network-cidr=10.244.0.0/16`

Ensure you capture the `kubeadm join` command for use later.

Ex: 
```
kubeadm join 10.2.0.9:6443 --token ki7cmd.lbcoynwhxjls39up \
    --discovery-token-ca-cert-hash sha256:a5a1cc2f35c16cab3d23d133f624e978a7d81650c88315a4c4fc454d935c4804
```
### Setup user access

`mkdir -p $HOME/.kube`

`sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`

`sudo chown $(id -u):$(id -g) $HOME/.kube/config`

### Deploy a Pod Network
*Note there are [multiple CNI providers to choose from](https://rancher.com/blog/2019/2019-03-21-comparing-kubernetes-cni-providers-flannel-calico-canal-and-weave/). We're using flannel here.*

`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

#### Validate the pods are ready
`kubectl get pods --all-namespaces`


## Worker Node 

### Join the Kubernetes cluster
*using the `kubeadm join` command output after initializing the cluster or you can find it again by running this:*

`kubeadm token create --print-join-command` 

```
sudo kubeadm join 10.2.0.9:6443 --token 3zoly5.7z1bttqmps5n0zm6 --discovery-token-ca-cert-hash sha256:4b88c57b4c0fd7fc62d3ddf388cecfbeea2c2c25c5f7f54a7ae17cb46926d630
```

## OPTIONAL NGINX INSTALL

From the Master Node: 
```
kubectl run --image=nginx nginx-server --port=80 --env="DOMAIN=cluster"
kubectl create deployment nginx-server --image=nginx
kubectl expose deployment nginx-server --port=80 --name=nginx-http
```

Validation on the Worker Node: `sudo docker ps`

Validation on the Master Node: `kubectl get svc`
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   20m
nginx-http   ClusterIP   10.108.188.90   <none>        80/TCP    2m1s
```

`curl -I 10.108.188.90`
```
HTTP/1.1 200 OK
Server: nginx/1.17.9
Date: Thu, 16 Apr 2020 22:45:15 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 03 Mar 2020 14:32:47 GMT
Connection: keep-alive
ETag: "5e5e6a8f-264"
Accept-Ranges: bytes
```
