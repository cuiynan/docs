[TOC]



# SpringCloud特性

 微服务中的容错大致分为两种：

其一是重试，其二是断路器；



> Distributed/versioned configuration 分布式配置
> Service registration and discovery 服务注册与发现
> Routing 路由
> Servie-to-Service calls 服务调用
> 服务之前调用分为几种方式：
>
> a. 使用 restTemplate进行调用，在集群时，使用@LoadBalance进行负载。
>
> b.使用@FeinClient + Nginx进行集群负载。---  目前项目中使用此种方式进行开发。

## Load balancing 负载均衡

### IRule源码

```java
interface IRule
​	abstractLoadBalanceRule implements IRule
​			ClientConfigEnabledRoundRobinRule extends AbstractLoadBalancerRule
​			RoundRobinRule extends AbstractLoadBalancerRule
​			RandomRule extends AbstractLoadBalancerRule
​			RetryRule extends AbstractLoadBalancerRule
​			XXXRule extends AbstractLoadBalancerRule //可以按自己的方式进行自定义实现
```

Ribbon默认使用轮询，源码为  BaseLoadBalancer#Default_Rule是  RoundRobinRule()，更改随机参照源码等；

## Circuit Breakers 短路
融断

## Global locks 全局锁

## Leadership election and cluseter state 

## Distributed messaging 分布式消息



# 分布式基本组成

Provider

Consumer

Registry

Router

Broker(服务代理)

## SpringCloud ConfigServer

看似非常强大，但是由于现在部署常用K8S进行，故暂不使用此方式进行配置文件的配置。

部署时由于环境问题，直接使用环境变量的方式在部署机上进行配置文件相关的修改。

##  SpringCloud Bus

与springCloud ConfigServer 配合使用。配置文件的自动更新（git 中的webhook）

内网穿透工具： http://natapp.cn

##  SpringCloud Stream

消息驱动，与MQ并用。简化mq的操作流程。

@StreamListener(value = StreamClient.INPU) 传输对象。

##  SpringCloud Zull

服务网关，项目中通常情况下使用nginx作集群，使用zull做gateway做（前置）限流、鉴权、参数校验，（后置）统计、日志等。

项目中没有使用，使用后非常方便。

### 跨域

直接将跨域的类放入@Configuration中。



### 限流方式

令牌桶限流ZullFilter/RateLimiter谷歌，用的时候详细研究。

## SpringCloud Hystrix

熔断、服务降级、依赖隔离、监控（ hystrix-dashboard）

 降级可使用在自己类中 抛异常时和 调用 第三方服务出现异常时，进行服务的降级；

@HystrixCommand

Circuit Breaker断路器，其中相关的半开状态如配置以下参数：

> circuitBreaker.requestVolumeThreshold
>
> circuitBreaker.sleepWindowInMilliseconds
>
> circuitBreaker.errorThresholdPercentage

## SpringCloud Sleuth

### 链路跟踪

使用插件zipkin.io，springboot1.X，注意需要在pom中提高aspectjweaver相关依赖版本。SpringBoot2.X 没有此问题。具体参照 git/euraka-consumer-feign。

> 大体部署增加 spring-cloud-starter-zipkin
>
> 配置文件增加 zipkin.base-url指定相关服务器，配置sleuth.sampler.probabilty=1。
>
> 直接启动开始测试了，非常方便。



# 其它

同事推荐 tensorflow中文网

周志华的书，Yoshua Bengio新书《Deep Learning》深度学习(中文）