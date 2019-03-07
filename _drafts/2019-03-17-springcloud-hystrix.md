---
layout: post
title:  "SpringCloud应用系列(3.2) hystrix服务保护"
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

# 2. 服务端环境搭建

## 2.1 依赖导入

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

## 2.2 修改启动引导类

```java
package com.roboslyq.springcloudeureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
//启用eureka注册中心
@EnableEurekaServer
public class SpringCloudEurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudEurekaApplication.class, args);
	}

}
```

## 2.3 application.properties配置

```properties
#应用名称
spring.application.name = spring-cloud-eureka
#应用端口
server.port = 8082

#actuator配置
management.endpoint.enable-by-default = false
management.endpoint.env.enabled=true
management.endpoint.health.enabled=true
management.endpoint.info.enabled=true
management.endpoints.web.exposure.include = env

#eureka相关配置
eureka.instance.hostname = localhost
#registerWithEureka=true ,自已会注册
eureka.client.registerWithEureka = false
#如果fetch-registry = true , 则去Eureka Server拉取注册信息
eureka.client.fetchRegistry = false
eureka.client.serviceUrl.defaultZone = http://${eureka.instance.hostname}:${server.port}/eureka/
```

## 2.4 启动测试

访问测试路径：`http://localhost:8082/`

![eureka-console](https://roboslyq.github.io/images/spring-cloud/spring-cloud-eureka/eureka-console.jpg)

# 3 客户端环境搭建

## 3.1 依赖导入

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

## 3.2 修改启动引导类

```java
package com.roboslyq.springcloudeurekaclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
//启用服务发瑞客户端
@EnableDiscoveryClient
@RestController
public class SpringCloudEurekaClientApplication {
	//测试请求使用
	@RequestMapping("/hello")
	public String home() {
		return "Hello world";
	}
	public static void main(String[] args) {
		SpringApplication.run(SpringCloudEurekaClientApplication.class, args);
	}

}


```

## 3.3 修改配置文件

```properties
spring.application.name = spring-cloud-eureka-client
server.port = 8083
#配置注册中心地址
eureka.client.serviceUrl.defaultZone = http://localhost:8082/eureka/
```

## 3.4 启动测试

刷新服务端测试路径：`http://localhost:8082/`

![eureka-client](https://roboslyq.github.io/images/spring-cloud/spring-cloud-eureka/eureka-client.png)

此图表明服务已经注册成功。