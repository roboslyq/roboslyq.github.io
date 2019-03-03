---
layout: post
title:  "SpringCloud应用系列(一)config配置中心"
categories: springCloud
tags:  springCloud config
author: roboslyq
---

* content
{:toc}

# 前言
本系列文章主要是以实超为主，后续会有单独的系列来分析具体的实现原理。

# 环境搭建
## [SPRING INITIALIZR](https://start.spring.io/)
如果是从0开始搭建springBoot(springCloud基于springboot搭建)相关项目，强烈推荐[SPRING INITIALIZR](https://start.spring.io/)。打开网站，如下图所示：

![springinitializer](https://roboslyq.github.io/images/spring-cloud/spring-cloud-config/springinitializer.jpg)

> Generate a XXX with YYY and Spring Boot ZZZ
其中XXX指使用Gradle还是Maven进行项目管理。YYY指创建的项目语言是Java，Kotlin或Groovy。ZZZ是指具体的Spring boot版本。
Spring boot版本一旦确定之后，其它相关依赖也相关确定了。

我们选择录入相关信息如下：
Group = com.roboslyq
Artifact = spring-cloud-config-server
Dependencies = Web Actuator Config Server

最后点击"Generate Project"即可生成相应的项目。


![springinitializer1](https://roboslyq.github.io/images/spring-cloud/spring-cloud-config/springinitializer1.jpg)

## 导入IDEA

解压上面生成的压缩包，目录结构如下：
![springinitializer2](https://roboslyq.github.io/images/spring-cloud/spring-cloud-config/springinitializer2.jpg)

将其导入IDEA(在IDEA中直接打开pom.xml文件即可)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<!-- 标准的Spring boot项目 -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.roboslyq</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-cloud-config-server</name>
	<description>Demo project for Spring Boot</description>
	<!-- 指定Java编译版本及SpringCloud版本 -->
	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
	</properties>
	<dependencies>
		<!-- actuator依赖导入  -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- spring mvc导入-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- 配置中心服务器依赖-->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>
</project>
```
## 添加相关配置
### 本地git配置
#### appplication.properties
修改`application.properties`,添加如下配置：
```
#应用名称
spring.appliacion.name = spring-cloud-config-server
#服务端口
server.port = 8080
#actuator配置
management.endpoints.enabled-by-default = false
management.endpoint.env.enabled=true
management.endpoint.health.enabled=true
management.endpoint.info.enabled=true
management.endpoints.web.exposure.include = env

#spring cloud config配置
spring.cloud.config.server.git.uri = ${user.dir}/src/main/resources/configs

```
> git.uri当然可以写绝对路径，但为了保证通用性建议使用${user.dir}。在IDEA环境下，user.dir为当前项目所在路径。

#### 新建配置文件

![springinitializer3](https://roboslyq.github.io/images/spring-cloud/spring-cloud-config/configs.jpg)

分别在每个配置文件中添加配置项
```
#default
roboslyq.user.name = roboslyq
#dev
roboslyq.user.name = roboslyq.dev
#prod
roboslyq.user.name = roboslyq.prod
```
#### 将上述文件添加到git控制
```
robos@ROBOSLYQ MINGW64 /d/IdeaProjects_community/spring-cloud-config-server/src/main/resources/configs
$ git init
Initialized empty Git repository in D:/IdeaProjects_community/spring-cloud-config-server/src/main/resources/configs/.git/

robos@ROBOSLYQ MINGW64 /d/IdeaProjects_community/spring-cloud-config-server/src/main/resources/configs (master)
$ git add .
warning: LF will be replaced by CRLF in roboslyq.properties.
The file will have its original line endings in your working directory

robos@ROBOSLYQ MINGW64 /d/IdeaProjects_community/spring-cloud-config-server/src/main/resources/configs (master)
$ git commit -m "spring cloud config demo"
[master (root-commit) e195202] spring cloud config demo
 3 files changed, 4 insertions(+)
 create mode 100644 roboslyq-dev.properties
 create mode 100644 roboslyq-prod.properties
 create mode 100644 roboslyq.properties

robos@ROBOSLYQ MINGW64 /d/IdeaProjects_community/spring-cloud-config-server/src/main/resources/configs (master)

```
#### 启动类启用配置中心
```
package com.roboslyq.springcloudconfigserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class SpringCloudConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudConfigServerApplication.class, args);
	}

}
```

#参考资料
简书.林湾村龙猫 [https://www.jianshu.com/](https://www.jianshu.com/p/edce8e8c139e)
