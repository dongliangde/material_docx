# golang

## 开源工具

### statik (工具)

https://github.com/rakyll/statik

statik 允许您将静态文件目录嵌入到 Go 二进制文件中，以便稍后从 http.FileSystem 提供服务。

```
$ statik -m -include=*.jpg,*.txt,*.html,*.css,*.js
```

现在的很多程序都会提供一个 Dashboard 类似的页面用于查看程序状态并进行一些管理的功能，通常都不会很复杂，但是其中用到的图片和网页的一些静态资源，如果需要用户额外存放在一个目录，也不是很方便，如果能打包进程序发布的二进制文件中，用户下载以后可以直接使用，就方便很多。

最近在阅读 InfluxDB 的源码，发现里面提供了一个 admin 管理的页面，可以通过浏览器来执行一些命令以及查看程序运行的信息。但是我运行的时候只运行了一个 influxd 的二进制文件，并没有看到 css, html 等文件。

原来 InfluxDB 中使用了 statik 这个工具将静态资源都编译进了二进制文件中，这样用户只需要运行这个程序即可，而不需要管静态资源文件存放的位置。

### hugo(工具)

https://github.com/gohugoio/hugo

Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

搭建个人博客有很多开源的博客框架，我们要介绍的框架叫作Hugo。Hugo 是一个基于Go 语言的框架，可以快速方便的创建自己的博客。Hugo 支持Markdown 语法，我们可以将自己的文章写成Markdown 的格式，放在我们用Hugo 创建的博客系统中，从而展示给他人。

### awesome-go (综合库)

精选的 Go 框架、库和软件的精选列表。

### Kubernetes(容器管理工具)

Kubernetes，也称为 K8s，是一个开源系统，用于 跨多个主机管理[容器化应用程序](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/concepts/overview/what-is-kubernetes)。它提供了用于部署、维护和扩展应用程序的基本机制。

### Docker(容器)

Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、 [bare metal](https://link.zhihu.com/?target=http%3A//www.whatis.com.cn/word\_5275.htm)、OpenStack 集群和其他的基础应用平台。

### Gin(web框架)

Gin 是一个用 Go (Golang) 编写的 HTTP web 框架。 它是一个类似于 [martini](https://link.zhihu.com/?target=https%3A//github.com/go-martini/martini) 但拥有更好性能的 API 框架，由于 [httprouter](https://link.zhihu.com/?target=https%3A//github.com/julienschmidt/httprouter)，速度提高了近 40 倍。

### Beego(web框架)

beego 是一个快速开发 Go 应用的 HTTP 框架，他可以用来快速开发 API、Web 及后端服务等各种应用，是一个 RESTful 的框架，主要设计灵感来源于 tornado、sinatra 和 flask 这三个框架，但是结合了 Go 本身的一些特性（interface、struct 嵌入等）而设计的一个框架。

### Go Micro(微服务框架)

Go Micro是一个流行的微服务架构，是一个插件化的基础框架，基于此可以构建微服务，Micro的设计哲学是可插拔的插件化架构。Go Micro 简单轻巧、易于上手、功能强大、扩展方便，是基于 Go 语言进行微服务架构时非常值得推荐的一个框架。

### Echo(web框架)

Echo web框架是go语言开发的一种高性能，可扩展，轻量级的web框架。Echo框架真的非常简单，几行代码就可以启动一个高性能的http服务端。

### Iris(web框架)

Iris 是一个快速、简单但功能齐全且非常高效的 Go 网络框架。

### Revel(综合框架)

Go语言编写的高效、全栈Web框架。

### Jupiter(微服务框架)

Jupiter是斗鱼开源的面向服务治理的Golang微服务框架。

### nsq(消息队列)

NSQ是实时的分布式消息平台。它的设计目标是为在多台计算机上运行的松散服务提供一个现代化的基础设施骨架。bitly开源的消息队列系统，性能非常高，目前他们每天处理数十亿条的消息。

### gin-vue-admin(gin、gorm脚手架)

Gin-vue-admin是一个基于[vue](https://link.zhihu.com/?target=https%3A//vuejs.org)和[gin](https://link.zhihu.com/?target=https%3A//gin-gonic.com)开发的全栈前后端分离的后台管理系统，集成jwt鉴权，动态路由，动态菜单，casbin鉴权，表单生成器，代码生成器等功能，提供多种示例文件，让您把更多时间专注在业务开发上。

### go-admin(gin、gorm脚手架)

基于Gin + Vue + Element UI的前后端分离权限管理系统,系统初始化极度简单，只需要配置文件中，修改数据库连接，系统支持多指令操作，迁移指令可以让初始化数据库信息变得更简单，服务指令可以很简单的启动api服务。

### go-zero(web 、rpc 框架)

go-zero 是一个集成了各种工程实践的 web 和 rpc 框架。通过弹性设计保障了大并发服务端的稳定性，经受了充分的实战检验。

### rpcx(rpc 框架)

Go 中最好的微服务框架，如 alibaba Dubbo，但功能更多，易于扩展。

### GoFrame(Go基础开发框架)

`GoFrame`是一款模块化、高性能、企业级的Go基础开发框架。

如果您初识`Go`语言，您可以将`GoFrame`类似于`PHP`中的`Laravel`, `Java`中的`SpringBoot`或者`Python`中的`Django`。

### Dapr(分布式框架)

Dapr 是一个可移植的、事件驱动的运行时，用于跨云和边缘构建分布式应用程序。

### Hugo(个人博客生成器)

Hugo 是一个用[Go](https://link.zhihu.com/?target=https%3A//golang.org)编写的静态 HTML 和 CSS 网站生成器。它针对速度、易用性和可配置性进行了优化。Hugo 获取包含内容和模板的目录，并将它们呈现为一个完整的 HTML 网站。

### Martini(web框架)

Martini 是一个非常新的 Go 语言的 Web 框架，使用 Go 的 net/http 接口开发，类似 Sinatra 或者 Flask 之类的框架，你可使用自己的 DB 层、会话管理和模板。

### YoyoGo(微服务框架)

YoyoGo 简单、轻量、快速、基于依赖注入的微服务框架。

### gitea(代码版本管理工具)

这个项目的目标是以最简单、最快、最轻松的方式建立一个自托管Git服务。使用Go，这可以通过Go支持的所有平台（包括x86、amd64、ARM和PowerPC体系结构上的Linux、macOS和Windows）的独立二进制分发来实现。

### tidb(HTAP数据库)

TiDB是一个与MySQL协议兼容的开源分布式HTAP数据库。

### im\_service(即时通讯框架)

golang即时通讯服务器。

### go-gin-api(脚手架框架)

基于 Gin 进行模块化设计的 API 框架，封装了常用功能，使用简单，致力于进行快速的业务研发。比如，支持 cors 跨域、jwt 签名验证、zap 日志收集、panic 异常捕获、trace 链路追踪、prometheus 监控指标、swagger 文档生成、viper 配置文件解析、gorm 数据库组件、gormgen 代码生成工具、graphql 查询语言、errno 统一定义错误码、gRPC 的使用、cron 定时任务 等等。

# 一致性哈希算法

## 分布式缓存背景

随着业务的扩展，流量的剧增，单体项目逐渐划分为分布式系统。对于经常使用的数据，我们可以使用Redis作为缓存机制，减少数据层的压力。因此，重构后的系统架构如下图所示：

![](<../golang/img/1545912147957.png>)

优化最简单的策略就是，把常用的数据保存到Redis中，为了实现高可用使用了3台Redis（没有设置集群，集群至少要6台）。每次Redis请求会随机发送到其中一台，但是这种策略会引发如下两个问题：

- 同一份数据可能在多个Redis数据库，造成数据冗余
- 某一份数据在其中一台Redis数据库已存在，但是再次访问Redis数据库，并没有命中数据已存在的库。无法保证对相同的key的所有访问都发送到相同的Redis中

要解决上述的问题，我们需要稍稍改变一些key存入Redis的规则：**使用hash算法** 例如，有三台Redis，对于每次的访问都可以通过计算hash来求得hash值。 如公式 h=hash(key)%3，我们把Redis编号设置成0,1,2来保存对应hash计算出来的值，h的值等于Redis对应的编号。 但是hash算法也会面临容错性和扩展性的问题。容错性是指当系统中的某个服务出现问题时，不能影响其他系统。扩展性是指当加入新的服务器后，整个系统能正确高效运行。

现假设有一台Redis服务器宕机了，那么为了填补空缺，要将宕机的服务器从编号列表中移除，后面的服务器按顺序前移一位并将其编号值减一，此时每个key就要按h = Hash(key) % 2重新计算。

同样，如果新增一台服务器，规则也同样需要重新计算，h = Hash(key) % 4。因此，系统中如果有服务器更变，会直接影响到Hash值，大量的key会重定向到其他服务器中，造成缓存命中率降低，而这种情况在分布式系统中是十分糟糕的。

一个设计良好的分布式哈希方案应该具有良好的单调性，即服务节点的变更不会造成大量的哈希重定位。一致性哈希算法由此而生~

## 一致性哈希算法

> 一致哈希 是一种特殊的哈希算法。在使用一致哈希算法后，哈希表槽位数（大小）的改变平均只需要对 K/n 个关键字重新映射，其中K是关键字的数量， n是槽位数量。然而在传统的哈希表中，添加或删除一个槽位的几乎需要对所有关键字进行重新映射。

 （哈希值是32位无符号整形），整个哈希空间环如下： 

![](<../golang/img/1545916990776.png>)

整个空间按顺时针方向组织，0和2^32-1在零点中方向重合。

 接下来，把服务器按照IP或主机名作为关键字进行哈希，这样就能确定其在哈希环的位置。

 ![](<../golang/img/1545917087577.png>)

然后，我们就可以使用哈希函数H计算值为key的数据在哈希环的具体位置h，根据h确定在环中的具体位置，从此位置沿顺时针滚动，遇到的第一台服务器就是其应该定位到的服务器。

 例如我们有A、B、C、D四个数据对象，经过哈希计算后，在环空间上的位置如下：

 