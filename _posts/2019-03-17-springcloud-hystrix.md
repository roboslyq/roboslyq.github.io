---
layout: post
title:  "SpringCloud应用系列(4.1) hystrix与Ribbon"
categories: springCloud
tags:  springCloud hystrix
author: roboslyq
---


* content
{:toc}
# 1.文章列表

1. [SpringCloud应用系列(1)config配置中心](https://roboslyq.github.io/2019/03/04/springcloud-config-server/)
2. [SpringCloud应用系列(2)eureka服务治理](https://roboslyq.github.io/2019/03/10/springcloud-eurake/)
3. [SpringCloud应用系列(3.1) ribbon服务调用](https://roboslyq.github.io/2019/03/06/springcloud-ribbon/)
4. [SpringCloud应用系列(3.2) feign负载均衡](https://roboslyq.github.io/2019/03/10/springcloud-feign/)

# 2.相关项目

- 注册中心：spring-cloud-eureka
- 服务提供者：spring-cloud-eureka-client
- 服务消费者: spring-cloud-hystrix

# 3. 服务端环境搭建

此次环境搭建主要是hystrix + ribbon来实现。

## 3.1 依赖导入

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

## 3.2 修改启动引导类

```java
package com.roboslyq.springcloudhystrix;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
public class SpringCloudHystrixApplication {
    @Bean //定义REST客户端，RestTemplate实例
    @LoadBalanced//开启负债均衡的能力
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudHystrixApplication.class, args);
	}
}

```

> `@EnableHystrix` 派生于`@EnableCircuitBreaker`,因此上面注解`@EnableHystrix` 可以用注解 `@EnableCircuitBreaker`来替代，最终效果一样。
>
> ```java
> @Target(ElementType.TYPE)
> @Retention(RetentionPolicy.RUNTIME)
> @Documented
> @Inherited
> @EnableCircuitBreaker
> public @interface EnableHystrix {
> 
> }
> 
> ```

## 3.3 application.properties配置

```properties
#应用名称
spring.application.name=spring-cloud-hystrix
#端口号
server.port=8085
#注册中心的地址
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/

```

## 3.4 Hystrix包装Ribbon

```java
package com.roboslyq.springcloudhystrix.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class HystrixRibbonService {

    @Autowired
    private RestTemplate restTemplate;

    //指定服务失败回调方法
    @HystrixCommand(fallbackMethod = "helloError")
    public String hello() {
        String result = restTemplate.getForEntity("http://SPRING-CLOUD-EUREKA-CLIENT/hello", String.class).getBody();
        System.out.println(result);
        return  result;
    }
	//失败回调方法
    public String helloError(){
        return "hello world is error!!!";
    }
}

```



## 3.5 修改控制器类

```java
package com.roboslyq.springcloudhystrix.controller;

import com.roboslyq.springcloudhystrix.service.HystrixRibbonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HystrixRibbonController {
    @Autowired
    HystrixRibbonService hystrixRibbonService;

    @GetMapping("/hello")
    public String hello(){
        //调用服务
        return hystrixRibbonService.hello();
    }
}

```

# 4. 启动测试

## 4.1 检查注册中心

访问测试路径：`http://localhost:8082/`

Eureka注册中心如下;

| Application                    | AMIs        | Availability Zones | Status                                                       |
| ------------------------------ | ----------- | ------------------ | ------------------------------------------------------------ |
| **SPRING-CLOUD-EUREKA-CLIENT** | **n/a** (1) | (1)                | **UP** (1) - [JK-LUOYONGQIAN.yx.yuexiu.com:spring-cloud-eureka-client:8083](http://jk-luoyongqian.yx.yuexiu.com:8083/actuator/info) |
| **SPRING-CLOUD-HYSTRIX**       | **n/a** (1) | (1)                | **UP** (1) - [JK-LUOYONGQIAN.yx.yuexiu.com:spring-cloud-hystrix:8085](http://jk-luoyongqian.yx.yuexiu.com:8085/actuator/info) |

## 4.2 请求服务

###　4.2.1 正常情况

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-hystrix/hystrix-normal-console.jpg)

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-hystrix/hystrix-normal-broswer.jpg)

### 4.2.2 异常情况

停掉服务提供者`spring-cloud-eureka-client` 再请求路径`http://localhost:8085/hello`

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-hystrix/hystrix-breaker.jpg)

> 由结果可知，已经进入熔断方法。