Client客户端对象 

# 1. Client客户端对象

client-go支持4种客户端对象与Kubernetes API Server进行交互，如图所示：

RESTClient是最基础的客户端。RESTClient对HTTP Request进行了封装，实现了RESTful风格的API。ClientSet、DynamicClient及DiscoveryClient客户端都是基于RESTClient实现的。

- ClientSet在RESTClient的基础上封装了对Resource和Version的管理方案。每一个Resource可以理解为一个客户端，而ClientSet则是多个客户端的集合，每一个Resource和Version都以函数的方式暴露给开发者。ClientSet只能够处理Kubernetes内置资源，它是通过client-gen代码生成器自动生成的。
- DynamicClient与ClientSet最大的不同之处是，ClientSet仅能访问Kubernetes自带的资源(即Client集合内的资源)，不能直接访问CRD自定义资源。DynamicClient能够处理Kubernetes中的所有资源对象，包括Kubernetes内置资源与CRD自定义资源
- DiscoveryClient发现客户端，用于发现kube-apiserver所支持的资源组、资源版本、资源信息(即Group、Version、Resources)

以上4种客户端都可以通过kubeconfig配置信息连接到指定到Kubernetes API Server。

## 1.1 kubeconfig配置管理

kubeconfig可用于管理访问kube-apiserver的配置信息，也支持多集群的配置管理，可在不同环境下管理不同kube-apiserver集群配置。kubernetes的其他组件都是用kubeconfig配置信息来连接kube-apiserver组件。kubeconfig存储了集群、用户、命名空间和身份验证等信息，默认kubeconfig存放在$HOME/.kube/config路径下。kubeconfig的配置信息如下:

```
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://127.0.0.1:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: default
    user: username
  name: kubernetes
current-context: kubernetes
kind: Config
preferences: {}
users:
- name: username
  user:
    token: asdf
```

kubeconfig配置信息分为3部分，分别为：

- clusters：定义kubernetes集群信息，例如kube-apiserver的服务地址及集群的证书信息
- users：定义kubernetes集群用户身份验证的客户端凭据，例如client-certificate、client-key、token及username/password等。
- context：定义kuberntes集群用户信息和命名空间等，用于将请求发送到指定的集群

## 1.2 RESTClient客户端

RESTClient是最基础的客户端，其他的ClientSet、DynamicClient及DiscoveryClient都是基于RESTClient实现的。RESTClient对HTTP Request进行了封装，实现了RESTful风格的API。

## 1.3 ClientSet客户端

RESTClient是最基础的客户端，使用时需要指定Resource和Version等信息，编写代码时需要提前知道Resource所在的Group和对应的Version信息。ClientSet相比而言使用更加便捷，一般情况，对Kubernetes进行二次开发时通常使用ClientSet。 ClientSet在RESTClient的基础上封装了对Resource和Version的管理方法，每个Resource可以理解为一个客户端，而ClientSet则是多个客户端的集合，每个Resource和Version都以函数的方式暴露给开发者。

## 1.4 DynamicClient客户端

DynamicClient客户端是一种动态客户端，可以对任意的Kubernetes资源进行RESTful操作，包括CRD资源。 DynamicClient内部实现了Unstructured，用于处理非结构化数据结构（即无法提前预知的数据结构），这也是DynamicClient能够处理CRD资源的关键。

> DynamicClient不是类型安全的，因此在访问CRD自定义资源是要注意，例如，在操作不当时可能会导致程序崩溃。 DynamicClient的处理过程是将Resource(如PodList)转换成Unstructured结构类型，Kubernetes的所有Resource都可以转换为该结构类型。处理完后再将Unstructured转换成PodList。过程类似于Go语言的interface{}断言转换过程。另外，Unstructured结构类型是通过map[string]interface{}转换的。

## 1.5 DiscoveryClient客户端

DiscoveryClient是发现客户端，主要用于发现Kubenetes API Server所支持的资源组、资源版本、资源信息。 kubectl的api-versions和api-resources命令输出也是通过DiscoveryClient实现的。其同样是在RESTClient的基础上进行的封装。DiscoveryClient还可以将资源组、资源版本、资源信息等存储在本地，用于本地缓存，减轻对kubernetes api sever的访问压力，缓存信息默认存储在：~/.kube/cache和~/.kube/http-cache下。