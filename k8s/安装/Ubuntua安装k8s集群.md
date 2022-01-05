# Ubuntu离线安装 k8s 1.23.1

- 一台兼容的 Linux 主机。Kubernetes 项目为基于 Debian 和 Red Hat 的 Linux 发行版以及一些不提供包管理器的发行版提供通用的指令
- 每台机器 2 GB 或更多的 RAM （如果少于这个数字将会影响你应用的运行内存)
- 2 CPU 核或更多
- 集群中的所有机器的网络彼此均能相互连接(公网和内网都可以)
- 节点之中不可以有重复的主机名、MAC 地址或 product_uuid。请参见这里了解更多详细信息。
- 禁用交换分区。为了保证 kubelet 正常工作，你 必须 禁用交换分区。

## 1，关闭防火墙

```
systemctl disable ufw 
systemctl stop ufw
```

## 2，安装docker

### 卸载已有docker

旧版本的docker是docker, docker.io, 或者 docker-engine，如果之前安装了这些，先卸载

```
 sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 使用仓库安装Docker

 更新apt 包索引和安装包，使得apt 可以通过HTTPS 使用仓库。	

```
sudo apt-get update
```

```
sudo apt-get install ca-certificates curl gnupg lsb-release
```

### 添加Docker的官方 GPG key

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### 如果没有curl，通过以下命令安装

```
apt-get install curl
```

### 设置稳定仓库

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 安装指定版本的Docker Engine 和containerd

```
apt-cache madison docker-ce
```

### 下载指定版本：替换为你想要下载的版本

```
sudo apt-get update
sudo apt-get install docker-ce=版本号 docker-ce-cli=版本号
```

### 验证是否安装成功

```
sudo docker run hello-world
```

### 通过apt离线下载deb包以及其依赖包

apt可以通过添加--download-only选项，简写为-d，表示只下载包不安装包，目前默认下载路径目录是/var/cache/apt/archives

示例：

```
apt reinstall mariadb-server -d 
```

这里建议是reinstall，因为如果包已经安装过的话就无法下载，reinstall的话无论是否安装都会下载

### 离线安装 docker

```
dpkg -i *.deb
```

### 为Docker配置用户组

 修改docker的cgroup driver为systemd 在/etc/docker目录下新增**daemon.json**,文件内容为: 

```
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

```
#添加当前用户到docker用户组
sudo usermod -aG docker {用户名}
#配置完成后需要重启docker
systemctl restart docker
```

### 启动

```
systemctl enable docker.service
```

```
systemctl restart docker
```

## 3，离线安装 kubeadm、kubelet 和 kubectl

- `kubeadm`：用来初始化集群的指令。
- `kubelet`：在集群中的每个节点上用来启动 Pod 和容器等。
- `kubectl`：用来与集群通信的命令行工具。

kubeadm **不能** 帮你安装或者管理 `kubelet` 或 `kubectl`，所以你需要 确保它们与通过 kubeadm 安装的控制平面的版本相匹配。 如果不这样做，则存在发生版本偏差的风险，可能会导致一些预料之外的错误和问题。 然而，控制平面与 kubelet 间的相差一个次要版本不一致是支持的，但 kubelet 的版本不可以超过 API 服务器的版本。 例如，1.7.0 版本的 kubelet 可以完全兼容 1.8.0 版本的 API 服务器，反之则不可以。

```
apt-get update && apt-get install -y apt-transport-https
```

### 切换root用户

```
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
```

```
cat > /etc/apt/sources.list.d/kubernetes.list << ERIC

 deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main 

ERIC
```

```
apt-get update
```

### 查询安装kubeamd安装版本

```
apt-cache madison kubeadm
```

### 将包下载到本地

```
apt-get install -y --download-only -o dir::cache::archives=/home/k8s kubelet=1.23.1 kubeadm=1.23.1 kubectl=1.23.1
```

### 离线安装 k8s

```
dpkg -i /home/k8s/*.deb
```

### 查看镜像依赖版本

```
kubeadm config images list --kubernetes-version=v1.23.1
```

```
k8s.gcr.io/kube-apiserver:v1.23.1
k8s.gcr.io/kube-controller-manager:v1.23.1
k8s.gcr.io/kube-scheduler:v1.23.1
k8s.gcr.io/kube-proxy:v1.23.1
k8s.gcr.io/pause:3.6
k8s.gcr.io/etcd:3.5.1-0
k8s.gcr.io/coredns/coredns:v1.8.6
```

### 下载镜像

目前k8s.gcr.io被国内墙了不能访问，除了有境外服务器直接下载外只能通过阿里镜像库下载

编写脚本

```
cat > download_image.sh << ERIC
#!/bin/bash
images=(
    kube-apiserver:v1.23.1
    kube-controller-manager:v1.23.1
    kube-scheduler:v1.23.1
    kube-proxy:v1.23.1
    pause:3.6
    etcd:3.5.10-0
    coredns:1.7.0
)

proxy=registry.cn-hangzhou.aliyuncs.com/google_containers/

echo '+----------------------------------------------------------------+'
for img in \${images[@]};
do
    docker pull \$proxy\$img
    docker tag  \$proxy\$img k8s.gcr.io/\$img
    docker rmi  \$proxy\$img
    echo '+----------------------------------------------------------------+'
    echo ''
done

ERIC
```

coredns镜像需要重新tag一下

```
docker tag a4ca41631cc7 k8s.gcr.io/coredns/coredns:v1.8.6
```

### 授予执行权限

```
chmod -R 755 download_image.sh
```

### 执行脚本后获取镜像信息

### 关闭Swap

```
swapoff -a && sed -ri 's/.*swap.*/#&/' /etc/fstab
```

### 初始化配置文件

```
kubeadm init
```

![1641345746571](C:\Users\cq\AppData\Local\Temp\1641345746571.png)

成功之后会出现 以上信息，随后执行上述提示的命令

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

通过命令查看pod状态

```
kubectl get pod -A
```

