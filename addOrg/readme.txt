关于负载均衡的考虑:
负载均衡应该从多个方面入手：
（1）前端和java SDK的交互。
     主要处理前端的http请求，对请求进行分流处理达到负载均衡。属于传统技术范围。这部分内容由java SDK设计者考虑。
（2）排序集群
     fabric1.0之后排序作为一个公共服务，担负整个网络交易排序。目前实现的order排序算法有solo和kafka两种模式。实际生产环境中，考虑到负载的问题，应该将order节点建立在kafka集群的基础上，也就是说应该采用kafka模式设置order节点。
     kafka集群的节点数量最小值为4，这是故障容错(crash fault tolerance) 所需要的最小数值。也就是说， 4个节点可以容许1个节点宕机的情况下，所有的通道能够继续读写且可以创建通道。
     zookeeper节点的数量可以是3、5或者7。它必须是一个奇数来避免分裂(split-brain)情景，大于1以避免单点故障。 通常认为超过7个ZooKeeper服务器则是多余的。
     采用4个kafka节点和3个zookeeper节点构建服务集群，成为order节点的服务基础。kafka的消息架构本身就带有负载均衡的考虑。
（3）fabric CA
     fabric CA提供系统中的证书服务。有两种方式与Fabric CA服务端交互：通过Fabric CA客户端，或者Fabric SDK，所有与Fabric CA的交互都是通过REST APIs来实现的。Fabric CA客户端或者SDK可能会连接到Fabric CA 集群中某一个Fabric CA服务端，户端连接的是一个HA代理节点，这个HA代理节点为Fabric CA集群作负载均衡。
     
