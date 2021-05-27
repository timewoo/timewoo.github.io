# gRPC介绍
## RPC介绍
RPC是Remote Procedure Call(远程过程调用)的简称，RPC是一种用于构建基于client-server的分布式系统通信的技术。PRC能够使处于不同
节点的服务之间相互调用像调用本地接口一样，对服务调用方隐藏了通信。在分布式系统中进程间的通信一般是HTTP和RPC两种形式，RPC相比较HTTP
来说有以下几个优点
- 传输效率  
  RPC可以自定义通信协议，可以基于TCP、UDP或者HTTP等
- 性能消耗  
  RPC框架自带高效的序列化机制，序列化和反序列化耗时更低，序列化后的字节数更小
- 负载均衡  
  RPC框架通常自带负载均衡策略，而HTTP则需要外部应用例如Nginx的支持
- 服务治理  
  RPC下游的服务新增，重启以及下线都能够自动通知上游使用者，HTTP需要事先通知并修改相关配置
## gRPC简介
gRPC是由Google开发并且开源的的高效率RPC框架，主要是面向于移动应用开发并且基于HTTP/2协议标准设计，同时基于ProtoBuf(Protocol 
Buffers)序列化协议开发，并且支持多种语言。gRPC主要有以下几个优点
- 语言中立  
  支持C，Java，Go，Python等多种语言来构建RPC服务，可以使不同语言构建的微服务之间的RPC通信更为简单。
- 基于IDL(Interface Definition Language,接口定义语言)定义服务  
  gRPC使用ProtoBuf来定义服务，ProtoBuf是由Google开发的一种与平台和语言无关，可扩展的数据序列化协议(类似XML、JSON)，压缩和传输效率
  高，语法简单，表达能力强。ProtoBuf可以根据定义的.proto文件来生成指定语言的服务端接口，客户端Stub和gRPC通信的数据结构，更加易于开发。
- 基于HTTP/2协议  
  HTTP/2相较于HTTP 1.x新增了消息头压缩，双向流，单TCP的多路复用，服务端推送等，这些特性使得gRPC更加适用于移动场景下的客户端和服务
  端之间的通信。

gRPC的服务大体架构如下  
![timewoo](https://timewoo.github.io/images/gRPC.svg)  
gRPC服务端使用C++构建，客户端使用Ruby或者Java构建，客户端通过一个Stub存根(代理)对象发起RPC调用，请求和响应通过ProtoBuf来进行序列化。
通过gRPC，使得远程服务的调用对于使用者来说更加简单和透明，底层的传输方式，序列方式以及通信细节等都不需要关注。
## 使用ProtoBuf进行服务定义
客户端在调用服务端的远程接口之前，需要约定接口的方法签名，请求和响应数据结构等，这个过程称之为服务定义。服务定义需要IDL(接口定义语言)来完成，
gRPC中默认使用ProtoBuf来定义服务。ProtoBuf是Google开源的一款序列化框架，内部定义了一种数据序列化的协议，其独立于语言和平台，支持多种语言
的实现，通过定义.proto文件，使用其编译器生成特定语言服务端接口、客户端Stub代码以及通信的数据结构。服务端和客户端之间的数据传输如下  
![timewoo](https://timewoo.github.io/images/ProtoBuf.png)  
ProtoBuf目前有proto2和proto3两个版本，proto3相比较proto2语法上更为简化，提供了一些新功能，并且支持更多的语言。gPRC推荐使用proto3来
定义服务文件
### 定义Message(消息)
消息是指PRC接口的请求参数和响应结果的的数据结构，类似Rest的Request和Response。下面是一个定义的message
```protobuf
// 指定proto3语法
syntax = "proto3";
//消息的主体结构
message Request{
  repeated bool result = 1;
}
```
消息定义的关键字是message，类似Java中的class，一个message就相当于是Java中的一个类。message中可以定义多个字段类型，定义的字段格式是
```
[Field Rules] [Field Type] [Field Name] = [Field Number]
```
- Field Rules用来修饰字段的规则，主要分为  
  - singular：message最多拥有一个该字段的值，proto3默认的字段修饰。
  - repeated：message可以拥有多个该字段的值，类似Java中的数组。
- Field Type是字段的数据类型，主要分为
  - 基本数据类型  
    ![timewoo](https://timewoo.github.io/images/proto3_type.png)
    基本上int32对应Java的int，int64对应Java的long，string对应Java的String，在proto3中在反序列化时会将未设置的字段值统一设置成默认值。
    - string默认值是空字符串
    - bytes默认值是空bytes
    - bool默认值是false
    - 数字类型默认值是0
    - enums默认值是第一个枚举常量值，默认是0
    - repeated默认值是空数组
    - message类型的默认值取决与具体语言，Java中是null  
      
    因此在proto3中无法区分字段是未设置值还是设置成了默认值，所以在后端校验字段值时需要注意，尽量不要使用字段的默认值去校验。
  - 复杂数据类型  
    enums和Map等  
    ```protobuf
    enum Corpus{
      UNIVERSAL = 0;
      WEB = 1;
      IMAGES = 2;
      LOCAL = 3;
      NEWS = 4;
      PRODUCTS = 5;
      VIDEO = 6;
    }
    map<key_type, value_type> map_field = N;
    ```
    还可以创建嵌套类型
    ```protobuf
    syntax="proto3";
    message Request{
      message Inner{
        string name = 1;
      }
    }
    ```
- Field Name是字段名
- Field Number代表的是字段的唯一编号，用于标识在二进制编码中的字段，所以在确定编号后不应该更改编号。而且编号范围在1-15的字段会需要一个
  字节来编码，16-2047的字段会需要两个字节来编码，所以尽量将频繁出现的字段设置1-15。同时编号无法设置成19000-19999，因为这个范围内的数字是保留数。
### 定义Service(服务接口)
服务是指RPC定义的通信接口，类似Java中的interface，ProtoBuf的编译器将会根据选择的语言生成服务接口和客户端Stubs。下面是一个定义的service
```protobuf
syntax="proto3";
service SearchRequest{
  rpc Search(Request) returns (Response);
}
message Request{
  string name=1;
}
message Response{
  int64 id = 1;
}
```
- service 代表一个服务接口类
- 定义一个服务接口的格式如下
  ```
  rpc [Service Name] ([Request]) returns ([Respones])
  ```
gRPC中定义了四种服务接口类型，主要分为三类
- Unary RPCs是最简单的RPC调用，客户端发送请求，从服务端获取响应结果，就像简单的本地方法调用。
  ```protobuf
  rpc Search(Request) returns (Response)
  ```
  Unary RPCs是同步方法，客户端发送请求后会一直阻塞等待服务端返回响应结果或者超时。
- Unidirectional Streams是单向流，指在一个RPC调用中可以处理服务端或客户端的stream流式消息，分为客户端流和服务端流
  - Server streaming RPCs是服务端流式RPC调用，客户端发送请求，服务端返回stream流式响应，客户端从流中按顺序获取结果。
    ```protobuf
    rpc Search(Request) returns (stream Response)
    ```
    服务端在stream流中返回N个消息作为响应，并且每个消息是单独发送的，客户端从stream流中按照顺序读取响应消息，可以在单个RPC调用中处理服务端流式响应。
    服务端在发送所有的消息之后才会返回服务端的status details(status code,status message)和trailing metadata给客户端。  
    好处是当客户端需要获取的响应消息较多时，服务端采用stream流式返回，能够减少服务器的瞬时压力，同时客户端可以异步的进行消息的处理，更有及时性。  
    ![timewoo](https://timewoo.github.io/images/gRPC-serverStream.png)
  - Client streaming RPCs是客户端流式RPC调用，客户端发送stream流请求，并且等待服务端读取消息并且返回响应，服务端从流中按顺序获取结果。
    ```protobuf
    rpc Search(stream Request) returns (Response)
    ```
    客户端在stream流中发送N个消息作为请求，并且每个消息是单独发送的，服务端可以读取任一或全部请求后再返回响应，客户端会一直等待服务端返回响应或超时。  
    好处是当客户端需要发送大量消息或者需要持续上报消息时，服务端可以持续接收服务客户端的消息，接收完成后返回客户端响应。
- Bidirectional streaming RPCs是双向流，指在一个RPC调用中服务端和客户端使用读写流来发送消息，两个流之间是相互独立的，所以服务端和客户端可以根据
  各自的顺序进行消息处理，比如服务端可以在获取客户端所有消息后返回，或者边读取边写入，读取时会按照流中的消息的顺序去读取。
  ```protobuf
  rpc Search(stream Request) returns (stream Response)
  ```
  好处是客户端和服务端可以按照任意的顺序去处理发送的消息，客户端和服务端可以并行进行消息的处理，有更高的灵活性。双向流充分利用了HTTP/2的多路复用功能，
  实现了客户端和服务端的全双工通信。
  ![timewoo](https://timewoo.github.io/images/gRPC-bidirectionalStream.png)
## 生成代码
在定义了gRPC通信的方法，请求和响应参数后，可以通过protocol编译器来生成指定语言的代码，现在protocol编译器支持生成Java，Kotlin，Python，C++，Go，
Ruby，Objective-C和C#的代码。protocol生成代码有以下两种方式
- 命令行执行protoc进行代码生成  
  首先需要下载protocol的编译器，地址为https://github.com/protocolbuffers/protobuf/releases/latest ，下载完成后配置protoc环境变量后执行下列命令生成指定语言的代码
  ```
  protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
  ```
  --proto_path：指定proto文件的目录，如果未指定，默认是在当前目录下。  
  --cpp_out/--java_out...：指定语言生成的代码的目录，--java_out=DST_DIR指定生成java代码的存放路径。   
  path/to/file.proto：指定需要生成的proto文件，可以指定多个。  
  由于protocol本身不提供RPC的实现，所以以上的命令只会生成proto文件定义的message代码，如果需要生成service代码则需要下载对应RPC实现插件，这里使用gRPC Java的插件生成
  对应的gRPC代码，首先选择https://mvnrepository.com/artifact/io.grpc/protoc-gen-grpc-java 的版本，点击"Files"下载对应系统的插件，然后执行下列命令生成gRPC代码
  ```
  protoc --proto_path=IMPORT_PATH --plugin=protoc-gen-grpc-java=PLUGIN_DIR --grpc-java_out=DST_DIR path/to/file.proto
  ```
  --plugin：指定插件的地址  
  执行上述命令后就会生成gRPC的java实现代码。
- idea插件生成代码  
  除了命令之外，可以使用idea的自动生成代码的插件，根据https://github.com/google/protobuf-gradle-plugin 配置idea的插件后就可以实现proto文件的
  代码自动生成。 
  
主要会生成了两个.java文件，一个是定义的请求响应的model类，一个是实现的gRPC类。结构如下
![timewoo](https://timewoo.github.io/images/gRPC-generate.png)
## 构建简单的gRPC服务
编写一个gRPC版本的helloWorld来展示gRPC如何构建服务
- 定义proto文件
  ```protobuf
  //指定proto语法版本
  syntax = "proto3";
  // 设置message是否拆分，默认为false，生成的message放在一个java类中
  option java_multiple_files = true;
  // 设置生成的java的package路径
  option java_package = "com.example.grpcinterface";
  // 接口服务
  service Greeter{
  // rpc接口
    rpc sayHello(HelloRequest) returns (HelloReply);
  }
  // 请求model
  message HelloRequest{
    string name = 1;
  }
  // 响应model
  message HelloReply{
    string message = 1;
  }
  ```
- 配置插件，自动生成java代码
  ![timewoo](https://timewoo.github.io/images/gRPC-demo-dir.png)
- 实现gRPC服务端  
  a.首先引入依赖
  ```
    implementation 'io.grpc:grpc-netty-shaded:1.36.0'
    implementation 'io.grpc:grpc-protobuf:1.36.0'
    implementation 'io.grpc:grpc-stub:1.36.0'
  ```
  b.服务端创建具体的服务接口实现类，扩展gRPC自动生成的抽象类。
  ```java
  public static class GreetImpl extends GreeterGrpc.GreeterImplBase{

      @Override
      public void sayHello(HelloRequest request, StreamObserver<HelloReply> responseObserver) {
          HelloReply helloReply = HelloReply.newBuilder().setMessage("Hello "+request.getName()).build();
          responseObserver.onNext(helloReply);
          responseObserver.onCompleted();
      }
  }
  ```
  c.创建服务的启动参数
  ```
  int port = 8081;
  Server server = ServerBuilder.forPort(port).addService(new GreetImpl()).build().start();
  log.info("start server");
  server.awaitTermination();
  ```
  完整的代码如下
  ```java
  @Slf4j
  public class GrpcServer {

    public static void main(String[] args) throws InterruptedException, IOException {
        int port = 8081;
        Server server = ServerBuilder.forPort(port).addService(new GreetImpl()).build().start();
        log.info("start server");
        server.awaitTermination();
    }

    public static class GreetImpl extends GreeterGrpc.GreeterImplBase{

        @Override
        public void sayHello(HelloRequest request, StreamObserver<HelloReply> responseObserver) {
            HelloReply helloReply = HelloReply.newBuilder().setMessage("Hello "+request.getName()).build();
            responseObserver.onNext(helloReply);
            responseObserver.onCompleted();
        }
    }
  }
  ```
  gRPC服务端创建的过程如下
  ![timewoo](https://timewoo.github.io/images/gRPC-server.png)
- 实现gRPC客户端  
  a.引入依赖，和服务端相同
  ```
    implementation 'io.grpc:grpc-netty-shaded:1.36.0'
    implementation 'io.grpc:grpc-protobuf:1.36.0'
    implementation 'io.grpc:grpc-stub:1.36.0'
  ```
  b.根据服务端地址创建ManagedChannel，创建客户端调用的stub，客户端通过stub发起rpc调用，获取服务端响应
  ```java
  @Slf4j
  public class GrpcClient {

    public static void main(String[] args) {
        String address = "localhost";
        int port = 8081;
        ManagedChannel channel = ManagedChannelBuilder.forAddress(address, port).usePlaintext().build();
        GreeterGrpc.GreeterBlockingStub blockingStub = GreeterGrpc.newBlockingStub(channel);
        Scanner scanner = new Scanner(System.in);
        while (true) {
            String name = scanner.next().trim();
            HelloRequest helloRequest = HelloRequest.newBuilder().setName(name).build();
            HelloReply helloReply = blockingStub.sayHello(helloRequest);
            String message = helloReply.getMessage();
            log.warn("get message:{}", message);
        }
    }
  }
  ```
  gRPC客户端的调用流程如下  
  ![timewoo](https://timewoo.github.io/images/gRPC-client.png)
## 总结
gRPC使用protobuf作为数据传输的载体，序列化和反序列化性能高，同时底层采用HTTP/2协议传输，可以使用HTTP/2的多路复用以及双向流特性来减少请求的延迟。
不足之处在于protobuf对于泛型支持不好，同时由于使用的是HTTP/2协议，部分云不支持基于HTTP/2的负载均衡，而且对于服务发现、负载均衡以及服务治理等方面的需求
需要自己实现，其实gRPC适用于Istio和K8s来进行微服务的调度和监控的场景，将服务发现、负载均衡、链路追踪以及服务治理等实现交给Istio和K8s来处理，gRPC
服务只用来进行业务的处理。但是目前spring cloud和K8s有部分功能上的重叠，像服务发现和负载均衡等服务治理功能交给spring cloud的组件来实现，这样的情况下
使用gRPC就需要自己去实现服务发现和负载均衡等功能。所以如果采用Istio方式进行服务的部署的情况下，gRPC是一个不错的选择，如果是在spring cloud下进行
服务治理，则在使用gRPC时需要更多的考虑。
  
  