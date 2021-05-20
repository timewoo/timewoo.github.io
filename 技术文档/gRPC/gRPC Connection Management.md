# gRPC连接管理
## 背景
gRPC使用HTTP/2作为基础的通信模式，而HTTP/2在请求时会重用已有的连接来减少通信过程中的连接延迟，所以gRPC客户端默认建立的都是HTTP/2
的长连接，因此在长连接中需要考虑如何管理连接，关闭无效连接以及保活等。同时长连接虽然可以保持连接状态，避免再次请求时重新建立连接，降低延迟，但是
在连接不活跃的状态下保持连接会增加服务端的资源消耗，所以为了避免不活跃的连接长期占用服务端的连接资源，还需要配置连接回收的相关策略。
## 连接管理
为了保证gRPC连接的活跃性，健康性和可用性，gRPC利用了很多的组件，最重要的是name resolver(名称解析器)和load balancer(负载均衡器)。name resolver
将名称转化成地址，并将地址交给load balancer，load balancer将根据地址建立连接，并且在连接中进行RPC的负载均衡。当连接失败时，load balancer将会从
地址列表的最后一个进行重连，同时name resolver会重新开始解析主机名。这在代理不可用或者DNS改变时能够保证获取到最新的可用的地址列表，当解析完成时，load balancer
将会获取一个新的地址列表，如果地址列表发生了变化，load balancer可能会降低新列表中不存在地址的连接速度，或者连接到新的地址。
## 识别失败的连接
gRPC连接的可用性取决于识别失败连接的能力，主要有两种类型的连接失败，一种是干净地失败，即失败的信息会在连接双方进行传递，一种是不太干净地失败，即失败的信息
不会在连接双方进行传递。  
前面一种失败的连接主要出现在服务端正常结束连接时，比如服务端正常关闭，或者连接超时通知服务端关闭连接时，TCP将会发送 FIN handshake来正常关闭
HTTP/2的连接，这样gRPC将会触发load balancer重新进行连接。这样不需要额外的操作即可识别失败的连接来进行重连操作。  
后面一种失败的连接主要出现在服务端死亡或者连接一直挂起而没有通知客户端，这样TCP将会进行长达10分钟的重试之后才会判定连接失败。在gRPC中认为在10分钟之后才能判断
连接失败是不可取的，因此gRPC通过设置keepalive来定期发送HTTP/2的ping来确定连接是否正常，如果ping在指定时间内没有返回，gRPC则认为连接失败，将会断开连接并且
开始重连。  
通过使用keepalive，gRPC保证了连接的可用性，并且使用HTTP/2来定期检查连接的状态，所有的这些操作对用户都是不透明，用户只需要在连接池中发送消息，连接的健康状态以及
重连操作都由gRPC自身维护。
## 保持连接活跃
在上面介绍的keepalive是用HTTP/2的ping操作来检查失败的连接，除此以外，keepalive还可以用来向代理发送信号来保持连接不被中断。  
由于gRPC使用HTTP/2来建立长连接，所以需要保证连接的长期存活并且在需要时发送数据，但是在代理一侧，在资源受限的情况下可能会杀死空闲的连接来回收资源，比如Google Cloud Platform
(GCP) load balancer将会把进入空闲状态10分钟以上的连接断开，Amazon Web Services Elastic Load Balancers(AWS ELBs)将会把进入空闲状态60秒以上的连接断开。  
所以gRPC可以通过keepalive来建立非空闲的连接，这样代理杀死空闲连接的规则就会跳过这些连接，保证了长连接的存活。  
keepalive具体的设置规则  
https://github.com/grpc/grpc/blob/master/doc/keepalive.md  
https://github.com/grpc/proposal/blob/master/A8-client-side-keepalive.md


reference：  
https://grpc.io/blog/grpc-on-http2

