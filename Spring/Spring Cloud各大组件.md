### Spring Cloud各大组件

#### 微服务与Spring Cloud

微服务（Microservice）架构模式的目的是将大型的、复杂的、长期运行的应用程序构建为一组组相互配合的服务，每个服务都可以很容易得到改良。Micro这个词意味着每个服务都应该足够小，但是，这里的小不能用代码量来衡量，而应该从业务逻辑上比较——符合单一职责原则。

使用微服务的好处：

+ 复杂度可控

  每一个微服务专注于单一功能，并通过定义良好的接口清晰表述服务边界。由于体积小、复杂度低，每个微服务都可由一个小规模开发团队完全掌控，易于保持高可维护性和开发效率。

+ 独立部署

  每个微服务都可以独立部署，当某个微服务发生变更时，无需编译部署整个应用。

+ 技术选型灵活

  微服务框架下，每个团队可以根据自身服务的需求和行业发展现状，自由选择最合适的技术栈。

+ 容错高

  在单一进程的传统架构下，当某一组件发生故障，故障可能在进程内扩散，形成全局的不可用。在微服务架构下，故障会隔离在单个服务中。若设计良好，其他服务可以通过重试、平稳退化等机制实现应用层面的容错。

+ 可扩展性强

  当应用的不同组件在需求上存在差异时，微服务架构就体现出了其灵活性，因为每个服务都可以根据实际需求进行扩展。

#### Eureka（服务治理）

保证AP，C-S架构。

##### 心跳连接

Client每30秒会发送一次心跳来续约，通过续约告知Server该Client正常运行。默认情况下，当Server90秒内没有收到Client的续约，就会将该实例从注册表中删除。

##### 获取注册表信息

Client从服务器获取注册信息，并缓存在本地，客户端会使用该信息查找其他服务，从而进行远程调用。该注册信息每30秒更新一次。

##### 自我保护机制

某个微服务健康，但是由于网络问题没有收到心跳包，如果注销这个服务不合理。当`EurekaServer`节点在短时间内丢失过多客户端时（默认85%），这个节点就会进入保护模式。Eureka的自我保护机制会保护注册表中的信息，不再删除注册表中的数据。宁可保留错误的服务注册信息，也不盲目注销任何健康的服务实例。

##### 三大角色

+ Eureka Server
+ Service Provider
+ Service Consumer

后两者都是Eureka Client。

##### Eureka对外提供三个功能

+ 服务注册：Eureka Client向Eureka Server注册自己，向服务节点发送实例名、IP、端口等信息。Eureka内部有二层缓存机制来维护整个注册表；
+ 提供注册表：Eureka Client可向服务节点请求服务注册表，并缓存到本地；
+ 同步状态：Eureka通过心跳机制同步客户端状态，维护注册表。

##### Eureka对比ZK

+ ZK保证CP

  Zookeeper有一个leader，当leader挂掉了，要进行选举，在这几十秒之内整个集群是不可用的。ZK的写都是由leader发起，并同步到其他节点，所以保证了强一致性。

+ Eureka满足AP

  Eureka各个节点是平等的，几个节点挂掉不会影响正常节点的工作。但从Eureka Server获取的数据有可能稍微过时，只保证了最终一致性。

##### 使用

引入依赖后，在主类上加上如下注解：

```java
@EnableEurekaServer
```

主要配置：

```yaml
server:
  port: 8000  # 服务端口
spring:
  application:
    name: spring-cloud-eureka-server  # 应用名字，根据它作为服务名
eureka:
  instance:
    hostname: localhost
  client:
    service-url:
      defaultZone: http://localhost:8000/eureka  # eureka server 的地址
    register-with-eureka: false  # 不向eureka注册自己
    fetch-registry: false  # 不向eureka server获取服务列表
```

服务提供者：

不论服务提供者还是服务调用者，对于Eureka Server客户端来说都是客户端，所以都引入Eureka Client依赖。

在启动类上加上

```java
@EnableDiscoveryClient    // 注册中心可以是其他
```

或

```java
@EnableEurekaClient      // 注册中心只能是Eureka
```

注解。

服务提供者配置：

```yaml
server:
  port: 9000  # 服务端口
spring:
  application:
    name: spring-cloud-provider  # 应用名字，根据它作为服务名
eureka:
  instance:
    hostname: localhost
  client:
    service-url:
      defaultZone: http://localhost:8000/eureka  # eureka server 的地址
    register-with-eureka: true  # 向eureka注册自己
    fetch-registry: true  # 向eureka server获取服务列表
```

服务调用者配置：

```yaml
server:
  port: 9001  # 服务端口
spring:
  application:
    name: spring-cloud-consumer  # 应用名字，根据它作为服务名
eureka:
  instance:
    hostname: localhost
  client:
    service-url:
      defaultZone: http://localhost:8000/eureka  # eureka server 的地址
    register-with-eureka: false  # 向eureka注册自己
    fetch-registry: true  # 向eureka server获取服务列表
```





#### Ribbon（客户端负载均衡）

常见负载均衡工具：Nginx，LVS

分类：

+ 集中式：反向代理
+ 进程式（客户端）

负载均衡算法：

轮询：遍历，轮询失败的话，找下一个。超过10个还是失败的话，则轮询结束。

随机：使用JDK提供的随机数，随机选取一个。

轮询失败重试：先按照轮询获取，失败重试轮询下一个，再失败则直接返回失败。

可用过滤策略：先过滤掉连接失败的服务节点，然后从健康的服务节点中用轮询策略选出一个。

响应时间加权：响应时间越小，权重越高，被选中的可能越高。服务刚启动时，由于统计信息不足，先使用轮询策略。

并发量最小可用：选择一个并发量最小的返回，通过一个属性记录并发量。比较耗时。



引入依赖后，在注册`RestTemplate` Bean的方法上加上`@LoadBlanced`注解。

`RestTemplate`调用服务时，使用服务提供者在配置文件中的spring.application.name属性值代替真实IP。



#### Spring Cloud Open Feign（声明式服务调用）

引入依赖后，在应用主类Application通过

```java
@EnableFeignClients
```

注解开启Spring Cloud Open Feign支持。

然后定义一个`HelloServiceFeign`接口，使用注解`@FeignClient`并指定远程服务名：

```java
@FeignClient(value = "hello-service-provider")
public interface HelloServiceFeign {

    @RequestMapping(value = "/demo/getHost", method = RequestMethod.GET)
    public String getHost(String name);

    @RequestMapping(value = "/demo/postPerson", method = RequestMethod.POST, produces = "application/json; charset=UTF-8")
    public Person postPerson(String name);
```

其中使用`@RequestMapping`注解来绑定REST接口。

使用`@FeignClient`修饰一个接口，可以像调用普通的对象一样，调用远程服务。



#### Spring Cloud Hystrix（容错性保护）

熔断器的原理很简单，如同电力过载保护，它可以实现快速失败。如果它在一段时间内检测到许多类似的错误（某个服务不可用），会强迫以后的多个调用快速失败，不再访问远程服务，从而防止应用程序不断尝试执行可能会失败的操作。熔断器也可以每隔一段时间再尝试调用远程服务，诊断错误是否已经修正，如果已经修正，之后的每次调用正常进行。

开启熔断器：

```properties
feign.hystrix.enabled=true
```

然后在`@FeignClient`注解上加上fallback属性。

```java
@FeignClient(value = "service-name", fallback = XX.class)
```

XX类同样实现了`@FeignClient`修饰的接口，方法体里定义了熔断时的处理。

#### Spring Cloud Zuul（API网关）

Spring的微服务节点在行为上没有高低贵贱之分，但角色上会有内部服务跟边缘服务之分。内部服务是对内暴露服务的节点，边缘服务用来对外部提供服务，也就是对外API接口。

为了防止程序被攻击，需要写各种鉴权机制，这些机制在每个微服务节点上都要实现一次。而运维方面边缘服务的Nginx上需要维护一份服务列表和服务地址的路由信息，随着节点的扩展或地址调整要修改这些信息。

为了解决鉴权问题，使业务节点只关心自己的业务，将对权限处理剥离到上层。外部请求首先到Zuul上，在Zuul上对权限进行统一的实现和过滤。

Zuul将自己作为一个微服务节点注册到Eureka上，获取所有的微服务实例信息。

网关的作用：

+ 统一入口：为全部的微服务提供一个唯一入口，网关起到外部和内部的隔离作用，保障了内部服务的安全性。
+ 鉴权校验：识别每个请求的权限，拒绝不符合要求的请求。
+ 动态路由：动态地将请求路由到不同的后端集群中。
+ 减少客户端与服务的耦合：服务可以独立发展，通过网关来做映射。

在主类上加以下注解开启网关：

```java
@EnableZuulProxy
```

访问Zuul的方式:

```
http://zuulHostIp:port/要访问的服务名称/服务中的URL
```



#### Spring Cloud Config（分布式配置中心）

在分布式系统中，为方便服务配置文件统一管理，更易于部署、维护，可以使用分布式配置中心组件。Spring Cloud Config支持配置文件放在本地，也支持放在远程Git仓库中。

放在本地：

```properties
spring.profiles.active=native
```

放在Git：

```properties
spring.cloud.config.server.git.uri = https://github.com/xx/xx/                    # 只定位到仓库
spring.cloud.config.server.git.search-paths = /springcloud-config/config-repo     # 配置文件在仓库的路径
spring.cloud.config.server.git.username = 
spring.cloud.config.server.git.password = 
```

配置中心的主类加上以下注解：

```java
@EnableConfigServer
```

客户端的`bootstrap.properties`文件：

```properties
spring.cloud.config.name=configtest
spring.cloud.config.profile=pro
# 获取文件的分支
spring.cloud.config.label=master
# 开启配置发现
spring.cloud.config.discovery.enabled=true
# 配置中心的服务名
spring.cloud.config.discovery.serviceId=springcloud-config-server
# 注册中心
eureka.client.serviceUrl.defaultZone=http://localhost:8005/eureka/
```

#### Spring Cloud Gateway

Spring Cloud Gateway是Spring Cloud中的网关，目标是替代Zuul，Spring Cloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

