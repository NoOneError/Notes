

# Nacos

## Nacos 单节点

1. 搜索docker镜像源

   ```shell
   docker search nacos
   ```

2. 拉去Nacos镜像

   ```shell
   docker pull nacos/nacos-server
   ```

3. 启动Nacos容器

   ```shell
   docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server
   ```
   
4. ```bash
   // 进入容器 
   docker exec -it nacos bash
   // 修改容器配置
   cd conf
   vi application.properties
   ```

## [nacos集群和配置持久化配置](https://lupengfei.blog.csdn.net/article/details/107414969)

- [Docker安装mysql5.7](https://www.cnblogs.com/daodaotest/p/13172272.html)
  
- [mysql备份](https://www.cnblogs.com/patrick-yeh/p/14211095.html)
  
- 初始化sql
  [sql脚本入口:](https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql) https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql

- 创建Nacos的工作目录：

  ```bash
  // 每个节点都创建
  mkdir -p /usr/local/docker/nacos-server
  mkdir -p /usr/local/docker/nacos-server/env
  mkdir -p /usr/local/docker/nacos-server/logs
  mkdir -p /usr/local/docker/nacos-server/init.d
  ```

- 调整custom.properties

  ```bash
  vim /usr/local/docker/nacos-server/init.d/custom.properties
  
  
  //添加一下配置
  management.endpoints.web.exposure.include=*
  ```

- 调整nacos-hostname.env

  ```bash
  vim /usr/local/docker/nacos-server/env/nacos-hostname.env
  
  #nacos dev env
  # 首选主机模式 hostname
  PREFER_HOST_MODE=hostname
  # 当前主机的IP
  NACOS_SERVER_IP=124.222.65.116
  # 集群的各个节点
  NACOS_SERVERS=47.100.220.41:8848 124.222.65.116:8848 81.69.17.120:8848
  # 数据库的配置
  MYSQL_SERVICE_HOST=47.100.220.41
  MYSQL_SERVICE_DB_NAME=nacos
  MYSQL_SERVICE_PORT=3306
  MYSQL_SERVICE_USER=root
  MYSQL_SERVICE_PASSWORD=123456
  
  # 从节点 这里就使用单节点测试,因此就不配置从节点
  #MYSQL_SLAVE_SERVICE_HOST=xxx 
  #MYSQL_SLAVE_SERVICE_PORT=3306
  
  # JVM参数 默认是2G 如果使用虚拟机,内存没有2G,就需要调整这里的参数,否则将无法启动
  JVM_XMS=256m
  JVM_XMX=256m
  JVM_XMN=256m
  
  ```

- 将配置文件拷贝到其他两台机器

  ```bash
  scp -r /usr/local/docker/nacos-server/env/nacos-hostname.env  root@192.168.1.161:/usr/local/docker/nacos-server/env/nacos-hostname.env
  scp -r /usr/local/docker/nacos-server/env/nacos-hostname.env  root@192.168.1.162:/usr/local/docker/nacos-server/env/nacos-hostname.env
  
  scp -r /usr/local/docker/nacos-server/init.d/custom.properties  root@192.168.1.161:/usr/local/docker/nacos-server/init.d/custom.properties
  scp -r /usr/local/docker/nacos-server/init.d/custom.properties  root@192.168.1.162:/usr/local/docker/nacos-server/init.d/custom.properties
  
  ```

- 分别启动三个节点

  ```bash
  docker run \
  -p 8848:8848 \
  -p 9848:9848 \
  -p 9849:9849 \
  --restart=always \
  --name nacos \
  --env-file=/usr/local/docker/nacos-server/env/nacos-hostname.env \
  -v /usr/local/docker/nacos-server/logs:/home/nacos/logs \
  -v /usr/local/docker/nacos-server/init.d/custom.properties:/home/nacos/init.d/custom.properties \
  -d nacos/nacos-server
  
  ```

## ==集群问题==

1. nacos默认端口8848，但是节点与节点之间的交流是 端口+1000/1001 。所以9848，9849端口docker也需要映射
2. 数据库必须是mysql5.7以上

## **放行端口**

首先确保防火墙是开着的

```
查看防火墙状态

systemctl status firewalld
```

```
开启防火墙

systemctl start firewalld
```

```shell
防火墙放行端口
1.添加端口
 6666 代表端口号
firewall-cmd --zone=public --add-port=6666/tcp --permanent
#说明:
#–zone #作用域
#–add-port=80/tcp #添加端口，格式为：端口/通讯协议
#–permanent 永久生效，没有此参数重启后失效

#多个端口:
firewall-cmd --zone=public --add-port=1-60000/tcp --permanent
2.刷新生效
firewall-cmd --reload
3.查看防火墙放行列表
firewall-cmd --list-all

#出问题就
systemctl restart docker

```

> 其次是一定要打开服务器安全组的限制

------------------------------------------------

## Springboot集成nacos注册中心

1. 依赖

   ```java
   <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    
    </dependency>
   ```
   
2. 配置

   ```yaml
   spring:
     cloud:
       nacos:
         discovery:
           server-addr: 47.100.220.41:28848  --nacos地址
     application:
       name: mall-order --注册在nacos的服务名
   ```

3. 开启注册中心

   ```java
   @EnableDiscoveryClient
   @SpringBootApplication //或者在@Configuration配置文件添加@EnableDiscoveryClient
   ```


------

## Springboot集成配置中心

1. 依赖

   ```xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
   ```

2. 创建bootstrap.properties(yml)

   ```yaml
   spring:
     cloud:
       nacos:
         config:
           # 配置文件后缀名为yaml
           file-extension: yaml
           service-addr: 47.100.220.41:28848
           namespace: prop    --命名空间  不写，默认是public空间
           group: 11       --指定命名空间下，分组。默认是DEFAULT_GROUP
           #其他配置文件
           ext-config:
             - data-id: other.yaml
               group: prop
               refresh: true #是否支持自动刷新
             - data-id: mybatis.yaml
               group: prop
               refresh: true #是否支持自动刷新
     application:
       name: product  --应用名     
   ```

3. 在nacos配置中心**对应命名空间——配置隔离**添加配置集mall-product.properties(yaml) ==默认规则== 应用名.yaml

4. 给mall-product.properties添加任何配置

   - @RefreshScope：动态更新获取并刷新配置
   - 如果配置中心和本地配置冲突，==优先选择配置中心==

==总结：==推荐使用命名空间来区分不同环境的配置，因为使用`profiles`或`group`会是不同环境的配置展示到一个页面，而Nacos控制台对不同的`Namespace`做了Tab栏分组展示

**！！！也可以不使用分组区分服务：** -dev区分

```yaml
${prefix}-${spring.profiles.active}.${file-extension}
```

在本地.yaml中配置

```yaml
spring:
  profiles:
  	active:dev
```

==效果====product-dev.yaml

## Nacos 底层原理



# Sentinel

## Sentinel安装

### Sentinel单节点

1. 搜索docker镜像源

   ```shell
   docker search sentinel
   ```

2. 拉去Nacos镜像

   ```shell
   docker pull bladex/sentinel-dashboard
   ```

3. 启动Nacos容器

   ```shell
   docker run --name sentinel -d -p 8858:8858 -d bladex/sentinel-dashboard
   ```

### **sentinel错误**

- [无法监控](https://www.cnblogs.com/brightfang/p/14470364.html)

## Sentinel控制台配置流控规则

**1，资源名**

唯一名称，默认为请求路径。

**2，针对来源**

Sentinel可以针对调用者进行限流，默认default(不区分来源)

**3，阀值类型/单机阀值：**

- **QPS（每秒钟的请求数量）**：当调用该api的QPS达到阀值的时候，进行限流。
- **线程数**：当调用api的线程数达到阀值的时候，进行限流。

**4，是否集群：**

默认不需要集群。

**5，流控模式**

- **直接**：当QPS超过阀值就进行限流。

- **关联**

  ：当关联的资源达到阀值，就限流自己。

  - 适用场景：查询和修改同一表的数据，如果是高并发的应用，查询接口的流量过大，就会影响修改接口的性能，反之同理，这就可以根据业务需求，去衡量希望优先读还是优先写。
  - 关联其实是一种保护关联资源的设计。

- **链路**

  ：只记录指定链路上的流量，即指定资源从入口资源进来的流量如果达到阀值就限流。

  - 链路其实是一种细粒度的针对来源，而编辑流控制中的针对来源输入框是微服务级别的，可以指定微服务过来流量达到阀值就限流。
  - 而链路是api级别的，指定的是api的调用流量达到阀值就限流。

**6，流控效果**

- **快速失败**：直接失败，抛出异常。相关源码：com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController
- **Warm Up(预热)**：根据coldFactor(冷加载因子，默认值为3)，从阀值/coldFactor，经过预热时长，才达到设置的QPS阀值。即如果阀值为100，冷加载因子为3，预热时长为10秒，那么就会用100/3作为最初的阀值，经过10秒之后才会将阀值达到100，进而进行限流，意思就是让允许通过的流量缓慢增加，在达到一定的时间之后才达到阀值这样会更好的保护微服务。
- 排队等待：匀速排队，让请求以均匀的速度通过，阀值类型必须设置成QPS，否则无效。此种模式可适用于应对突发流量的场景。相关源码：com.alibaba.csp.sentinel.slots.block.flow.controller.RateLimiterController


# openFeign

## ~~RestTepmlate~~

- 配置restTemplate

```java
@Bean
@LoadBalanced  //ribbon负载均衡  
public RestTepmlate restTemplate(){
    return new RestTemplate();
}
```

- 远程调用

```java
@AutoWired
private RestTemplate restTemplate;

@GetMapping("/xxxx/{id}")
public String paymentInfo(@PathVariable()){
    return restTemplate.getForObject(url+"/xxx/xxx"+id,String.class)
}
```



## 1.springBoot集成feign

1. 依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-loadbalancer</artifactId>
           <version>2.2.5.RELEASE</version>
    </dependency>
   ```

2. 开启远程调用

   ```java
   @EnableFeignClients
   @SpringBootApplication //或者在@Configuration配置文件添加@EnableFeignClients
   ```

3. 编写接口

   ```java
   @FeignClient("mall-product") //mall-product 为注册中心服务名
   public interface OrderFeignService {
       @GetMapping("/mallproduct/skuimages/test")  //目标服务中的接口
       public String getList();
   }
   ```

   

## 2.超时控制

openFeign底层使用Ribbon控制，默认超时时间1s。修改ribbon超时即可

```yaml
ribbon:
  ReadTimeout: 5000  #建立连接所用时间
  ConnectTimeout: 5000 #从服务器读取到可用资源所用时间
```

## 3.日志增强

对Feign接口的调用情况进行监控和输出

1. 配置bean

   ```java
   @Bean
   Logger.Level feignLoggerLevel(){
       return Logger.Level.FULL;
       /**
       * NONE 默认不显示任何日志
       * BASIC 仅记录请求方法、URL、响应状态码及执行时间
       * HEADERS 除BASIC信息，还有请求和响应头
       * FULL 除EADERSHEADERS信息，还有请求和相应的正文及元数据
       /
   }
   ```

2. 在yaml文件开启日志

   ```yaml
   logging:
     level:
       com.datayes.web.mom.mapper.**: debug
   ```

   



# gateWay

## 网关简介

在微服务架构中，一个系统会被拆分为很多个微服务。那么作为客户端要如何去调用这么多的微服务呢？如果没有网关的存在，我们只能在客户端记录每个微服务的地址，然后分别去调用。这样的话会产生很多问题，例如：

- 客户端多次请求不同的微服务，增加客户端代码或配置编写的复杂性
- 认证复杂，每个微服务都有独立认证
- 存在跨域请求，在一定场景下处理相对复杂

为解决上面的问题所以引入了网关的概念：所谓的API网关，就是指系统的统一入口，它封装了应用程序的内部结构，为客户端提供统一服务，一些与业务本身功能无关的公共逻辑可以在这里实现，诸如认证、鉴权、监控、路由转发等。

## Gateway简介

Spring Cloud Gateway是Spring公司基于Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。它的目标是替代Netflix Zuul，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控和限流。

- 优点：

1. 性能强劲：是第一代网关Zuul的1.6倍
2. 功能强大：内置了很多实用的功能，例如转发、监控、限流等
3. 设计优雅，容易扩展

- 缺点：

1. 其实现依赖Netty与WebFlux，不是传统的Servlet编程模型，学习成本高
2. 不能将其部署在Tomcat、Jetty等Servlet容器里，只能打成jar包执行
3. 需要Spring Boot 2.0及以上的版本，才支持

## Gateway核心架构

> 基本概念

路由(Route) 是 gateway 中最基本的组件之一，表示一个具体的路由信息载体。主要定义了下面的几个信息:

-  id：路由标识、区别于其他route
- uri：路由指向的目的地uri，即客户端请求最终被转发到的微服务
- order：用于多个route之间的排序，数值越小排序越靠前，匹配优先级越高
- predicate：断言的作用是进行条件判断，只有断言都返回真，才会真正的执行路由 
- filter：过滤器用于修改请求和响应信息

> 执行流程

1. Gateway Client向Gateway Server发送请求
2. 请求首先会被HttpWebHandlerAdapter进行提取组装成网关上下文
3. 然后网关的上下文会传递到DispatcherHandler，它负责将请求分发给RoutePredicateHandlerMapping
4. RoutePredicateHandlerMapping负责路由查找，并根据路由断言判断路由是否可用
5. 如果过断言成功，由FilteringWebHandler创建过滤器链并调用
6. 请求会一次经过PreFilter--微服务--PostFilter的方法，最终返回响应

## GateWay快速应用

- ### 创建一个gateway模块，导入相关依赖

```xml
<dependencies>
        <!--nacos客户端-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--gateway网关-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
        </dependency>
    </dependencies>
```

- ## 创建启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {

    public static void main(String[] args){
        SpringApplication.run(GatewayApplication.class, args);
    }
}

```

- 添加配置文件

```yaml
server:
  port: 7000
spring:
  application:
    name: api-gateway
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    gateway:
      routes:                       # 路由数组[路由 就是指定当请求满足什么条件的时候转到哪个微服务]
       - id: product_route          # 当前路由的标识, 要求唯一
         uri: lb://service-product  # lb指的是从nacos中按照名称获取微服务,并遵循负载均衡策略
         predicates:                # 断言(就是路由转发要满足的条件)
          - Path=/shop-pro/**       # 当请求路径满足Path指定的规则时,才进行路由转发
         filters:                   # 过滤器,请求在传递过程中可以通过过滤器对其进行一定的修改
          - StripPrefix=1           # 转发之前去掉1层路径
       - id: order_route
         uri: lb://service-order
         predicates:
          - Path=/shop-order/**
         filters:
          - StripPrefix=1
```

## Predicate断言配置

- After    

  判断时间在`After`配置的时间之后规则才生效

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver] //ZonedDateTime.now()
```

- Before  

   判断在`Before`之前路由配置才生效

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

- Between    

  在时间之内规则生效

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

- Cookie
  带cookie，名字和正则表达式
  cookie中需要带有key并且符合value的正则表达式。不带cookie会404

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=key, value
```

- Header
  请求头文件中带key的名称时 X-Request-Id 值是正则表达式\d+（数字）
  其他信息自己添加

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

- Host
  主机是yumbo.top或者huashengshu.top的子域名的规则生效，如果不带这个信息，则会返回404
  头文件中设置 Host

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.yumbo.top,**.huashengshu.top
```

- Method
  请求方法要是配置中设置的GET或者POST路由才会生效

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

- Path
  请求路径要符合Path设置的规则路由才生效

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```

- Query
  请求参数要包含green参数路由才生效

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=green
```

- RemoteAddr
  如果请求的远程地址是192.168.1.10，则此路由匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:

   - id: remoteaddr_route
     uri: https://example.org
     predicates:
     - RemoteAddr=192.168.1.1/24
```

- Weight
  这条路径将把80%的流量转发到weighthigh.org，并将20%的流量转发到weighlow.org
  权重路由，按照权重分流量

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

## 自定义filter 全局过滤器

```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

@Component
public class CustomGlobalFilter implements GlobalFilter, Ordered {
 
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }
 
    @Override
    public int getOrder() {
        return -1;
    }
}
```

## SpringCloud Gateway 超时配置

connect-timeout：必须以毫秒为单位指定连接超时。
response-timeout：响应超时必须指定为java.time.Duration文件

Http超时(响应和连接)可以为所有路由配置，并为每个特定路由重写。

1、全局超时配置

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

2、单个路由超时配置

```yaml
  - id: per_route_timeouts
    uri: https://example.org
    predicates:
      - name: Path
        args:
          pattern: /delay/{timeout}
    metadata:
      response-timeout: 200
      connect-timeout: 200
```
------------------------------------------------
