# 作为k8s容器运行时，containerd跟docker的对比

# 调用关系的对比

![xfn6e4nnl3](G:\md资料文件\xfn6e4nnl3.png)

![b1tez6us37](G:\md资料文件\b1tez6us37.png)

# 容器日志及相关参数

 

| 对比项                 | docker                                                       | containerd                                                   |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存储路径               | docker作为k8s容器运行时的情况下，容器日志的落盘由docker来完成， 保存在类似`/var/lib/docker/containers/$CONTAINERID`目录下。kubelet会在`/var/log/pods`和`/var/log/containers`下面建立软链接，指向`/var/lib/docker/containers/$CONTAINERID`目录下的容器日志文件 | containerd作为k8s容器运行时的情况下， 容器日志的落盘由kubelet来完成，保存到`/var/log/pods/$CONTAINER_NAME`目录下，同时在`/var/log/containers`目录下创建软链接，指向日志文件 |
| 配置参数               | 在docker配置文件中指定：     "log-driver": "json-file",      "log-opts": {"max-size": "100m","max-file": "5"} | 方法一：在kubelet参数中指定：   --container-log-max-files=5 --container-log-max-size="100Mi"  方法二：在KubeletConfiguration中指定：     "containerLogMaxSize": "100Mi",     "containerLogMaxFiles": 5, |
| 把容器日志保存到数据盘 | 把数据盘挂载到"data-root"(缺省是/var/lib/docker)即可         | 创建一个软链接/var/log/pods指向数据盘挂载点下的某个目录  在TKE中选择"将容器和镜像存储在数据盘"，会自动创建软链接/var/log/pods |

# stream server

kubectl exec/logs等命令需要在apiserver跟容器运行时之间建立流转发通道。

docker API本身提供stream服务，kubelet内部的docker-shim会通过docker API做流转发。

containerd的stream服务需要单独配置：

```
[plugins.cri]
  stream_server_address = "127.0.0.1"
  stream_server_port = "0"
  enable_tls_streaming = false
```

在k8s 1.11之前，kubelet并不会做stream proxy, 只会做redirect。也就是把containerd暴露的stream server地址告诉apiserver, 让apiserver直接来访问containerd的stream server。这种情况下，需要给stream server使能tle认证来做安全防护。

从k8s1.11引入了kubelet stream proxy (<https://github.com/kubernetes/kubernetes/pull/64006>)， 从而使得containerd stream server只需要监听本地地址即可。

# CNI网络

| 对比项        | docker                                      | containerd                                                   |
| ------------- | ------------------------------------------- | ------------------------------------------------------------ |
| 谁负责调用CNI | kubelet内部的docker-shim                    | containerd内置的cri-plugin(containerd 1.1以后)               |
| 如何配置CNI   | kubelet参数 --cni-bin-dir 和 --cni-conf-dir | containerd配置文件(toml)：  plugins.cri.cni     bin_dir = "/opt/cni/bin"     conf_dir = "/etc/cni/net.d" |

# 常见命令

containerd不支持docker API和docker CLI， 但是可以通过cri-tool实现类似的功能。

| 镜像相关功能     | docker         | containerd      |
| ---------------- | -------------- | --------------- |
| 显示本地镜像列表 | docker images  | crictl images   |
| 下载镜像         | docker pull    | crictl pull     |
| 上传镜像         | docke push     | 无              |
| 删除本地镜像     | docker rmi     | crictl rmi      |
| 查看镜像详情     | docker inspect | crictl inspecti |

------

| 容器相关功能 | docker         | containerd     |
| ------------ | -------------- | -------------- |
| 显示容器列表 | docker ps      | crictl ps      |
| 创建容器     | docker create  | crtctl create  |
| 启动容器     | docker start   | crtctl start   |
| 停止容器     | docker stop    | crictl stop    |
| 删除容器     | docker rm      | crictl rm      |
| 查看容器详情 | docker inspect | crictl inspect |
| attach       | docker attach  | crictl attach  |
| exec         | docker exec    | crictl exec    |
| logs         | docker logs    | crictl logs    |
| stats        | docker stats   | crictl stats   |

------

| POD相关功能 | docker | containerd      |
| ----------- | ------ | --------------- |
| 显示POD列表 | 无     | crictl pods     |
| 查看POD详情 | 无     | crictl inspectp |
| 运行POD     | 无     | crictl runp     |
| 停止POD     | 无     | crictl stopp    |