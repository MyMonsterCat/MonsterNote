

# 服务网关GateWay

### 1.1 简介

​	GateWay是在Spring生态系统之上构建的API网关服务，基于Spring5，SpringBoot2和ProjectReactor等技术。GateWay旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能，例如：熔断、限流、重试等

​	**gateway之所以性能好,因为底层使用WebFlux,而webFlux底层使用netty通信(NIO)**

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/%E5%BE%AE%E6%9C%8D%E5%8A%A1/gateway%E7%9A%843.png)



### 1.2 GateWay的特性

Spring Cloud Gateway有如下特性

- 基于Spring Framework 5, Project Reactor和Spring Boot 2.0进行构建

- 动态路由:能够匹配任何请求属性

- 可以对路由指定Predicate (断言)和Filter (过滤器) 

- 集成Hystrix的断路器功能

- 集成Spring Cloud服务发现功能

- 易于编写的Predicate (断言)和Filter (过滤器) 

- 请求限流功能

- 支持路径重写

  

### 1.3 GateWay与zuul的区别

在SpringCloud Finchley正式版之前，Spring Cloud推荐的网关是Netflix提供的Zuul

- Zuul1.x，是一个**基于阻塞I/O的API** Gateway
- Zuul 1.x**基于Servlet 2. 5使用阻塞架构**，它**不支持任何长连接**(如WebSocket) ，Zuul的设计模式和Nginx较像,每次IO操作都是从工作线程中选择一个执行, 请求线程被阻塞到工作线程完成，但是差别是Nginx用C++实现，Zuul 用Java实现，而JVM本身会有第一次加载较慢的情况， 使得Zuul的性能相对较差
- Zuul 2.x理念更先进，想基于Netty非阻塞和支持长连接,但SpringCloud目前还没有整合。Zuul 2.x的性能较Zuul 1.x有较大提升
  在性能方面，根据官方提供的基准测试，Spring Cloud Gateway的RPS (每秒请求数)是Zuul 的1.6倍。
- Spring Cloud Gateway建立在Spring Framework 5、Project Reactor和Spring Boot2之上，使用非阻塞API。
- Spring Cloud Gateway还支持WebSocket，并且与Spring紧密集 成拥有更好的开发体验



### 1.4 zuul 1.x的模型

SpringCloud中所集成的Zuul版本，采用的是**Tomcat容器**，使用的是传统的Servlet **IO**处理模型

> 既然是Tomcat容器，那我们来回顾下Servlet生命周期（Servlet由Servlet Container进行生命周期管理）
>
> Container启动时构造Servlet对象并调用Servlet init()进行初始化
>
> Container运行时接受请求，并为每个请求分配一个线程（一般从线程池中获取空闲线程）然后调用Service()
>
> Container关闭时调用destory()销毁Servlet

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/%E5%BE%AE%E6%9C%8D%E5%8A%A1/Servlet%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

我们不妨来仔细研究一下上图流程，会产生什么问题。

> 首先应当明确的是，Servlet是一个网络IO模型，当一个请求进入Container时，就会为其绑定一个线程，在并发不高的场景下这种模型是适用的。
>
> 但是一旦遇到高并发的情况，线程数量就会上涨，而使用线程资源的代价是昂贵的（上下文切换，内存消耗大），严重影响了请求的处理时间。
>
> 所以zuul1.x是基于Servlet之上的一个阻塞式处理模型，即spring实现了处理所有request请求的一个servlet(DispatcherServlet)，并由该servlet阻塞式处理。所以Zuul无法摆脱Servlet模型的弊端



### 1.5 什么是webflux

**是一个非阻塞的web框架,类似springmvc这样的**

传统的Web框架，比如说: struts2, springmvc等都是基 于Servlet API与Servlet容器基础之上运行的。

但是在Senvlet3.1之后有了异步非阻塞的支持。而WebFlux是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。

相对于传统的web框架来说，它可以运行在诸如Netty, Undertow及支持Servlet3.1的容器 上。

非阻塞式+函数式编程(Spring5必须让你使用java8)

Spring WebFlux是Spring 5.0引入的新的响应式框架，区别于Spring MVC,它不需要依赖Servlet API,它是完全异步非阻塞的，并且基于Reactor来实现响应式流规范。



### 1.6 GateWay的三大核心概念

#### 1.6.1 Route(路由)

​	路由是构建网关的基本模块，它由ID、目标URI、一系列的断言和过滤器组成，如果断言为true则匹配该路由（**就是根据某些规则,将请求发送到指定服务上**）



#### 1.6.2 Predicate（断言）

​	参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由（就是判断,如果符合条件就是xxxx,反之yyyy）



#### 1.6.3 Filter(过滤)

​	指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改（**路由前后,过滤请求**）



### 1.7 GateWay的工作原理

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/%E5%BE%AE%E6%9C%8D%E5%8A%A1/gateway%E7%9A%8412.png)

​	客户端向Spring Cloud Gateway发出请求。然后在Gateway Handler Mapping中找到与请求相匹配的路由,将其发送到Gateway
Web Handler.

​	Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

​	过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前( "pre”)或之后( "post" )执行业务逻辑。

​	Filter在"pre" 类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，在"post"类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。



### 1.8 使用GateWay的两种方式

#### 1.8.1 在配置文件yml中配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001   #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由

        - id: payment_routh2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由


eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

#### 1.8.2 代码中注入RouteLocator的Bean(硬编码方式)

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/%E5%BE%AE%E6%9C%8D%E5%8A%A1/gateway%E7%9A%8420.png)



### 1.9 动态路由

​	上面的配置虽然首先了网关,但是是在配置文件中写死了要路由的地址

​	现在需要修改,不指定地址,而是根据微服务名字进行路由,我们可以在注册中心获取某组微服务的地址（默认情况下Gateway会根据注册中心的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能）

#### 修改GateWay模块的配置文件

需要注意的是uri的协议为lb，表示启用Gateway的负载均衡功能

lb://serviceName是spring cloud gateway在微服务中自动为我们创建的负载均衡uri

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由

        - id: payment_routh2
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由
eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
 
```

### 1.10 常用的断言

注意上述的配置文件，我们之前其中配置了断言，**这个断言表示,如果外部访问路径是指定路径,就路由到指定微服务上**，可以看到，这里有一个Path，这个是断言的一种，此外还有很多种断言。

``After`` : 可以指定,只有在指定时间后,才可以路由到指定微服务

```yaml
 - After=2020-03-08T10:59:34.102+08:00[Asia/Shanghai]
```

这里表示，只有在**2020年的2月21的15点51分37秒**之后，访问才可以路由，在此之前的访问，都会报404

如何获取当前时区?

```java
 ZoneDateTime zbj = ZonedDateTime.now();
```

``before``：与after类似,他说在指定时间之前的才可以访问

``between``：需要指定两个时间,在他们之间的时间才可以访问

``cookie``:只有包含某些指定cookie(key，value)，的请求才可以路由·

``Header``:只有包含指定请求头的请求,才可以路由

```yaml
- Header=X-Request-Id, \d+   #请求头中要有X-Request-Id属性并且值为整数的正则表达式
```

``host``:只有指定主机的才可以访问，比如我们当前的网站的域名是www.aa.com，那么这里就可以设置,只有用户是www.aa.com的请求,才进行路由

``method``:只有指定请求才可以路由，比如get请求...

``path``:只有访问指定路径，才进行路由，比如访问/abc才路由

``Query``:必须带有请求参数才可以访问

### 1.11 Filter过滤器



#### 1.11.1 生命周期

**在请求进入路由之前,和处理请求完成,再次到达路由之前**



#### 1.11.2 种类

##### ① GateWayFilter，单一的过滤器

##### ② GlobalFilter，全局过滤器



#### 1.11.3 自定义过滤器

实现两个接口

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/%E5%BE%AE%E6%9C%8D%E5%8A%A1/gateway%E7%9A%8444.png)



