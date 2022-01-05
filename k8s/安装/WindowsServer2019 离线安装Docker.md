# 在WindowsServer2019 添加k8s节点

 参考资料https://www.jianshu.com/p/b7cf999db583

###1，安装windows_update.msu

开启Hyper-V

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

安装完成之后需要重启

### 2, 开启RRAS 功能，用于 Pod 跨主机通信

![img](https://upload-images.jianshu.io/upload_images/15327309-18662a7abc19a38b.png?imageMogr2/auto-orient/strip|imageView2/2/w/708/format/webp)



## 3,安装docker

1. 下载ee版本的Windows Server,地址如下

   https://download.docker.com/components/engine/windows-server/19.03/docker-19.03.0.zip
   docker-19.03.0.zip

   安装docker 命令

   ```
   Write-Host "install docker"
   
   Expand-Archive .\docker-19.03.0.zip -DestinationPath $Env:ProgramFiles -Force
   
   $null = Install-WindowsFeature containers
   
   env:path += ";env:ProgramFiles\docker"
   
   dockerd --register-service
   
   Start-Service docker
   
   Write-Host "docker installed "
   
   ```

### 4,执行PrepareNode脚本安装kubectl

安装之前把k文件夹和PrepareNode.ps1脚本拷贝到Windows

运行脚本：  

```
PrepareNode.ps1
```

### 5,安装calico

参考资料https://docs.projectcalico.org/getting-started/windows-calico/kubernetes/standard#install-calico-and-kubernetes-on-windows-nodes

首先在C盘下新建k文件夹解压calico.zip压缩文件

获取主节点/home/cq/.kube/目录下config文件，并拷贝到Windows节点C:\k目录下

需要在主节点安装calico组件的时候，需要对calico.yaml文件进行修改(可能出现某个节点无法绑定ipv4地址)

	# calico节点无法绑定ipv4新加配置
	- name: IP_AUTODETECTION_METHOD
	value: "interface=eth0"
	# Auto-detect the BGP IP address.
	- name: IP
	value: "autodetect"
	# Enable IPIP
	# 添加windows节点,修改为Never,之前为Always
	- name: CALICO_IPV4POOL_IPIP
	value: "Never"
	# 添加windows节点新加配置
	- name: CALICO_AUTODETECTION_METHOD
	value: "interface=eth0"
#### secret

https://github.com/projectcalico/calico/issues/4037

在通过编辑clusterrolebinding来修复它系统：节点和添加系统：节点组作为一个主题。这需要在控制实例中完成，在该实例中，您对kubectl拥有完全的管理权限：

```
kubectl.exe edit clusterrolebinding system:node
```

**Add the following:**

```
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:nodes
```

执行install-calico-windows.ps1脚本

通过一下脚本关联主机点

```
~ # kubeadm token create --print-join-command                                                                                  
kubeadm join 172.29.9.239:6443 --token iuqm3l.vrmrlqcqtgqbwdtk     --discovery-token-ca-cert-hash sha256:02739e517b832cf38109779fd178b35fb71efe6a3b13cfd9c44580c273211fec
```

在主节点修改calicoctl

./calicoctl ipam configure --strictaffinity=true

### 6，查看

### 查看node

linux控制界面查看：

```
~ # kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
oran-master1   Ready    master   2d20h   v1.18.2
oran-node1     Ready    <none>   160m    v1.18.2
```



 

 