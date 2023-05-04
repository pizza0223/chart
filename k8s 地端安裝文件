## 環境配置
# OS: Ubuntu 18, 最小安裝。
```
k8s-master 192.168.66.164  4core CPU   8 GB RAM  100GB HDD
k8s-minion1  192.168.66.166  4core CPU   8 GB RAM  100GB HDD
k8s-minion2  192.168.66.167  4core CPU   8 GB RAM  100GB HDD
```

### 注意：以下設定到重啟 containerd 為止三台都要執行。(開始) ###
### 若是不想重複的動作執行三次，可以先開一台做過之後再 clone 出其他節點並修改 hostname ###

# 關掉 swap
```
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

# 安裝前置套件
```
apt-get update
apt-get install ca-certificates curl gnupg vim -y
```

## 安裝 docker 

# 新增 Docker 官方 GPG key
```
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg
```

# 設定 repository
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

# 更新 安裝 docker
```
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## k8s 安裝前作業
# 新增 kernal 模組
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```
modprobe overlay
modprobe br_netfilter
```

# 新增 sysctl 系統參數
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

# 使參數生效
```
sysctl --system
```

## 安裝 k8s
# 更新套件源，安裝 (最後一行是因為部分平台的 node 會自己自動進行更新，為了避免未預期的情況下被升級版本，因此新增此設定)
```
apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl --insecure -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

## 啟動 systemd

# 設定
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

# 重啟
```
systemctl restart containerd
```

### 注意：以下設定到重啟 containerd 為止三台都要執行。(結束) ###

## 初始化 master 節點 (注意：只要在 master 節點上執行即可)
```
kubeadm init --apiserver-advertise-address 192.168.66.164 --pod-network-cidr 10.244.0.0/16
```

# 最後若有看到這一段文字基本上就算是完成 master 節點設定
```
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

kubeadm join 192.168.66.164:6443 --token aqatip.5bumjle5pnj1b3wn \
        --discovery-token-ca-cert-hash sha256:1281e171a4d854a9d63cf80c22eca13e5fc1c9e272eb537e0672efe6a7e83323
```

## 設定 master 節點
# 請照以上的說明 "to start using cluster...." 下面的指令執行
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

# 輸入以下命令可以讓 shell 自動補完
```
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
```

## 安裝 Flannel 網路 (注意：只要在 master 節點上執行即可)
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## 將 k8s-minion1、k8s-minion2 加入此 master 節點
# 將初始化成功時所拿到的指令拷貝出來到其他兩台執行之 (用 root 執行)
```
kubeadm join 192.168.66.164:6443 --token aqatip.5bumjle5pnj1b3wn \
        --discovery-token-ca-cert-hash sha256:1281e171a4d854a9d63cf80c22eca13e5fc1c9e272eb537e0672efe6a7e83323
```
