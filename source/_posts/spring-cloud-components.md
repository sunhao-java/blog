title: spring-cloud各组件
tags: [spring, spring-cloud]
date: 2018-09-04
---

# spring-cloud各组件

<!-- more -->

1. `Ribbon`，客户端负载均衡，特性有区域亲和、重试机制。
2. `Eureka`，服务注册中心，特性有失效剔除、服务保护。
2. `Hystrix`，客户端容错保护，特性有服务降级、服务熔断、请求缓存、请求合并、依赖隔离。
3. `Dashboard`，`Hystrix`仪表盘，监控集群模式和单点模式，其中集群模式需要收集器`Turbine`配合。
3. `Feign`，声明式服务调用，本质上就是`Ribbon+Hystrix`
4. `Zuul`，API服务网关，功能有路由分发和过滤
5. `Config`，分布式配置中心，支持本地仓库、SVN、Git、Jar包内配置等模式，
6. `Bus`，消息总线，配合Config仓库修改的一种Stream实现，
4. `Stream`，消息驱动，有Sink、Source、Processor三种通道，特性有订阅发布、消费组、消息分区。
6. `Sleuth`，分布式服务追踪，需要搞清楚TraceID和SpanID以及抽样，如何与ELK整合。

独挑大梁，独自启动不需要依赖其它组件。
> 1. `Eureka`
> 2. `Dashboard`
> 3. `Zuul`
> 4. `Config`

# 含义、依赖
1. `Eureka`和`Ribbon`，是最基础的组件，一个注册服务，一个消费服务。
2. `Hystrix`为了优化`Ribbon`、防止整个微服务架构因为某个服务节点的问题导致崩溃，是个**保险丝**的作用。
3. `Dashboard`给`Hystrix`统计和展示用的，而且监控服务节点的整体压力和健康情况。
4. `Turbine`是集群收集器，服务于`Dashboard`的。
5. `Feign`是方便我们程序员些更优美的代码的。
6. `Zuul`是加在整个微服务最前沿的防火墙和代理器，隐藏微服务结点IP端口信息，加强安全保护的。
7. `Config`是为了解决所有微服务各自维护各自的配置，设置一个同意的配置中心，方便修改配置的。
8. `Bus`是因为`Config`修改完配置后各个结点都要refresh才能生效实在太麻烦，所以交给bus来通知服务节点刷新配置的。
9. `Stream`是为了简化研发人员对MQ使用的复杂度，弱化MQ的差异性，达到程序和MQ松耦合。
10. `Sleuth`是因为单次请求在微服务节点中跳转无法追溯，解决任务链日志追踪问题的。