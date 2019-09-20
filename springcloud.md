[TOC]

# SpringCloud特性
## Distributed/versioned configuration 分布式配置

## Service registration and discovery 服务注册与发现

## Routing 路由

## Servie-to-Service calls 服务调用
服务之前调用分为几种方式：

a. 使用 restTemplate进行调用，在集群时，使用@LoadBalance进行负载。

b.使用@FeinClient + Nginx进行集群负载。---  目前项目中使用此种方式进行开发。

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



# 其它

同事推荐 tensorflow中文网

周志华的书，Yoshua Bengio新书《Deep Learning》深度学习(中文）