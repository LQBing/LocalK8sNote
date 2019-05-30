# install k8s

## ALERTS

DO NOT USE DYNAMIC MEMORY AS A VIRTUAL MACHINE

PLEASE TURN OFF SWAP

## reference

https://kubernetes.io/docs/setup/independent/install-kubeadm/

https://blog.csdn.net/nklinsirui/article/details/80581286

https://github.com/kubernetes/dashboard

https://www.liangzl.com/get-article-detail-114351.html

https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md

https://kubernetes.io/docs/reference/access-authn-authz/rbac/

https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/baremetal.md

https://metallb.universe.tf/installation/

## install docker

    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh --mirror Aliyun
    sudo usermod -aG docker your-user

add blow content to file `/etc/docker/daemon.json`

    {
    "registry-mirrors": ["https://registry.docker-cn.com"]
    }

reload daemon and restart docker service

    sudo systemctl daemon-reload 
    sudo service docker restart

### move docker storage location

stop docker service

    sudo systemctl stop docker.service

mount `/var/lib/docker` to other location

    sudo mkdir <new docker folder>
    sudo mount --rbind <new docker folder> /var/lib/docker

start docker service

    sudo systemctl start docker

add blow content to file `/etc/fstab` for mount after reboot

    /var/lib/docker <new docker folder> ext4 defaults 0 0

## install kubeadm

`kubectl` a tool for client

`kubeadm` a tool for server

### Debian / Ubuntu

    apt-get update && apt-get install -y apt-transport-https
    curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
    cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
    EOF
    apt-get update
    apt-get install -y kubelet kubeadm kubectl

### CentOS / RHEL / Fedora

    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF
    setenforce 0
    yum install -y kubelet kubeadm kubectl
    systemctl enable kubelet && systemctl start kubelet

## prepare kubeadm.conf

output config file `kubeadm.conf`

    kubeadm config print init-defaults > kubeadm.conf

modify `advertiseAddress` and `imageRepository`.

set `advertiseAddress` as server IP

set `imageRepository` as `registry.aliyuncs.com/google_containers`

set `podSubnet` under `networking` as `10.244.0.0/16`

example:

```
apiVersion: kubeadm.k8s.io/v1beta1
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.2.120
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: parcelx-3-20
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta1
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: ""
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.14.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

## turn off swap

    swapoff -a

disable swap after reboot, edit `/etc/fstab` and remove swap record

## install kube cluster

    kubeadm config images list --config kubeadm.conf
    kubeadm config images pull --config kubeadm.conf
    kubeadm init --config kubeadm.conf

## get login ca

To start using your cluster, you need to run the following as a regular user:

```
mkdir -p $HOME/.kube
sudo cp -rf /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## taint nodes

Execute blow command. If not, error `0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.` will appear when pods up

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## install cni

    wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

~~replace `quay.io/coreos` with `quay-mirror.qiniu.com/coreos` in file kube-flannel.yml~~

    kubectl apply -f kube-flannel.yml

## clean kube cluster

    kubeadm reset


## join cluster

    kubeadm join 192.168.2.120:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:1edda60ebcd1295f610619e3480c2ff9616ad50e1b1526d2d1927132ef5537a1

if token is overtime, creat a token and get token name

    kubeadm token create --print-join-command
    kubeadm token list

print token value

    openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

join

    kubeadm join 192.168.2.120:6443 --token <token name> \
        --discovery-token-ca-cert-hash sha256:<token value>


## find debug infos

    journalctl  -u kubelet -f

## install kubenetes-dashboard

    wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

modify file `kubernetes-dashboard.yaml`, replace `k8s.gcr.io` with `registry.aliyuncs.com/google_containers`

    kubectl apply -f kubernetes-dashboard.yaml

copy folder `~/.kube` to local, run blow command

    kubectl proxy

### access dashboard URL

create a yaml file `dashboard-admin.yaml`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

apply `dashboard-admin.yaml`

```
kubectl apply -f dashboard-admin.yaml
```

Bearer Token

    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

or 

    kubectl describe secret -n kube-system admin-user

login with the secret you get

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/


## install betallb

Service type `loadbalancer` can not be used by baremetal. For result this problem, we need project `betallb`

To install MetalLB, simply apply the manifest:

```
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
```

After creating the following ConfigMap, MetalLB takes ownership of one of the IP addresses in the pool and updates
the *loadBalancer* IP field of the `ingress-nginx` Service accordingly.


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.2.150-192.168.180
```

apply file `metallb-conf.yaml`

```
kubectl apply -f metallb-conf.yaml
```

## install ingress 

```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.24.1/deploy/mandatory.yaml
kubectl apply -f mandatory.yaml
```

change `spec.type` from `NodePort` to `LoadBalancer`

```
kubectl edit svc ingress-nginx  -n ingress-nginx
```

