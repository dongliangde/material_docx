# gRPC

# 安装

## 1.1.1. gRPC简介

- gRPC由google开发，是一款语言中立、平台中立、开源的远程过程调用系统
- gRPC客户端和服务端可以在多种环境中运行和交互，例如用java写一个服务端，可以用go语言写客户端调用

### 1.1.2. gRPC与Protobuf介绍

- 微服务架构中，由于每个服务对应的代码库是独立运行的，无法直接调用，彼此间的通信就是个大问题
- gRPC可以实现微服务，将大的项目拆分为多个小且独立的业务模块，也就是服务，各服务间使用高效的protobuf协议进行RPC调用，gRPC默认使用protocol buffers，这是google开源的一套成熟的结构数据序列化机制（当然也可以使用其他数据格式如JSON）
- 可以用proto files创建gRPC服务，用message类型来定义方法参数和返回类型



### 1.1.3. 安装gRPC和Protobuf

- go get github.com/golang/protobuf/proto
- go get google.golang.org/grpc（无法使用，用如下命令代替） 
- git clone <https://github.com/grpc/grpc-go.git> $GOPATH/src/google.golang.org/grpc
- git clone <https://github.com/golang/net.git> $GOPATH/src/golang.org/x/net
- git clone <https://github.com/golang/text.git> $GOPATH/src/golang.org/x/text
- go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
- git clone <https://github.com/google/go-genproto.git> $GOPATH/src/google.golang.org/genproto
- cd $GOPATH/src/
- go install google.golang.org/grpc

- go get github.com/golang/protobuf/protoc-gen-go

- 上面安装好后，会在GOPATH/bin下生成protoc-gen-go.exe

- 但还需要一个protoc.exe，windows平台编译受限，很难自己手动编译，直接去网站下载一个，地址：<https://github.com/protocolbuffers/protobuf/releases/tag/v3.9.0> ，同样放在GOPATH/bin下

  注意：这里面好多都是需要vpn才能下载好的！分享一个下载好的<https://pan.baidu.com/s/1T8eJkHib2uPL3gNMCRdZmQ> 提取码 s42z 

  # 1. gRPC简介

   [gRPC](https://grpc.io/) 是一个高性能、开源、通用的RPC框架，由Google推出，基于[HTTP2](https://http2.github.io/)协议标准设计开发，默认采用[Protocol Buffers](https://developers.google.com/protocol-buffers/)数据序列化协议，支持多种开发语言。gRPC提供了一种简单的方法来精确的定义服务，并且为客户端和服务端自动生成可靠的功能库。

  在gRPC客户端可以直接调用不同服务器上的远程程序，使用姿势看起来就像调用本地程序一样，很容易去构建分布式应用和服务。和很多RPC系统一样，服务端负责实现定义好的接口并处理客户端的请求，客户端根据接口描述直接调用需要的服务。客户端和服务端可以分别使用gRPC支持的不同语言实现。

  ![img](http://www.topgoer.com/static/wei/grpc/grpc_concept_diagram_00.png)

  ## 1.1. 主要特性

  - 强大的IDL

    gRPC使用ProtoBuf来定义服务，ProtoBuf是由Google开发的一种数据序列化协议（类似于XML、JSON、hessian）。ProtoBuf能够将数据进行序列化，并广泛应用在数据存储、通信协议等方面。

  - 多语言支持

    gRPC支持多种语言，并能够基于语言自动生成客户端和服务端功能库。目前已提供了C版本grpc、Java版本grpc-java 和 Go版本grpc-go，其它语言的版本正在积极开发中，其中，grpc支持C、C++、Node.js、Python、Ruby、Objective-C、PHP和C#等语言，grpc-java已经支持Android开发。

  - HTTP2

    gRPC基于HTTP2标准设计，所以相对于其他RPC框架，gRPC带来了更多强大功能，如双向流、头部压缩、多复用请求等。这些功能给移动设备带来重大益处，如节省带宽、降低TCP链接次数、节省CPU使用和延长电池寿命等。同时，gRPC还能够提高了云端服务和Web应用的性能。gRPC既能够在客户端应用，也能够在服务器端应用，从而以透明的方式实现客户端和服务器端的通信和简化通信系统的构建。

    更多介绍请查看[官方网站](https://grpc.io/)

    # 1. Protobuf⇢Go转换

    这里使用一个测试文件对照说明常用结构的protobuf到golang的转换。只说明关键部分代码，详细内容请查看完整文件。示例文件在`proto/test`目录下。

    ### 1.1.1. Package

    在proto文件中使用`package`关键字声明包名，默认转换成go中的包名与此一致，如果需要指定不一样的包名，可以使用`go_package`选项：

    ```
    package test;
    option go_package="test";
    ```

    ### 1.1.2. Message

    proto中的`message`对应go中的`struct`，全部使用驼峰命名规则。嵌套定义的`message`，`enum`转换为go之后，名称变为`Parent_Child`结构。

    示例proto：

    ```
    // Test 测试
    message Test {
        int32 age = 1;
        int64 count = 2;
        double money = 3;
        float score = 4;
        string name = 5;
        bool fat = 6;
        bytes char = 7;
        // Status 枚举状态
        enum Status {
            OK = 0;
            FAIL = 1;
        }
        Status status = 8;
        // Child 子结构
        message Child {
            string sex = 1;
        }
        Child child = 9;
        map<string, string> dict = 10;
    }
    ```

    转换结果：

    ```
    // Status 枚举状态
    type Test_Status int32
    
    const (
        Test_OK   Test_Status = 0
        Test_FAIL Test_Status = 1
    )
    
    // Test 测试
    type Test struct {
        Age    int32       `protobuf:"varint,1,opt,name=age" json:"age,omitempty"`
        Count  int64       `protobuf:"varint,2,opt,name=count" json:"count,omitempty"`
        Money  float64     `protobuf:"fixed64,3,opt,name=money" json:"money,omitempty"`
        Score  float32     `protobuf:"fixed32,4,opt,name=score" json:"score,omitempty"`
        Name   string      `protobuf:"bytes,5,opt,name=name" json:"name,omitempty"`
        Fat    bool        `protobuf:"varint,6,opt,name=fat" json:"fat,omitempty"`
        Char   []byte      `protobuf:"bytes,7,opt,name=char,proto3" json:"char,omitempty"`
        Status Test_Status `protobuf:"varint,8,opt,name=status,enum=test.Test_Status" json:"status,omitempty"`
        Child  *Test_Child `protobuf:"bytes,9,opt,name=child" json:"child,omitempty"`
        Dict   map[string]string `protobuf:"bytes,10,rep,name=dict" json:"dict,omitempty" protobuf_key:"bytes,1,opt,name=key" protobuf_val:"bytes,2,opt,name=value"`
    }
    
    // Child 子结构
    type Test_Child struct {
        Sex string `protobuf:"bytes,1,opt,name=sex" json:"sex,omitempty"`
    }
    ```

    除了会生成对应的结构外，还会有些工具方法，如字段的getter:

    ```
    func (m *Test) GetAge() int32 {
        if m != nil {
            return m.Age
        }
        return 0
    }
    ```

    枚举类型会生成对应名称的常量，同时会有两个map方便使用：

    ```
    var Test_Status_name = map[int32]string{
        0: "OK",
        1: "FAIL",
    }
    var Test_Status_value = map[string]int32{
        "OK":   0,
        "FAIL": 1,
    }
    ```

    ### 1.1.3. Service

    定义一个简单的Service，`TestService`有一个方法`Test`，接收一个`Request`参数，返回`Response`：

    ```
    // TestService 测试服务
    service TestService {
        // Test 测试方法
        rpc Test(Request) returns (Response) {};
    }
    
    // Request 请求结构
    message Request {
        string name = 1;
    }
    
    // Response 响应结构
    message Response {
        string message = 1;
    }
    ```

    转换结果：

    ```
    // 客户端接口
    type TestServiceClient interface {
        // Test 测试方法
        Test(ctx context.Context, in *Request, opts ...grpc.CallOption) (*Response, error)
    }
    
    // 服务端接口
    type TestServiceServer interface {
        // Test 测试方法
        Test(context.Context, *Request) (*Response, error)
    }
    ```

    生成的go代码中包含该Service定义的接口，客户端接口已经自动实现了，直接供客户端使用者调用，服务端接口需要由服务提供方实现。



grpc

根据proto 文件生成go 

protoc --go_out=plugins=grpc:. -I=${GOPATH}/src -I=. *.proto 

protoc --go_out=plugins=grpc:. -I=C:/Users/cq/go/src  -I=. data_service.proto



gRPC主要有4种请求和响应模式，分别是`简单模式(Simple RPC)`、`服务端流式（Server-side streaming RPC）`、`客户端流式（Client-side streaming RPC）`、和`双向流式（Bidirectional streaming RPC）`。 
