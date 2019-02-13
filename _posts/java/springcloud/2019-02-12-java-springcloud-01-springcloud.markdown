---
layout: post
title:  "Spring Cloud（一）初步了解"
date:   2019-02-12
categories: springcloud
tag: springcloud
---

* content
{:toc}

# Spring Cloud 是什么 #

&nbsp;&nbsp;&nbsp;&nbsp;Spring Cloud是一系列框架的有序集合。他利用SpringBoot的开发便利性巧妙的简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都是可以利用SpringBoot的开发风格做到一键启动与部署。Spring并没有重复造轮子，而是将目前各家公司开发的比较成熟的、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行再封装，屏蔽掉复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和维护的分布式系统开发工具包。

&nbsp;&nbsp;&nbsp;&nbsp;微服务是可以独立部署、水平扩展、独立访问（或者有独立的数据库）的服务单元，SpringCloud就是这些微服务的大管家，采用了微服务这种架构以后，项目的数量会非常多，SpingCloud作为大管家需要管理好这些微服务，自然需要很多小弟来帮忙

主要的小弟有：Spring Cloud Config、Spring Cloud Netflix(Eureka、Hystrix、Zuul......)、Spring Cloud Bus、Spring Cloud Security.......每个模块都有独挡一面的才能

# Spring Cloud 核心成员 #

## Spring Cloud Netflix ##

&nbsp;&nbsp;&nbsp;&nbsp;SpringCloud的核心成员之一，它主要的成员有Eureka、Hystrix、Zuul、Archaius.......

- **Netflix Eureka**

>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**服务中心**：云端服务发现，一个基于REST的服务，用于定位服务，以实现云端中间服务层发现和故障转移。这个是SpringCloud最核心的模块之一，服务中心，任何服务需要其他服务的支持都需要从这里来拿，同样你有什么特殊的功能都需要要这里来报道，方便以后其他服务模块来调用；它的好处是你不需要直接去找各种服务的支持，只需要到服务中心来领取，也不需要知道提供服务支持的服务在哪里，还是几个服务来支持的，反正你需要啥来拿就行了，服务中心来保证稳定性和质量。 

- **Netflix Hystrix**

>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**熔断器**：容错管理工具，旨在通过熔断机制控制服务与第三方节点，从而对延迟和故障提供更强大的容错能力。比如某个服务挂掉了，但是你需要它的支持，然后调用它之后，半天没有响应，你却不知道，一直等待这个响应；有可能别的服务也在调用你的服务功能，那么当请求多了之后，就会验证阻塞整体服务的性能。这个时候Hystrix就派上用场了，当hystrix发现某个服务状态不稳定时，就会立马让他下线，让其它服务来顶上来，或者直接给你回复说不用等了，这个服务今天肯定是好不了了，该干嘛去干嘛去别再这里排队了。

- **Netflix Zuul**

>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**微服务网关**：Zuul是在云平台是提供动态路由，监控，弹性，安全等边缘服务的框架。Zuul相当于是设备和Netflix流应用的Web网站后端所有请求的前门。当其他服务来找总服务来办事的时候就一定要先经过Zuul,看一下该请求是否满足要求，用于将服务拦截后者是帮忙查找对应的调用服务。

- **Netflix Archaius**

>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**配置管理API**：包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。可以实现动态后去配置，原理是每隔60S（默认，可以配置）从配置资源读取一次内容，这样修改了配置文件后不需要重启服务就可以使修改后的内容生效，前提使用archaius的API来读取。

## Sping Cloud Config ##

&nbsp;&nbsp;&nbsp;&nbsp;**配置中心**：配置管理工具包，让你可以把配置放到远程服务器，集中化管理集群配置，目前支持本地存储、Git以及Subversion。就是以后大家的配置信息集中放置到一起，方便统一管理与升级。

## Sping Cloud Bus ##

&nbsp;&nbsp;&nbsp;&nbsp;**事件、消息总线**：用于在集群（例如：配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现部署。确保这个服务中消息保持畅通。

# Spring Cloud 其它成员 #

## Spring Cloud Starters ##

&nbsp;&nbsp;&nbsp;&nbsp;Spring Boot式的启动项目，为Spring Cloud提供开箱即用的依赖管理。

## Spring Cloud Security ##

&nbsp;&nbsp;&nbsp;&nbsp;基于Spring Security的安全工具包，为你的应用程序添加安全机制。这个模块负责所有服务的安全问题，设置不同的模块访问特定的资源，不能把所有的模块信息泄露。

# Spring Cloud 和 Spring Boot的关系 #

Spring Boot是Spring的一套快速配置的脚手架，可以基于Spring Boot快速开发单个微服务，Spring Cloud是一个基于Spring Boot实现的云应用开发工具；Spring Boot专注于快速、方便集成的单个个体，Spring Cloud是关注全局的服务治理框架；Spring Boot使用了默认大于配置的管理理念，很多集成方案已经帮你选择好了，能不配置就不配置，Spring Cloud很大的一部分是基于Spring Boot来实现，可以不基于Spring Boot吗？不可以。

Spring Boot可以离开Spring Cloud独立使用开发项目，但是Spring Cloud离不开Spring Boot，属于依赖关系。

>    Spring ——> Spring Boot ——>Spring Cloud

# Spring Cloud 的优势 #

&nbsp;&nbsp;&nbsp;&nbsp;微服务的框架那么多，比如：dubbo、Kubernetes,为什么就要使用过Spring Cloud呢？

- 产生于Spring大家族，Spring在企业级开发框架中无人能敌，来头很大，可以保证后续的更新、完善。比如dubbo已经不再提供给更新了
- 有Spring Boot这个独立干将可以省很多事，Spring Boot可以提供很多可用的模块供开发
- 作为一个微服务治理的大家伙，考虑很全面，几乎微服务治理的方方面面都考虑到了，方便开发开箱即用
- Spring Cloud活跃很高，教程很丰富，遇到问题很容易找到解决方案
- 轻轻松松几行代码就能完成熔断、负载均衡、服务中心的各种平台功能

&nbsp;&nbsp;&nbsp;&nbsp;Spring Cloud对于中小型互联网公司来说是一种福音，因为这类公司往往没有实力后者足够的资金投入去开发自己的分布式系统基础设施，使用Spring Cloud一站式解决方案能在从容应对业务发展的同时减少开发成本。同时，随着近几年微服务架构和Docker容器概念的火爆，也会让Spring Cloud在未来越来越“云”化的软件开发风格中立有一席之地，尤其是在目前五花八门的分布式解决方案中提供标准化的、全站式的技术方案，意义可能堪比当前Servlet规范的诞生，有效推进服务端软件系统技术水平的进步。

# 参考资料 #

[http://www.ityouknow.com/springcloud/2017/05/01/simple-springcloud.html](http://www.ityouknow.com/springcloud/2017/05/01/simple-springcloud.html)


