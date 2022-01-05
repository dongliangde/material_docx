# 仿真热迁移系统使用手册

## 1.镜像构建

根据构建镜像目录文件生成镜像

- criu                           `CRIU（Checkpoint/Restore In Userspace）运行在linux操作系统上的一个软件工具`

- criu-http-server      `测试httpp-server程序`

- criu-server               `仿真热迁移系统`

- Dockerfile                `构建docker images文件`

- kuboard_criu.yaml `运行在Kubernetes仿真热迁移系统yaml文件  `

   `构建镜像时必须使用criu和criu-server，criu-http-server测试热迁移系统的http-server程序`

  `Dockerfile构建镜像的文件，使用的时候可以根据自己的业务需求来更改Dockerfile文件`

  `使用kuboard_criu.yaml需要注意里面的配置镜像仓库地址，启动对外端口等`

  

## 2.仿真热迁移系统运行

确保构建完成docker镜像之后，根据Kubernetes环境来修改kuboard_criu.yaml文件

通过命令将kuboard_criu.yaml中的配置应用到pod 

```
kubectl apply -f kuboard_criu.yaml
```

运行之后会出现以下日志代表创建pod成功

```
cq@ubuntu:~/sss$ sudo kubectl apply -f kuboard_criu.yaml 
deployment.apps/svc-criu created
service/svc-criu created
secret/cq created
```

通过命令查看是否运行成功

```
cq@ubuntu:~/sss$ sudo kubectl get pods --all-namespaces
NAMESPACE     NAME                          READY   STATUS              RESTARTS   AGE
default       svc-criu-8d6969765-b86x6        0/1     ContainerCreating   0          2s

```

通过kubectl get pods --all-namespaces命令可以看到容器创建中

```
default       svc-criu-8d6969765-b86x6        0/1     ContainerCreating   0          2s

```

稍等一会再次通过命令查看是否启动成功，看到 READY 1/1代表启动成功

```
cq@ubuntu:~/sss$ sudo kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
default       svc-criu-8d6969765-b86x6                  1/1     Running     0          27s

```

通过以下IP和Port来访问swagger api文档

```
http://IP:Port/swagger/index.html
```

## 3.接口介绍

### 1.启动程序

 简要描述

- 根据命令参数来启动docker内的程序并返回程序进程ID接口

请求URL

- http://IP:Port/startCmd

请求方式

- POST

表头

- Content-Type: application/json

参数

| 参数名    | 必选 | 类型   | 说明                                |
| --------- | ---- | ------ | ----------------------------------- |
| path      | 是   | string | 运行程序命令 例`./criu-http-server` |
| parameter | 否   | string | 运行所需参数                        |

输入示例

```
{
  "path": "./criu-http-server",
  "parameter":""
}
```

返回示例

```
{
    "code": 200,
    "data": 14,                `进程ID`
    "msg": "Success"
}
```

### 2.冻结已启动程序

 简要描述

- 根据进程ID和保存冻结文件地址dir冻结已启动程序接口

请求URL

- http://IP:Port/dump 

请求方式

- POST

表头

- Content-Type: application/json

参数

| 参数名          | 必选 | 类型   | 说明                           |
| --------------- | ---- | ------ | ------------------------------ |
| pid             | 是   | string | 冻结已启动程序的进程ID         |
| dir             | 是   | string | 冻结文件保存地址               |
| shellJob        | 是   | bool   | 是否是一个命令行程序           |
| tcpEstablished  | 是   | bool   | 是否已建立TCP连接              |
| tcpSkipInFlight | 是   | bool   | 是否跳过正在运行的TCP连接      |
| leaveRunning    | 是   | bool   | 在不停止程序时冻结当前程序进程 |
| tcpClose        | 是   | bool   | 关闭已建立的TCP                |

输入示例

```
{
  "pid": 14,
  "dir": "/app/111",
  "shellJob":true,
  "leaveRunning": false,
  "tcpEstablished": true,
  "tcpSkipInFlight":true,
  "tcpClose":true
}
```

返回示例

```
{
    "code": 200,
    "data": "",
    "msg": "Success"
}
```

### 3.恢复冻结程序

 简要描述

- 根据冻结程序的路径恢复程序启动接口

请求URL

- http://IP:Port/restore 

请求方式

- POST

表头

- Content-Type: application/json

参数

| 参数名          | 必选 | 类型   | 说明                      |
| --------------- | ---- | ------ | ------------------------- |
| dir             | 是   | string | 冻结文件保存地址          |
| shellJob        | 是   | bool   | 是否是一个命令行程序      |
| tcpEstablished  | 是   | bool   | 是否已建立TCP连接         |
| tcpSkipInFlight | 是   | bool   | 是否跳过正在运行的TCP连接 |
| tcpClose        | 是   | bool   | 关闭已建立的TCP           |
| rstSibling      | 是   | bool   | 将根任务还原为同级        |

输入示例

```
{
  "dir": "/app/111",
  "shellJob":true,
  "tcpEstablished": true,
  "tcpSkipInFlight":true,
  "tcpClose":true,
  "rstSibling":false
}
```

返回示例

```
{
    "code": 200,
    "data": "",
    "msg": "Success"
}
```

