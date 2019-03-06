---
layout: post
title:  "SpringCloud应用系列(3.1) ribbon服务调用"
categories: springCloud
tags:  springCloud ribbon
author: roboslyq
---

* content
{:toc}
# 1.文章列表

1. [SpringCloud应用系列(1)config配置中心](https://roboslyq.github.io/2019/03/04/springcloud-config-server/)
2. [SpringCloud应用系列(2)eureka服务治理](https://roboslyq.github.io/2019/03/10/springcloud-eurake/)

# 2. 项目依赖

* 注册中心：spring-cloud-eureka
* 服务提供者：spring-cloud-eureka-client
* 服务消费者: spring-cloud-ribbon-client

# 3.服务端环境搭建

## 3.1 依赖导入

```xml
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
package com.springcloudribbonclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableDiscoveryClient
public class SpringCloudRibbonClientApplication {
	@Bean //定义REST客户端，RestTemplate实例
	@LoadBalanced//开启负债均衡的能力
	RestTemplate restTemplate() {
		return new RestTemplate();
	}
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudRibbonClientApplication.class, args);
	}
}
```

## 3.3 创建控制器类

```java
package com.springcloudribbonclient.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class ConsumerController {
    @Autowired
    private RestTemplate restTemplate;
    @GetMapping(value = "/hello")
    public String hello() {
        return restTemplate.getForEntity("http://SPRING-CLOUD-EUREKA-CLIENT/", String.class).getBody();
    }
}
```

> 1、控制器类请求路径为`/hello`
>
> 2、控制器类通过`RestTemplate`指定服务提供者的路径`http://SPRING-CLOUD-EUREKA-CLIENT/`。其中
>
> `SPRING-CLOUD-EUREKA-CLIENT`为服务提供者的应用名称。即`spring.application.name`属性对应的名称。

## 3.4 application.properties配置

```properties
#应用名称
spring.application.name=spring-cloud-ribbon-client
#端口号
server.port=8084
#注册中心的地址
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

## 3.4 启动测试

### 3.4.1 服务注册

访问测试路径：`http://localhost:8082/`

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-ribbon/eureka-registry-ribbon.jpg)

### 3.4.2 服务调用

访问测试路径：`http://localhost:8084/`

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-ribbon/eureka-invoked-ribbon.png)

# 4. 负载均衡实现

> 在前端的基础上，修改服务服务提供者的端口，启动多个提供者。然后再进行消费。

## 4.1 修改服务提供者代码

```java
package com.roboslyq.springcloudeurekaclient;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@SpringBootApplication
//启用服务发瑞客户端
@EnableDiscoveryClient
@RestController
public class SpringCloudEurekaClientApplication {
	@Value("${server.port}")
	private int serverPort;
	//测试请求使用
	@RequestMapping(value ="/hello",method = RequestMethod.GET)
	public String home() {
		return  "hello:" + serverPort;
	}
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudEurekaClientApplication.class, args);
	}

}
```

> 注入当前启动服务的端口，并且返回给调用者。调用者可以通过端口区分具体调用了哪个服务提供者。

##　4.2 调整服务提供启动端口

可以通过mvn工具打成jar包，然后指定启动端口。或者直接在IDEA，修改启动端口，如下图所示：

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-ribbon/change-server-port.png)

## 4.3 调整消费消费代码

```java
package com.springcloudribbonclient.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class ConsumerController {
    @Autowired
    private RestTemplate restTemplate;

    @GetMapping(value = "/hello")
    public String hello() {
        String result = restTemplate.getForEntity("http://SPRING-CLOUD-EUREKA-CLIENT/hello", String.class).getBody();
        System.out.println(result);
        return  result;
    }
}
```

> 添加打印代码，打印服务提供者返回信息

## 4.4测试结果

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-ribbon/ribbon-loadbalance.png)

由结果可见，对服务提供者实现比较均衡的轮询调用。