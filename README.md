# springcloud-analysis

> 基于spring-boot2.x和spring-cloud 重新设计原先基于vert.x的分布式统计系统

### 🌈 注册中心 

#### consul 

[consul download](http://www.consul.io/)

如果是开发环境，部署在windows,解压后配置环境变量，启动命令为``consul agent -dev``

#### eureka （server 暂不用） 

startup :

```
java -jar server-1.0.0-SNAPSHOT.jar --spring.profiles.active=eureka1
java -jar server-1.0.0-SNAPSHOT.jar --spring.profiles.active=eureka2
java -jar server-1.0.0-SNAPSHOT.jar --spring.profiles.active=eureka3
```

如果是本地测试，还需要设置下host:

```
127.0.0.1   eureka1
127.0.0.1   eureka2
127.0.0.1   eureka3
```

### 🍀 配置中心 （config） 

配置中心采用的是git管理内容，是自建的gogs。如何搭建戳 [Devops](http://7le.top/2017/10/09/%E7%8E%A9%E8%80%8DDevops%20Git+Gogs+Jenkins+Docker)

如果是为了测试图个方便，也可以使用也可以配置本地存储，只需要设置：

```
spring:
  profiles:
    active: native
  cloud:
     config:
       server:
         native:
           search-locations: file:E:/properties/ 
```
在修改之后需要自己触发``curl -X POST http://127.0.0.1:7000/application/bus-refresh``


### 🍁 监控 -- Actuator

spring-boot 2.x 需要自己开放端点，配置如下：
```
management:
  endpoints:
    web:
      exposure:
        include: '*'   # 代表全部放开，可以自行选择
      base-path: /application
```
Actuator 部分端点：

| HTTP 方法|     路径|   描述|
| :-------- | --------:| :------: |
|GET|/beans|描述应用程序上下文里全部的Bean，以及它们的关系|
|GET|/health|健康检查     |
|GET|/env|获取全部环境属性     |
|GET|/env/{toMatch}|根据名称获取特定的环境属性值     |
|GET|/configprops|描述配置属性(包含默认值)如何注入Bean     |
|GET|/mappings| 描述全部的URI路径，以及它们和控制器(包含Actuator端点)的映射关系    |
|GET|/metrics|报告各种应用程序度量信息，比如内存用量和HTTP请求计数   |
|GET|/metrics/{requiredMetricName}|报告指定名称的应用程序度量值     |
|POST|/bus-refresh|端点手动刷新配置     |
|GET|/httptrace|提供基本的HTTP请求跟踪信息(时间戳、HTTP头等)     |


### 🐧 网关 （gateway）

#### 目前使用 -- zuul

路由重试配置：

```
spring:
  cloud:
    loadbalancer:
      retry:
        enabled: true
zuul:
  retryable: true
ribbon:
  connectTimeout: 500                 # 请求连接的超时时间
  readTimeout:  1000                  # 请求处理的超时时间
  maxAutoRetries: 2                   # 对当前实例的重试次数
  maxAutoRetriesNextServer: 1         # 切换实例的重试次数
  okToRetryOnAllOperations: true      # 对所有操作请求都进行重试
```

实现了动态路由和集群通知  博客可以戳 [springcloud：实现zuul的动态路由和集群通知](http://7le.top/2018/04/18/springcloud%EF%BC%9A%E5%AE%9E%E7%8E%B0zuul%E7%9A%84%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1%E5%92%8C%E9%9B%86%E7%BE%A4%E9%80%9A%E7%9F%A5/)

### 🐳 链路跟踪 -- zipkin

#### linux 部署：

```
wget -O zipkin.jar 'https://search.maven.org/remote_content?g=io.zipkin.java&a=zipkin-server&v=LATEST&c=exec'
nohup java -jar zipkin.jar &  
```

web页面：http://your_ip:9411 ，默认9411 可以自行修改
