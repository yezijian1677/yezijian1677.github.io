+++
author = "augenye"
title = "服务归因[ACE]"
date = "2022-10-05"
categories = [
"Servers"
]
tags = [
"组件"
]
+++

参考天猫服务归因，此框架的大部分概念来自于下面的文章 [服务归因组件](https://developer.aliyun.com/article/777742)，自行实现框架的思路也与之一致。<br />用下文的话来说，此框架主要是对以下内容的思考：<br />对“属性-分类-执行”问题的产品化思考与实践上的实践。<br />解决了属性分类的通用性问题，具有良好的扩展性和代码语义

## 组件
使用ACE需要了解的组件，通过这些组件搭配使用，才能使用好这个框架，解决复杂业务逻辑问题。
### Executor（执行器）
执行器，顾名思义，是在业务语义匹配后执行的一组逻辑。<br />执行器被归因器所绑定，可以被多个归因器绑定， 能够进行复用，但是这个复用的粒度需要开发者自行考量<br />eg: 定义一个发送微信消息的执行器，它将会发送微信消息，定义有三个要素

- 继承 `IExecutor`接口
- 注解定义执行器信息，必填的为 `name`，后续归因器绑定的执行器，也是基于此名称
- 实现`execute`方法，执行这个执行器专属的业务功能

![执行器](http://img.augenye.cn/images/20221006/7e7b56184cc34805a32477320bc57632.png?imageView2/0/interlace/1/q/100|imageslim)
<a name="Zp8mV"></a>
### Attributor（属性校验器）
属性校验器，**针对一个属性进行**校验，针对一个属性可以用不同的处理规则。<br />可以通过内置的注解`RulerMethod`定义一个此属性的校验方法，这是一个模板方法<br />需要开发者去定义好，此方法接受两个参数 `AceContext 和 fieldName`

- AceContext：归因上下文，整个归因需要用到的信息都在这个对象里
- fieldName：此校验器校验的字段名称

eg: 定一个平台属性的校验器，当平台有很多，如微信，抖音，微博等等，现在需要对平台这个属性进行校验，这时候由于我们是需要判断平台的类型，所以我们定一个判断类型是否匹配的校验方法<br />定义这一类校验器有以下步骤

- 实现 `IAttributor`接口
- 注解定义属性校验器的信息，这个是归因器绑定的属性校验器
- 定义这个属性自己的校验方法，可以有多个，**但是入参固定**

![属性校验器](http://img.augenye.cn/images/20221006/8b9ef4e606684c67a3f705a1ae6ca462.png?imageView2/0/interlace/1/q/100|imageslim)
<a name="f3EX2"></a>
### Classifier（归因器）
归因器，针对一组属性规则进行校验，满足条件则会执行绑定的执行器。<br />所以定义一个归因器，它有以下的要素

- 实现 `IClassifier`接口，无需实现内置方法，已有默认执行逻辑，除非你需要自定义自己的归因流程
- 注解定义归因器的信息
    - 归因器名称：用于场景绑定
    - `matcher`：任意Matcher绑定的**属性校验无效（false），则该归因无效**
    - `filter`：任意Matcher绑定的**属性校验有效（true），则该归因无效**
    - `executor`：场景绑定的执行器，执行一组业务逻辑

eg: 发送微信消息的归因器，当 `platform`为`wechat`的时候，执行发送微信消息<br />![归因器1](http://img.augenye.cn/images/20221006/92057fa2a8cc43709e08b35e5acbe5b9.png?imageView2/0/interlace/1/q/100|imageslim)<br />eg: 抖音平台的发送消息逻辑也大同消息，唯一的不同是属性校验方法的值不同和关联的执行器不同<br />![归因器2](http://img.augenye.cn/images/20221006/d6a79c50aad34a4f8944c8b6b58b7b85.png?imageView2/0/interlace/1/q/100|imageslim)
<a name="gt0Lu"></a>
### Scene（归因场景）
归因场景，绑定一组归因器，场景主要是聚合一组业务逻辑，代表一组归因器属于一个场景，场景有两种执行模式

- SINGLE: **单次匹配**，任意一个归因器执行成功，此场景成功，执行对应的场景执行器，停止后续场景匹配，**仅所有归因无效，该场景无效**
- MULTI：**多次匹配**，任意一个归因器执行成功，此场景成功，执行对应的归因器绑定的执行器，继续对后续场景进行匹配，若有匹配中，则也执行对应的执行器，**仅所有归因无效，该场景无效**

定义一个场景的逻辑很简单，它和归因器一样，无需实现对应的方法，场景已有内置逻辑，仅在需要扩展自己的场景逻辑的时候，重写该方法。虾<br />定义场景步骤如下：

- 实现 `IScene`接口
- 注解定义场景信息
    - `classifier`：归因器列表，用上述定义的长巾
    - `schema`：执行模式，如上所描述，SINGLE为默认模式

eg：发送消息，根据平台类型归因，匹配后执行对应的发送消息逻辑<br />![归因场景](http://img.augenye.cn/images/20221006/a8eb0d5ad48d4b3e89ae535506fed747.png?imageView2/0/interlace/1/q/100|imageslim)
<a name="UpLMr"></a>
### AceWorker（场景归因执行）
上面都定义好了，该如何使用呢，这就需要使用的AceWorker#process，具体使用用例如下，他就会针对上述进行归因，匹配后执行<br />
![aceWorker](http://img.augenye.cn/images/20221006/bf77b56170724deda057b0106761a5bf.png?imageView2/0/interlace/1/q/100|imageslim)
<br />
![aceworker1](http://img.augenye.cn/images/20221006/62ccdf67efe749b2801b5cde044b7751.png?imageView2/0/interlace/1/q/100|imageslim)
<br />修改 platform 的值为 douyin , 根据归因的规则，可以知道，将会执行抖音消息的执行器<br />![aceworker4](http://img.augenye.cn/images/20221006/43e82f42749a4395b8a29632790cdc13.png?imageView2/0/interlace/1/q/100|imageslim)
<a name="AmXsf"></a>
### 其他
还有其他内置的工具类，后续介绍
<a name="URPvU"></a>
## 使用
<a name="BZyOc"></a>
### 引入依赖
```xml
<dependency>
  <groupId>com.augenye.ace</groupId>
  <artifactId>universe-ace</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```
<a name="UHzNy"></a>

### 开启ace配置
集成了自动状态，springboot项目直接使用参数装配开启ACE
```properties
ace.switch=true
```

