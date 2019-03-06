---
layout: post
title:  "SpringCloud应用系列(3.2) feign负载均衡"
categories: springCloud
tags:  springCloud feign
author: roboslyq
---

* content
{:toc}
# 1.文章列表

1. [SpringCloud应用系列(1)config配置中心](https://roboslyq.github.io/2019/03/04/springcloud-config-server/)
2. [SpringCloud应用系列(2)eureka服务治理](https://roboslyq.github.io/2019/03/10/springcloud-eurake/)
3. [SpringCloud应用系列(3.1) ribbon服务调用](https://roboslyq.github.io/2019/03/06/springcloud-ribbon/)

# 2. 项目依赖

* 注册中心：spring-cloud-eureka
* 服务提供者：spring-cloud-eureka-client
* 服务消费者: spring-cloud-feign-client

# 3.Feign消费端环境搭建

## 3.1 依赖导入

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

## 3.2 修改启动引导类

```java
package com.springcloudribbonclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
//@EnableDiscoveryClient
@EnableFeignClients
public class SpringCloudRibbonClientApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudRibbonClientApplication.class, args);
	}
}

```

## 3.3 创建具体消费类

```java
package com.springcloudribbonclient.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
//Feign注解，value为服务提供者的serviceId
@FeignClient(value = "SPRING-CLOUD-EUREKA-CLIENT")
public interface FeignServiceDemo {
    //配置服务提供者的URI及相关参数，此注解与SpringMVC通用
    @RequestMapping(value = "/hello", method = RequestMethod.GET, produces = "application/json; charset=UTF-8")
    String hello1();
}

```

> 此类不需要具体实现，具体实现由代理技术生成

## 3.4 创建控制器类

```java
package com.springcloudribbonclient.controller;

import com.springcloudribbonclient.service.FeignServiceDemo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class ConsumerController {
    //注入接口
    @Autowired
    private FeignServiceDemo feignServiceDemo;

    @GetMapping(value = "/hello1")
    public String hello1() {
        String result = feignServiceDemo.hello1();
        System.out.println("feigh:"+result);
        return  result;
    }
}
```

> 1、控制器类请求路径为`/hello1`。
>
> 2、控制器类通过`feignServiceDemo`来调用服务器提供的相应服务。

## 3.4 application.properties配置

```properties
#应用名称
spring.application.name = feign-consumer
#端口号
server.port=9000
#注册中心的地址
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/

```



# 4. 负载均衡实现

> 此服务提供提供者与`SpringCloud应用系列(3.1) ribbon服务调用`中的服务提供者保持一致。但此处使用打包的方式进行启动。

## 4.1 打包服务提供者代码

```shell
C:\Users\robos>D:

D:\>cd D:\IdeaProjects_community\spring-cloud-learn-git\spring-cloud-learn-resources\spring-cloud-learn\spring-cloud-eureka-client

D:\IdeaProjects_community\spring-cloud-learn-git\spring-cloud-learn-resources\spring-cloud-learn\spring-cloud-eureka-client>mvn clean -Dmaven.test.skip -U install package

```

打包结果：

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-feign/feign-package.png)

## 4.2 启动服务提供者

启动服务提供者，并且指定端口。

```shell
java -jar target/spring-cloud-eureka-client-0.0.1-SNAPSHOT.jar --server.port=9002
java -jar target/spring-cloud-eureka-client-0.0.1-SNAPSHOT.jar --server.port=9003
java -jar target/spring-cloud-eureka-client-0.0.1-SNAPSHOT.jar --server.port=9004
```

> 可以通过`--server.port`参数启动多个服务提供者同时启动

## 4.3 检查注册中心

| Application                    | AMIs        | Availability Zones | Status                                                       |
| ------------------------------ | ----------- | ------------------ | ------------------------------------------------------------ |
| **FEIGN-CONSUMER**             | **n/a** (1) | (1)                | **UP** (1) - [ROBOSLYQ:feign-consumer:9000](http://roboslyq:9000/actuator/info) |
| **SPRING-CLOUD-EUREKA-CLIENT** | **n/a** (3) | (3)                | **UP** (3) - [ROBOSLYQ:spring-cloud-eureka-client:9003](http://roboslyq:9003/actuator/info) , [ROBOSLYQ:spring-cloud-eureka-client:9001](http://roboslyq:9001/actuator/info) , [ROBOSLYQ:spring-cloud-eureka-client:9002](http://roboslyq:9002/actuator/info) |

>由此可知，一共启动了3个服务提供者和一个FEIGN消费者。

## 4.4 结果验证

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-feign/feign-result.png)

