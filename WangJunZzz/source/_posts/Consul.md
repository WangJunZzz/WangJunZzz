---
title: Consul
date: 2019-07-21 22:46:21
tags:
- Consul
categories: 
- 微服务
---

### Consul
+ Consul包含多个组件,但是作为一个整体,为你的基础设施提供服务发现和服务配置的工具。
1. 服务发现：Consul的客户端可提供一个服务，比如api或者mysql，另外一些客户端可使用Consul去发现一个指定的服务提供者，通过DNS或者Http应用程序很容易找到他依赖的服务。
2. 健康检查：Consul客户端可提供任意数量的健康检查，指定一个服务（比如一个api是否返回了200）或者使用本地节点（比如内存是否大于90%），这个信息可由opreator用来监视集群的健康，被服务发现组件用来避免将流量发送到不健康的主机。
3. Key/Value存储：应用程序可根据自己需要使用的Consul的层级的Key/Value存储，比如动态配置，功能标记，协调，领袖选举等。
4. 多数据中心：Consul支持开箱即用的多数据中心，这意味着用户不需要担心建立额外的抽象层让业务扩展到多个区域，Consul面对DevOps和应用开发者友好，是他适合现代弹性的基础设置。

### Consul使用场景
1. Docker 实例的注册与配置共享。
2. Coreos 实例的注册与配置共享。
3. SaaS 应用的配置共享、服务发现和健康检查。
4. vitess 集群。
5. 与 confd 服务集成，动态生成 nginx 和 haproxy 配置文件。

### Consul优势
- 市面现在有很多类似的软件比如：zookeeper 、Etcd、doozerd、eureka，Consul 相比这些软件有什么优势呢？
1. 使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 采用的是 Paxos, 而 etcd 使用的则是 Raft。
2. 支持 多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障，而其部署则需要考虑网络延迟，分片等情况等. zookeeper 和 etcd 均不提供多数据中心功能的支持。
3. 支持 健康检查。 etcd 不提供此功能。
4. 支持 HTTP、DNS 和 GPRS 协议接口。 zookeeper 的集成较为复杂，etcd 只支持 http 协议。
5. 官方提供 WEB管理界面，etcd 无此功能。

### 名词解释
1. Client：表示 Consul 客户端模式，是 Consul 节点的一种模式，所有注册到 Client 节点的服务会被转发到 Server 。本身无状态不持久化如何数据。Client 通过 HTTP、DNS、GRPC 接口请求转发给局域网内的服务端集群。
2. Server：表示 Consul 的服务端模式， Server 功能和 Client 都一样，不同的是 Server 持久化数据到本地。在局域网内与本地 Client 通讯，通过广域网与其他数据中心通讯。每个数据中心的 Server 数量推荐为 3 个或是 5 个。
3. Server-Leader ：表示这个 Server 是它们的老大，它和其它 Server 不一样的一点是，它需要负责同步注册的信息给其它的 Server 节点，同时也要负责各个节点的健康监测。如果 Leader 宕机了，通数据中心的所有 Server 内部会使用 Raft 算法来在其中选取一个 Leader 出来。
4. Agent ：Agent 是 Consul 的核心进程，Agent 的工作是维护成员关系信息、注册服务、健康检查、响应查询等等。Consul 集群的每一个节点都必须运行 agent 进程。

### Consul的工作原理
1. 使用微服务架构。传统的单体架构不够灵活不能很好的适应变化，从而向微服务架构进行转换，而伴随着大量服务的出现，管理运维十分不便，于是开始搞一些自动化的策略，服务发现应运而生。但是引入服务发现就可能引入一些技术栈，增加系统总体的复杂度。
 ![](/images/consul.png)
2. 在服务器Server1、Server2、Server3上分别部署了Consul Server，假设他们选举了Server2上的Consul Server节点为Leader.
3. 在服务器Server4和Server5上通过Consul Client分别注册Service A、B、C。服务注册到Consul可以通过HTTP API（8500端口）的方式，也可以通过Consul配置文件的方式。Consul Client可以认为是无状态的，它将注册信息通过RPC转发到Consul Server，服务信息保存在Server的各个节点中，并且通过Raft实现了强一致性。
4. 在服务器Server6中Program D需要访问Service B，这时候Program D首先访问本机Consul Client提供的HTTP API，本机Client会将请求转发到Consul Server，Consul Server查询到Service B当前的信息返回，最终Program D拿到了Service B的所有部署的IP和端口，然后就可以选择Service B的其中一个部署并向其发起请求了。如果服务发现采用的是DNS方式，则Program D中直接使用Service B的服务发现域名，域名解析请求首先到达本机DNS代理，然后转发到本机Consul Client，本机Client会将请求转发到Consul Server，Consul Server查询到Service B当前的信息返回，最终Program D拿到了Service B的某个部署的IP和端口。
+ http://blog.didispace.com/consul-service-discovery-exp

### 参数说明
1. advertise：通知展现地址用来改变我们给集群中的其他节点展现的地址，一般情况下-bind地址就是展现地址。
2. bootstrap：	用来控制一个server是否在bootstrap模式，在一个datacenter中只能有一个server处于bootstrap模式，当一个server处于bootstrap模式时，可以自己选举为raft leader。
3. bootstrap-expect：在一个datacenter中期望提供的server节点数目，当该值提供的时候，consul一直等到达到指定sever数目的时候才会引导整个集群，该标记不能和bootstrap共用。
4. bind：该地址用来在集群内部的通讯IP地址，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0。
5. client：consul绑定在哪个client地址上，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1。
6. config-file：明确的指定要加载哪个配置文件。
7. config-dir：配置文件目录，里面所有以.json结尾的文件都会被加载。
8. data-dir：提供一个目录用来存放agent的状态，所有的agent允许都需要该目录，该目录必须是稳定的，系统重启后都继续存在。
9. dc:该标记控制agent允许的datacenter的名称，默认是dc1。
10. encrypt：	指定secret key，使consul在通讯时进行加密，key可以通过consul keygen生成，同一个集群中的节点必须使用相同的key。
11. join：加入一个已经启动的agent的ip地址，可以多次指定多个agent的地址。如果consul不能加入任何指定的地址中，则agent会启动失败，默认agent启动时不会加入任何节点。
12. retry-interval：两次join之间的时间间隔，默认是30s。
13. retry-max：尝试重复join的次数，默认是0，也就是无限次尝试。
14. log-level：consul agent启动后显示的日志信息级别。默认是info，可选：trace、debug、info、warn、err。
15. node：节点在集群中的名称，在一个集群中必须是唯一的，默认是该节点的主机名。
16. protocol：consul使用的协议版本。
17. rejoin：使consul忽略先前的离开，在再次启动后仍旧尝试加入集群中。
18. server：定义agent运行在server模式，每个集群至少有一个server，建议每个集群的server不要超过5个。
19. syslog：开启系统日志功能，只在linux/osx上生效。
20. pid-file：提供一个路径来存放pid文件，可以使用该文件进行SIGINT/SIGHUP(关闭/更新)agent。

``` bash
# 启动Consul和web管理
docker run -d -p 8001:8500  --name=consul_server_1 consul agent -server -bootstrap -ui -node=1 -client='0.0.0.0'
# 查看consul信息
docker exec consul_server_1 consul members
#Node  Address          Status  Type    Build  Protocol  DC   Segment
#1     172.17.0.2:8301  alive   server  1.5.2  2         dc1  <all>
#通过命令获取到引导 Consul 的 Ip 地址 172.17.0.2 
#加入到集群， 命名为 -node=2 、-node=3
 docker run -d  --name=consul_server_2 consul agent -server -bootstrap -ui -node=2 -client='0.0.0.0' -join='172.17.0.2'
 docker run -d  --name=consul_server_3 consul agent -server -bootstrap -ui -node=3 -client='0.0.0.0' -join='172.17.0.2'
 # 添加Client
 docker run -d  --name=consul_server_4 consul agent -client -node=4 -join='172.17.0.2' -client='0.0.0.0'
 # Client需要集群继续添加node5，6就好
```
 ![](/images/node.png)





