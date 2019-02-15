---
layout: post
title:  "Spring Cloud（二）服务注册中心"
date:   2019-02-12
categories: springcloud
tag: eureka
---

* content
{:toc}

# 背景简介 #

## 服务中心 ##

服务中心又称注册中心，管理各种服务功能包括服务的注册、发现、熔断、负载均衡等。

所有的服务全部注册到服务中心，服务通过服务中心来获取你需要的服务，服务调用者不用关心你调用服务的IP地址，由几台服务器组成，每次直接去服务中心获取可以使用的服务即可。

由于各种服务注册到了服务中心，就有了去做很多高级功能条件。比如：几台服务提供相同服务来做负载均衡；监控服务器调用成功率来做熔断，移除服务列表中的故障点；监控服务调用时间来对不同的服务器设置不同的权重等等。

## Eureka ##
Spring Cloud封装了Netflix公司开发的Eureka模块来实现服务注册和发现。Eureka采用C-S的设计架构，Eureka Server作为服务注册功能的服务器，它是服务注册中心。而系统中的其他服务，使用Eureka的客户端连接到Eureka Server，并维持心跳连接。这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。Spring Cloud的一些模块（比如Zuul）就可以通过Eureka Server来发现系统中的其他服务，并执行相关的逻辑。

Eureka由两个组件组成：Eureka服务器和Eureka客户端。Eureka服务器用作服务注册服务器，Eureka客户端是一个java客户端，用来简化与服务器的交互、作为轮询负载均衡，并提供服务的故障切换支持。netflix在其生产环境中使用的是另外的客户端，它提供基于流量、资源利用率以及出错状态的加权负载均衡。

Eureka由三个角色组成：

1. Eureka Server
- 提供服务注册发现

2. Service Provider
- 服务提供方
- 将自身服务注册到Eureka,从而使服务消费方能够找到自己

3. Service Consumer
- 服务消费方
- 从Eureka获取注册服务列表，从而能够消费服务

----------

# 案例实践 #

项目插件版本：
> spring-boot.version：1.5.6.RELEASE
> 
> jdk:1.8
> 
> spring-cloud.version：Dalston.SR4

## 1：Eureka Server ##

1、pom依赖

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
            <version>1.5.6.RELEASE</version>
        </dependency>
    </dependencies>


2、配置文件application.yml

    server:
	  port: 10001
	eureka:
	  client:
	    register-with-eureka: false
	    fetch-registry: false
	    service-url:
	      defaultZone: http://localhost:10001/eureka/
	security:
	  basic:
	    enabled: true
	  user:
	    name: root
	    password: 666666


配置文件参数解释：
> - eureka.client.register-with-eureka:表示是否将自己注册到Eureka Server，默认是true;
> - eureka.client.fetch-registry:表示是否从Eureka Server获取注册信息，默认为true;
> - eureka.client.serviceUrl.defaultZone:设置与Eureka Service交互的地址，查询服务和注册服务都要依赖这个地址。默认是http://localhost:8761/eureka：多个地址可使用,分割。
> - security.basic.enabled：设置登录验证是否开启（true/false）
> - security.user.name：登录注册中心账号
> - security.user.password：登录注册中心密码

3、启动代码添加注解（@EnableEurekaServer）
    
    @SpringBootApplication
    @EnableEurekaServer
    public class DemoEurekaServerApplication {    
	    public static void main(String[] args) {
		    SpringApplication.run(DemoEurekaServerApplication.class, args);
		}
    }

4、启动项目，浏览器访问:`http://localhost:10001`,效果图如下

![image_png](/images/2019-02/2019-02-12-java-springcloud-02-eureka/TL20190213165351.png)


## 2：Eureka Provider ##

1、pom依赖

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

2、配置文件application.yml
    
	server:
	  port: 10101
	spring:
	  application:
	    name: demo-eureka-provider
	eureka:
	  client:
	    service-url:
	      defaultZone: http://root:666666@localhost:10001/eureka/


3、启动代码添加注解（@EnableDiscoveryClient）

    @SpringBootApplication
	@EnableDiscoveryClient
	public class DemoEurekaProviderApplication {
	    public static void main(String[] args) {
	        SpringApplication.run(DemoEurekaProviderApplication.class, args);
	    }	
	}

4、controller层服务提供（在controller层新建UserController）

	@RestController
	public class UserController {
	
	    @RequestMapping(value = "/login", method = RequestMethod.GET)
	    public String login() throws Exception{
	        return "用户已验证";
	    }
	}


5、启动项目，浏览器访问:`http://localhost:10001`,效果图如下

![image_png](/images/2019-02/2019-02-12-java-springcloud-02-eureka/TL20190213162227.png)

## 3：Eureka Consumer ##

1、pom依赖

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

2、配置文件application.yml

	server:
	  port: 10102
	spring:
	  application:
	    name: demo-eureka-consumer
	eureka:
	  client:
	    service-url:
	      defaultZone: http://root:666666@localhost:10001/eureka/

3、启动代码添加注解（@EnableDiscoveryClient与@EnableFeignClients）

	@SpringBootApplication
	@EnableDiscoveryClient
	@EnableFeignClients
	public class DemoEurekaConsumerApplication {
	    public static void main(String[] args) {
	        SpringApplication.run(DemoEurekaConsumerApplication.class, args);
	    }
	}

4、fegin调用实现

	@FeignClient(name = "demo-eureka-provider")
	public interface UserFeignClient {
	
	    @RequestMapping(value = "/login", method = RequestMethod.GET)
	    public String login();
	}

注意事项：
> - @FeignClient(name = "demo-eureka-provider")中的name是远程服务名称，及spring.application.name配置的名称
> - 此类中的方法和远程服务中contoller中的方法名和参数需保持一致。

6、启动项目，浏览器访问:`http://localhost:10001`,效果图如下

![image_png](/images/2019-02/2019-02-12-java-springcloud-02-eureka/TL20190213162146.png)

7、浏览器分别访问：`http://127.0.0.1:10101/login`、`http://127.0.0.1:10102/userlogin`

http://127.0.0.1:10101/login 返回结果为：用户已验证 

![image_png](/images/2019-02/2019-02-12-java-springcloud-02-eureka/TL20190213162000.png)

http://127.0.0.1:10102/userlogin返回结果为：userlogin:用户已验证 

![image_png](/images/2019-02/2019-02-12-java-springcloud-02-eureka/TL20190213162024.png)

**说明客户端已经成功通过feign调用了远程服务login,并且将结果返回到了浏览器。**

----------

# 示例代码 #

[github示例代码](https://github.com/yanglei1992/springcloudstudy/tree/master/workplace-idea/01-demo-eureka-producer-consumer)


----------

# 参考资料 #

[http://www.ityouknow.com/springcloud/2017/05/10/springcloud-eureka.html](http://www.ityouknow.com/springcloud/2017/05/10/springcloud-eureka.html)


