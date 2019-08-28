[TOC]
# springMVC
## 1. MVC大流程
数据服务：request -> HandlerMapping -> HandleMethod (controller) -> 依据前端Accept转为相关格式的数据。

页面服务：request ->HandlerMapping -> HandleMethod(return “index”) -> ViewResolver -> render -> HTML.

## 2. 拦截器
HandlerInterceptor执行流程：

 	 preHandler -> handlerMethod(Method#invock方法行真实类) -> postHandle -> afterHandler 

针对于implements HandlerIntercepter中增加顺序是：

```java
public class SpringMVCApplication extends WebMvcConfigurerAdapter {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new SecondIntercepter());
        registry.addInterceptor(new FirstIntercepter());
        registry.addInterceptor(new ThirdIntercepter());
    }
}
```

则执行顺序为：

```java
com.cc.intercepter.FirstIntercepter preHandle
com.cc.intercepter.SecondIntercepter preHandle
com.cc.intercepter.ThirdIntercepter preHandle
com.cc.intercepter.ThirdIntercepter postHandle
com.cc.intercepter.SecondIntercepter postHandle
com.cc.intercepter.FirstIntercepter postHandle
com.cc.intercepter.ThirdIntercepter afterCompletion
com.cc.intercepter.SecondIntercepter afterCompletion
com.cc.intercepter.FirstIntercepter afterCompletion
```

即  pre 1、2、3 ,post 321 after 321; 
若更改增加 顺序相应的执行顺序也会改变 ,具体代码参见git中practis项目中的springMVC module.



 

## 3. 指定请求的输入和输出
### 前端header相关说明
* @RequestMapping中的produces对应请求头中的"Accept”，代表客户端可以接收什么格式，服务端发送什么格式。
* @RequestMapping中的consumers对应请求头中的“Content-Type”，代码客户端发送什么格式，服务器接收什么类型的参数；

### 返回XML还是JSON,也可以自己定义数据类型
#### 局部配置请求及返回
使用@PostMapping(value="", consumes=MediaType.APPLICATION_JSON_UTF8_VALUE,produces=MediaType.APPLICATION_XM_)
#### 全局配置请求及返回
参见practis项目中的spring-boot modle 中的SampleController.
#### 调用方式
在前端header 增加 Accept 值 为application/xml或者为application/json。
#### 实现方法
在项目pom中增加 以下代码便可实现返回客户端是json格式还是xml格式。
```xml
 <!-- 加入本包，springboot将统一默认返回XML信息，来源springboot源码WebMvcConfigurationSupport类中的jackson2XmlPresent-->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
        </dependency>
```
#### 实现原理
追代源码的方式：
```
EnableWebMvc
	DelegatingWebMvcConfiguration
		WebMvcConfigurationSupport#addDefaultHttpMessageConverters
```
源码addDefaultHttpMessageConverters(List<HttpMessageConverter<?>> messageConverters)方法中，if (jackson2XmlPresent) {} if(jackson2Present){}判断用于哪种方法进行解析，如MappingJackson2XmlHttpMessageConverter，提供两种方法:
1. read*: 通过http请求内容转化为对应的bean；
2. write*:通过bean序列化为对应的响应内容；
其中canRead,canWrite用于判断输入是否可以正常解析。
#### 可扩展数据的输入输出 
extends AbstractHttpMessageConverter，具体可以到时有具体业务时再深入研究。

### 4. SpringMVC controller线程安全

# Spring事务

## 1. JDBC隔离级别

JDK源码Connection中定义了5个 JDBC事务级别，如下:

```java
TRANSACTION_NONE			//JDBC中不支持事务；
TRANSACTION_READ_UNCOMMITTED//可读末提交的，即允许脏读、幻读；
TRANSACTION_READ_COMMITTED	//读取已提交的，禁止脏读，允许幻读、不可重复读；
TRANSACTION_REPEATABLE_READ	//可重复读允许幻读，即禁止脏读，不可重复读；
TRANSACTION_SERIALIZABLE	//串读，即禁止脏读、幻读、不可重复读；
```

即事务从上往下，级别越高性能越差。

spring源码TransactionDefinition中定义了以下级别，如下：

```java
ISOLATION_DEFAULT			//spring默认jdbc级别,mysql/oracle默认REPEATABLE_READ级别；
ISOLATION_READ_UNCOMMITTED	//直接引用java jdbc中的事务
ISOLATION_READ_COMMITTED	//直接引用java jdbc中的事务
ISOLATION_REPEATABLE_READ	//直接引用java jdbc中的事务
ISOLATION_SERIALIZABLE		//直接引用java jdbc中的事务
```

## 2. Spring事务传播级别

传播即 @Transactional 修饰的方法A中，调用 B，C。但B/C并无事务说明，即方法A中的@Transactional(?)，即可往下传播。

使用方法，@Transactional(propagation=Propagation.NESTED)等等；

```java
PROPAGATION_REQUIRED	//支持当前事务，如果没有，默认新建；
PROPAGATION_SUPPORTS	//支持当前事务，如果没有，以非事务方式执行；
PROPAGATION_MANDATORY	//支持当前事务，如果没有，抛出异常；
PROPAGATION_REQUIRES_NEW//新建事务，如果当前已有，把当前事务挂起；
PROPAGATION_NOT_SUPPORTED//非事务执行，如果当前已有，把当前事务挂起；
PROPAGATION_NEVER		//非事务执行，如果当前已有，抛出异常；
PROPAGATION_NESTED		//如果当前存在事务，则嵌套事务内执行；若无事务，执行REQUIRED方式；
```

spring默认PROPAGATION_REQUIRED。

## 3. 保护点

​	使用场景：当嵌套事务时，可以回滚到指定的保护点或者还原点位置。

# Spring Boot

官网定义：Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

We take an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss. Most Spring Boot applications need very little Spring configuration.

## 1. 特性

1. Create stand-alone Spring applications
   创建独立的Spring应用；
2. Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files) 
   嵌入web容器，不需要部署war文件；
3. Provide opinionated 'starter' dependencies to simplify your build configuration 
   提供固化的'starter'依赖，简化构建配置; 
4. Automatically configure Spring and 3rd party libraries whenever possible 
   自动装配spring或第三方类库；
5. Provide production-ready features such as metrics, health checks and externalized configuration
   提供运维特性，如指标信息、健康检查及外部化配置；
6. Absolutely no code generation and no requirement for XML configuration
   绝无代码生成，并且不需要XML配置；

《SpringBoot编程思想》小马哥总结5大特性：

1. 创建独立的应用----springApplication
2. 内嵌web 容器------tomcat//jetty/undertow应用服务器
3. 外部化配置---------starter等
4. 自动装载------------满足条件时的第三方jar自动依赖
5. 运维特性-----------使用springActuator。 



## 2. 运行方式

### jar

java -jar jarName或者使用java -jar META-INF/MANIFEST.MF/Main-Class,如下所示

```java
java -jar org.springframework.boot.loader.JarLauncher	
```

### war
```java
java -jar org.springframework.boot.loader.WarLauncher	
```
### maven
```java
mvn spring-boot:run
```

### 3.官网reference技巧

https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/htmlsingle/

文档默认有导航，注意后边地址修改为`/htmlsingle/` ，即是当前页中的全部文档，支持快速查找定位了。

