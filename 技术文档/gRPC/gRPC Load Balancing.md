# gRPC负载均衡

## 背景
由于分布式服务的逐渐兴盛，现在每个服务都会部署多个实例，因此如何将请求的流量分担到多个实例上，避免集中请求时单个实例的故障影响整个服务成为了分布式高可用
的关注点，因此提出了负载均衡的概念。  
负载均衡，英文名称为Load Balance，其含义就是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，例如FTP服务器、Web服务器、企业核心应用服务器
和其它主要任务服务器等，从而协同完成工作任务。  
因此在线上部署多个gRPC服务时需要考虑如何设计负载均衡策略来保证服务的稳定和高可用，主要的问题在于gRPC采用的是HTTP/2，建立的是长连接，可以在连接中传输多个
请求，所以在gRPC中的负载均衡策略是需要按照每次调用(Per-Call)而不是每次连接(Per-Connection)来进行请求的分配。

## 负载均衡选择
现在主流的实现负载均衡的方法主要有下面几种
- Proxy Load Balancer(代理负载均衡器)：是目前进行负载均衡的最主要的方式，客户端发送请求到代理负载均衡器，代理负载均衡器将请求分发到服务端可用的实例之一。
代理负载均衡器会监控每个实例的负载请求，并且有着内部实现的负载均衡算法。代理负载均衡器的优点是将负载均衡的策略放在了外部，对客户端是透明的，缺点是更高的延迟，因为
代理负载均衡器需要存储请求和响应的临时副本，这在出现大量请求时会增加请求和响应的延迟。代理负载均衡器一般是在L4(传输层)和L7(应用层)去进行负载均衡。  
![timewoo](https://timewoo.github.io/images/gRPC-proxy_load_balancer.png)
  - L4代理负载均衡器：在传输层中使用TCP连接来进行负载均衡，负载均衡器接收来自客户端的TCP连接，同时打开与后端实例之一的另一个连接，并且在两个连接中
  复制数据，不需要进行额外的处理。L4代理负载均衡器相比L7有着更低的延迟，资源消耗得更少。由于L4在传输层进行负载，只要是在一个TCP连接中的发送数据都是负载
  到同一个后端实例，这在HTTP/1中是可以接受的，因为在HTTP/1中每次请求都会重新建立连接，每次请求都会重新选择一个实例去负载，但是在HTTP/2中使用的是长连接，
  在一个连接中会发送多个请求数据，这样请求数据都会负载到同一个实例中。由于同一个连接会负载到同一个实例上，在配置了自动伸缩的实例扩容之后，当某个实例的
  负载过大时，会触发自动扩容来新增实例，但是负载过大的实例上的连接并不会减少，新增的实例无法分担请求，这样就无法做到实例的自动伸缩。如果需要保证请求的数据不负载到同一个客户端，
  要么客户端定期重连，要么服务端定期重连，但是这样就无法使用HTTP/2的多路复用以及双向流特性。
  - L7负载均衡器：在应用层对传输的请求进行负载均衡，L7负载均衡器解析HTTP/2的请求，根据每一个请求分配不同的后端实例，同时L7负载均衡器和后端实例之间通过HTTP/2进行
  连接，这样就可以在不同的后端实例中分配同一个客户端的流。L7负载均衡器也可以根据HTTP的header将特定请求分配到同一个后端实例。  
    
  因此如果gRPC需要采用代理来实现负载均衡就需要选择支持HTTP/2的L7负载均衡器。目前AWS的ALB支持相关配置。
- Client Load Balancer(客户端负载均衡器)：将负载均衡的逻辑放在客户端去实现，客户端通过名称解析器或者外部配置获取所有的服务器地址，根据内部实现的负载均衡算法，
比如轮询或者随机等，来选择相应的服务端进行连接。  
![timewoo](https://timewoo.github.io/images/gRPC-client_load_balancer.png)  
这种负载均衡的方式将客户端的逻辑变得更重，客户端除了本身的业务逻辑之外，还需要实现负载的算法，服务端的地址检查，服务端的健康检查等逻辑的实现。而且在多语言/版本的
客户端中需要单独维护各自的负载均衡策略，极大地增加了客户端代码逻辑的复杂度。客户端负载均衡的好处是请求不需要经过代理，直接连接到服务端，提高了性能。缺点是需要客户端增加了
更多的处理逻辑。
- External Load Balancer(外部负载均衡器)：专门用来实现负载均衡的服务端，外部负载均衡器提供复杂的负载均衡算法，服务发现以及名称解析等功能，客户端通过和外部负载均衡器
连接，获取负载均衡的配置以及可用的服务端地址列表。外部负载均衡器和后端服务连接，获取服务端的健康状态来通知客户端更新服务端的地址列表。好处是相比较代理负载均衡器，客户端
直接连接服务端，降低延迟，提高性能。相比较客户端负载均衡器，客户端的复杂性降低，不需要维护多语言多版本的复杂均衡逻辑。缺点是需要额外部署外部负载均衡器，需要专门的服务来提供
负载均衡。  
![timewoo](https://timewoo.github.io/images/gRPC-external_load_balancer.png)
## gRPC的负载均衡策略
gRPC推荐使用外部负载均衡器的方式来实现负载均衡，外部负载均衡器给客户端提供服务端的最新的地址列表，客户端内部实现简单的负载算法(轮询)，但是客户端仅有简单的负载算法，并且
不推荐用户扩展客户端的负载算法，相反，应该在外部负载均衡器中实现负载均衡策略。  
下面是gRPC客户端负载均衡的工作流程，负载均衡策略处于名称解析和服务端连接之间。  
![timewoo](https://timewoo.github.io/images/gRPC-load-balancing.png)  
1.客户端启动时通过名称解析器解析服务端名称，获取一个或多个IP地址，每个IP显示它是服务端地址还是外部负载均衡器地址，以及一个服务端配置，该配置指定客户端使用的负载平衡
策略(轮询还是grpclb)。  
2.客户端实例化负载均衡器策略，如果返回的IP地址中有外部负载均衡器地址，则客户端就会忽略服务端配置，直接使用grpclb，相反，则会使用服务端配置来实例化负载均衡策略。如果
没有服务端配置则默认使用选择第一个可用的服务器地址的策略。  
3.负载均衡策略会和每个服务端地址创建一个subchannel。
- 在除了grpclb以外的负载均衡策略中，会和名称解析器返回的每一个服务端地址建立subchannel，同时会忽略外部负载均衡器的地址。
- 在grpclb的策略中，工作流程如下：  
  a.grpclb策略会和名称解析器返回的其中一个外部负载均衡器建立stream连接，策略请求外部负载均衡器获取服务端地址，用来在客户端初次请求时服务端地址。  
  - 在grpclb策略中，名称解析器返回的非外部负载均衡器地址将会作为回退，防止在LB策略启动时无法连接到外部负载均衡器。  
  
  b.如果在外部负载均衡器中配置了获取服务端的负载情况，则服务端会向外部负载均衡器发送负载相关信息。  
  c.外部负载均衡器返回服务端的地址列表给客户端的grpclb策略，grpclb策略将会和地址列表上每个服务端地址建立subchannel。
  
4.对于每个RPC请求，客户端负载均衡策略将选择哪个subchannel去发送。
  - 在grpclb策略中，客户端将会根据外部负载均衡器返回的服务端地址顺序发送请求，如果返回的服务端地址为空，则会阻塞到有服务端地址返回为止。
## reference：
https://grpc.io/blog/grpc-load-balancing  
https://github.com/grpc/grpc/blob/master/doc/load-balancing.md  
https://majidfn.com/blog/20201222-grpc-load-balancing