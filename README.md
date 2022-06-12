<p align="center">
	<img alt="logo" src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a70070ebc4546aba734e3ba97e7b193~tplv-k3u1fbpfcp-zoom-1.image" width="50%">
</p>
<p align="center">
	<strong>基于配置中心的轻量级动态线程池，内置监控告警功能，可通过SPI自定义扩展实现</strong>
</p>

<p align="center">
  <a href="https://gitee.com/dromara/dynamic-tp"><img src="https://gitee.com/dromara/dynamic-tp/badge/star.svg"></a>
  <a href="https://gitee.com/dromara/dynamic-tp/members"><img src="https://gitee.com/dromara/dynamic-tp/badge/fork.svg"></a>
  <a href="https://github.com/dromara/dynamic-tp"><img src="https://img.shields.io/github/stars/dromara/dynamic-tp?style=flat-square&logo=github"></a>
  <a href="https://github.com/dromara/dynamic-tp/network/members"><img src="https://img.shields.io/github/forks/dromara/dynamic-tp?style=flat-square&logo=GitHub"></a>
  <a href="https://github.com/dromara/dynamic-tp/blob/master/LICENSE"><img src="https://img.shields.io/github/license/dromara/dynamic-tp.svg?style=flat-square"></a>
  <a target="_blank" href="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/530709dc29604630b6d1537d7c160ea5~tplv-k3u1fbpfcp-watermark.image"><img src='https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddfaed2cce2a47608fb0c0c375a10f08~tplv-k3u1fbpfcp-zoom-1.image' alt='备注加群'></a>
</p>

<p align="center">
    官网： <a href="https://dynamictp.cn">https://dynamictp.cn</a> 🔥
</p>

---

## 背景

**使用 ThreadPoolExecutor 过程中你是否有以下痛点呢？**

> 1.代码中创建了一个 ThreadPoolExecutor，但是不知道那几个核心参数设置多少比较合适
>
> 2.凭经验设置参数值，上线后发现需要调整，改代码重启服务，非常麻烦
>
> 3.线程池相对开发人员来说是个黑盒，运行情况不能感知到，直到出现问题

如果你有以上痛点，动态可监控线程池（DynamicTp）或许能帮助到你。

如果看过 ThreadPoolExecutor 的源码，大概可以知道其实它有提供一些 set 方法，可以在运行时动态去修改相应的值，这些方法有：

```java
public void setCorePoolSize(int corePoolSize);
public void setMaximumPoolSize(int maximumPoolSize);
public void setKeepAliveTime(long time, TimeUnit unit);
public void setThreadFactory(ThreadFactory threadFactory);
public void setRejectedExecutionHandler(RejectedExecutionHandler handler);
```

现在大多数的互联网项目其实都会微服务化部署，有一套自己的服务治理体系，微服务组件中的分布式配置中心扮演的就是动态修改配置，
实时生效的角色。那么我们是否可以结合配置中心来做运行时线程池参数的动态调整呢？答案是肯定的，而且配置中心相对都是高可用的，
使用它也不用过于担心配置推送出现问题这类事儿，而且也能减少研发动态线程池组件的难度和工作量。

**综上，可以总结出以下的背景**

- 广泛性：在 Java 开发中，想要提高系统性能，线程池已经是一个 90%以上的人都会选择使用的基础工具

- 不确定性：项目中可能会创建很多线程池，既有 IO 密集型的，也有 CPU 密集型的，但线程池的参数并不好确定；需要有套机制在运行过程中动态去调整参数

- 无感知性，线程池运行过程中的各项指标一般感知不到；需要有套监控报警机制在事前、事中就能让开发人员感知到线程池的运行状况，及时处理

- 高可用性，配置变更需要及时推送到客户端；需要有高可用的配置管理推送服务，配置中心是现在大多数互联网系统都会使用的组件，与之结合可以大幅度减少开发量及接入难度

---

## 简介

**基于以上背景分析，我们对线程池 ThreadPoolExecutor 做一些扩展增强，主要实现以下目标**

> 1.实现对运行中线程池参数的动态修改，实时生效
>
> 2.实时监控线程池的运行状态，触发设置的报警策略时报警，报警信息推送办公平台
>
> 3.定时采集线程池指标数据，配合像 grafana 这种可视化监控平台做大盘监控

**经过多个版本的迭代，目前最新版本具有以下特性** ✅

- **代码零侵入**：所有配置都放在配置中心，对业务代码零侵入

- **轻量简单**：基于 springboot 实现，引入 starter，接入只需简单4步就可完成，顺利3分钟搞定

- **高可扩展**：框架核心功能都提供 SPI 接口供用户自定义个性化实现（配置中心、配置文件解析、通知告警、监控数据采集、任务包装等等）

- **线上大规模应用**：参考[美团线程池实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)，美团内部已经有该理论成熟的应用经验

- **通知报警**：提供多种报警维度（配置变更通知、活性报警、容量阈值报警、拒绝触发报警、任务执行或等待超时报警），已支持企业微信、钉钉、飞书报警，同时提供 SPI 接口可自定义扩展实现

- **监控**：定时采集线程池指标数据，支持通过 MicroMeter、JsonLog 日志输出、Endpoint 三种方式，可通过 SPI 接口自定义扩展实现

- **任务增强**：提供任务包装功能，实现TaskWrapper接口即可，如 TtlTaskWrapper 可以支持线程池上下文信息传递，以及给任务设置标识id，方便问题追踪

- **兼容性**：JUC 普通线程池也可以被框架监控，@Bean 定义时加 @DynamicTp 注解即可

- **可靠性**：框架提供的线程池实现 Spring 生命周期方法，可以在 Spring 容器关闭前尽可能多的处理队列中的任务

- **多模式**：参考Tomcat线程池提供了 IO 密集型场景使用的 EagerDtpExecutor 线程池

- **支持多配置中心**：基于主流配置中心实现线程池参数动态调整，实时生效，已支持 Nacos、Apollo、Zookeeper、Consul，同时也提供 SPI 接口可自定义扩展实现

- **中间件线程池管理**：集成管理常用第三方组件的线程池，已集成Tomcat、Jetty、Undertow、Dubbo、RocketMq、Hystrix等组件的线程池管理（调参、监控报警）

---

## 设计

**框架功能大体可以分为以下几个模块**

> 1.配置变更监听模块
>
> 2.服务内部线程池管理模块
>
> 3.三方组件线程池管理模块
>
> 4.监控模块
>
> 5.通知告警模块

- 配置变更监听模块

  1.监听特定配置中心的指定配置文件（已实现 Nacos、Apollo、Zookeeper、Consul），可通过内部提供的SPI接口扩展其他实现

  2.解析配置文件内容，内置实现 yml、properties、json 配置文件的解析，可通过内部提供的 SPI 接口扩展其他实现

  3.通知线程池管理模块实现参数的刷新

- 服务内部线程池管理模块

  1.服务启动时从配置中心拉取配置，生成线程池实例注册到内部线程池注册中心以及Spring容器中

  2.接受配置监听模块的刷新事件，实现线程池参数的刷新

  3.代码中通过依赖注入（推荐）或者 DtpRegistry.getExecutor() 方法根据线程池名称来获取线程池实例

- 三方组件线程池管理

  1.服务启动获取第三方中间件的线程池，被框架管理起来

  2.接受参数刷新、指标收集、通知报警事件，进行相应的处理

- 监控模块

  实现监控指标采集以及输出，默认提供以下三种方式，也可通过内部提供的 SPI 接口扩展其他实现

  1.默认实现 JsonLog 输出到磁盘，可以自己采集解析日志，存储展示

  2.MicroMeter采集，引入 MicroMeter 相关依赖，暴露相关端点，采集指标数据，结合 Grafana 做监控大盘

  3.暴雷自定义 Endpoint 端点（dynamic-tp），可通过 http 方式实时访问

- 通知告警模块

  对接办公平台，实现通知告警功能，已支持钉钉、企微、飞书，可通过内部提供的 SPI 接口扩展其他实现，通知告警类型如下

  1.线程池主要参数变更通知

  2.阻塞队列容量达到设置的告警阈值

  3.线程池活性达到设置的告警阈值

  4.触发拒绝策略告警，格式：A/B，A：该报警项前后两次报警区间累加数量，B：该报警项累计总数

  5.任务执行超时告警，格式：A/B，A：该报警项前后两次报警区间累加数量，B：该报警项累计总数

  6.任务等待超时告警，格式：A/B，A：该报警项前后两次报警区间累加数量，B：该报警项累计总数

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91ea4c3e1166426e8dca9903dacfd9eb~tplv-k3u1fbpfcp-zoom-1.image)

---

## 使用

- 接入步骤

  1.引入相应配置中心的依赖，具体见下述 mavne依赖

  2.配置中心配置线程池实例，配置见下述（给出的是全配置项，不用的可以删除）

  3.启动类加 @EnableDynamicTp 注解

  4.使用 @Resource 或 @Autowired 进行依赖注入，或通过 DtpRegistry.getDtpExecutor("name")获取

  5.通过以上4步就可以使用了，是不是感觉超简单

- maven 依赖，见官网文档，[maven 依赖](https://dynamictp.cn/guide/use/maven.html)

- 线程池配置，见官网文档，[配置文件](https://dynamictp.cn/guide/use/config.html)

- 代码使用，见官网文档，[代码使用](https://dynamictp.cn/guide/use/code.html)

- 更详细使用实例请参考 `example` 工程

---

## 通知报警

- 触发报警阈值会推送相应报警消息（活性、容量、拒绝、任务等待超时、任务执行超时），且会高亮显示相应字段
  
  更多见官网文档，[通知报警](https://dynamictp.cn/guide/notice/alarm.html)

![告警](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d65151e3e9ca460eac18f30ea6be05d3~tplv-k3u1fbpfcp-zoom-1.image)


- 配置变更会推送通知消息，且会高亮变更的字段

![变更通知](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30093a6ede914887bb5566139352fb8b~tplv-k3u1fbpfcp-zoom-1.image)


---

## 监控

![监控数据](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec5a7d1a31e7418ba5d9a101a5c03826~tplv-k3u1fbpfcp-zoom-1.image)

通过 collectType 属性配置监控指标采集类型，默认 logging

- MicroMeter：通过引入相关 MicroMeter 依赖采集到相应的平台（如 Prometheus，InfluxDb...）

- Logging：定时采集指标数据以 Json 日志格式输出磁盘，
  地址 ${logPath}/dynamictp/${appName}.monitor.log

  ```bash
  {"datetime": "2022-04-17 11:35:15.208", "app_name": "dynamic-tp-nacos-cloud-demo", "thread_pool_metrics": {"activeCount":0,"queueSize":0,"largestPoolSize":0,"poolSize":0,"rejectHandlerName":"CallerRunsPolicy","queueCapacity":2000,"fair":false,"queueTimeoutCount":0,"rejectCount":0,"waitTaskCount":0,"taskCount":0,"runTimeoutCount":0,"queueRemainingCapacity":2000,"corePoolSize":4,"queueType":"VariableLinkedBlockingQueue","completedTaskCount":0,"dynamic":true,"maximumPoolSize":6,"poolName":"dtpExecutor1"}}
  {"datetime": "2022-04-17 11:35:15.209", "app_name": "dynamic-tp-nacos-cloud-demo", "thread_pool_metrics": {"activeCount":0,"queueSize":0,"largestPoolSize":0,"poolSize":0,"rejectHandlerName":"CallerRunsPolicy","queueCapacity":2000,"fair":false,"queueTimeoutCount":0,"rejectCount":0,"waitTaskCount":0,"taskCount":0,"runTimeoutCount":0,"queueRemainingCapacity":2000,"corePoolSize":2,"queueType":"TaskQueue","completedTaskCount":0,"dynamic":true,"maximumPoolSize":4,"poolName":"dtpExecutor2"}}
  {"datetime": "2022-04-17 11:35:15.209", "app_name": "dynamic-tp-nacos-cloud-demo", "thread_pool_metrics": {"activeCount":0,"queueSize":0,"largestPoolSize":0,"poolSize":0,"queueCapacity":2147483647,"fair":false,"queueTimeoutCount":0,"rejectCount":0,"waitTaskCount":0,"taskCount":0,"runTimeoutCount":0,"queueRemainingCapacity":2147483647,"corePoolSize":1,"queueType":"LinkedBlockingQueue","completedTaskCount":0,"dynamic":false,"maximumPoolSize":1,"poolName":"commonExecutor"}}
  {"datetime": "2022-04-17 11:35:15.209", "app_name": "dynamic-tp-nacos-cloud-demo", "thread_pool_metrics": {"activeCount":0,"queueSize":0,"largestPoolSize":100,"poolSize":100,"queueCapacity":2147483647,"fair":false,"queueTimeoutCount":0,"rejectCount":0,"waitTaskCount":0,"taskCount":177,"runTimeoutCount":0,"queueRemainingCapacity":2147483647,"corePoolSize":100,"queueType":"TaskQueue","completedTaskCount":177,"dynamic":false,"maximumPoolSize":400,"poolName":"tomcatWebServerTp"}}
  ```

- 暴露 EndPoint 端点(dynamic-tp)，可以通过 http 方式请求
  ```json
  [
      {
          "dtp_name": "remoting-call",
          "core_pool_size": 6,
          "maximum_pool_size": 12,
          "queue_type": "SynchronousQueue",
          "queue_capacity": 0,
          "queue_size": 0,
          "fair": false,
          "queue_remaining_capacity": 0,
          "active_count": 0,
          "task_count": 21760,
          "completed_task_count": 21760,
          "largest_pool_size": 12,
          "pool_size": 6,
          "wait_task_count": 0,
          "reject_count": 124662,
          "reject_handler_name": "CallerRunsPolicy"
      }
  ]
  ```

---

## 联系我

使用过程中有任何问题，或者对项目有什么想法或者建议，可以加入社群，跟群友一起交流讨论。

微信群已满200人，可以关注微信公众号，加我个人微信拉群（备注：dynamic-tp）。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/530709dc29604630b6d1537d7c160ea5~tplv-k3u1fbpfcp-watermark.image)
