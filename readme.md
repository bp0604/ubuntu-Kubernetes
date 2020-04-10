# Ubuntu 上安装 Kubernetes

## 1 系统准备

### 1.1 登录免费 Ubuntu 资源

登录以下网址 https://katacoda.com/courses/ubuntu/playground

认证后，即可操作。

### 1.2 关防火墙
```
sudo ufw disable
```

### 1.3 关闭系统 swap
```
sudo swapoff -a
```

## 2 Kubernetes 环境

### 2.1 添加 K8s 安装密钥
```
sudo apt update &&  sudo apt install -y apt-transport-https curl
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg |  sudo apt-key add -
```

### 2.2 配置 K8S 源
```
sudo touch /etc/apt/sources.list.d/kubernetes.list
sudo echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"  >> /etc/apt/sources.list.d/kubernetes.list
```

### 2.3 安装 kubeadm 及 kubelet 等工具
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

保持版本取消自动更新，这里也可以省略
```
sudo apt-mark hold kubelet kubeadm kubectl
```

## 3 kubeadm 初始化
### 3.1 初始化操作
```
sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.1 --pod-network-cidr=10.240.0.0/16
````

### 3.2 配置非 root 的操作
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g)  $HOME/.kube/config
```

### 3.3 coredns 问题解决
如果 coredns 服务可能一直处于 pending 状态，那么我们需要做如下操作
```
kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
```

## 4 查看 pod 状态
```
kubectl get pods --all-namespaces
```

## 5 运行
### 5.1 Master 隔离解除
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### 5.2 加入工作节点
这项工作是集群的，单机可以忽略
执行上面 init 出现的 join 结果
```
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>

# kubeadm token list (可以获取<token>的值)
# kubeadm token create (可以创建<token>新的值)

# --discovery-token-ca-cert-hash (以下命令可以获取 --discovery-token-ca-cert-hash的值)
# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```












