---
layout: post
title: mysql安装错误解决办法
category: javaweb
tags: Essay
---


#1 基本概念

##1.1 spring

Spring是一个分层的JavaSE/EE full-stack轻量级开源框架。它是为了解决企业应用开发的复杂性而创建的。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。 简单来说，Spring是一个轻量级的控制反转（IoC）和面向切面（AOP）的容器框架。

##1.2 spring-mvc

Spring MVC属于SpringFrameWork的后续产品，已经融合在Spring Web Flow里面。Spring MVC 分离了控制器、模型对象、分派器以及处理程序对象，这种分离让它们更容易进行定制。

##1.3 mybatis

mybatis是一个轻量级的持久层框架， 包括SQL Maps和Data Access Objects（DAO）MyBatis 消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。MyBatis 使用简单的 XML或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Old Java Objects，普通的 Java对象）映射成数据库中的记录。

##1.4 maven

Maven项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。

#2 搭建框架

##2.1 引入相关jar包

使用maven可以导入并管理项目所需的jar包，我们的`<modelVersion>4.0.0</modelVersion>`

先设置一些需要用到的property：

```xml
<properties>
	<jdk>1.8</jdk>
	<!--指定Maven用什么编码来读取源码及文档 -->
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<!--指定Maven用什么编码来呈现站点的HTML文件 -->
	<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
	<!-- database -->
	<mysql>5.1.37</mysql>
	<druid>1.0.12</druid>
	<c3p0>0.9.5.1</c3p0>
	<!-- mybatis -->
	<mybatis>3.3.0</mybatis>
	<mybatis-spring>1.2.3</mybatis-spring>
	<mybatis.generator>1.3.2</mybatis.generator>
	<!-- apache -->
	<commons-codec>1.10</commons-codec>
	<httpcomponents>4.5</httpcomponents>
	<commons-lang3>3.2.1</commons-lang3>
	<commons-io>2.4</commons-io>
	<commons-compress>1.10</commons-compress>
	<!-- json -->
	<fastjson>1.2.7</fastjson>
	<!-- xml -->
	<dom4j>1.6.1</dom4j>
	<jaxen>1.1.6</jaxen>
	<!-- spring -->
	<spring>4.2.6.RELEASE</spring>
	<aspectjweaver>1.8.5</aspectjweaver>
	<!-- web -->
	<!-- file upload -->
	<commons-fileupload>1.3.1</commons-fileupload>
	<!-- template -->
	<freemarker>2.3.23</freemarker>
	<!-- log -->
	<slf4j>1.7.5</slf4j>
	<log4j>2.3</log4j>
	<disruptor>3.3.2</disruptor>
	<!-- mail -->
	<mail>1.4.7</mail>
	<commons-email>1.3.2</commons-email>
	<!-- jedis -->
	<jedis>2.0.0</jedis>
	<!-- test -->
	<junit>4.12</junit>
	<!--hibernate validator-->
	<hibernate-validator>5.2.1.Final</hibernate-validator>
</properties>
```

然后导入这些包，一些包的说明见注释：


```xml
<dependencies>
	<!-- mysql -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>${mysql}</version>
		<scope>runtime</scope>
	</dependency>
	<!-- spring -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-webmvc</artifactId>
		<version>${spring}</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context-support</artifactId>
		<version>${spring}</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-oxm</artifactId>
		<version>${spring}</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-tx</artifactId>
		<version>${spring}</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-jdbc</artifactId>
		<version>${spring}</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-aspects</artifactId>
		<version>${spring}</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-instrument-tomcat</artifactId>
		<version>${spring}</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-jms</artifactId>
		<version>${spring}</version>
	</dependency>
	<!--添加aspectjweaver包 -->
	<dependency>
		<groupId>org.aspectj</groupId>
		<artifactId>aspectjweaver</artifactId>
		<version>${aspectjweaver}</version>
	</dependency>
	<!-- 为了方便进行单元测试，添加spring-test包 -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-test</artifactId>
		<version>${spring}</version>
		<scope>test</scope>
	</dependency>
	<!-- apache -->
	<dependency>
		<groupId>org.apache.commons</groupId>
		<artifactId>commons-lang3</artifactId>
		<version>${commons-lang3}</version>
	</dependency>
	<dependency>
		<groupId>commons-io</groupId>
		<artifactId>commons-io</artifactId>
		<version>${commons-io}</version>
	</dependency>
	<dependency>
		<groupId>commons-codec</groupId>
		<artifactId>commons-codec</artifactId>
		<version>${commons-codec}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.commons</groupId>
		<artifactId>commons-compress</artifactId>
		<version>${commons-compress}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.httpcomponents</groupId>
		<artifactId>httpclient</artifactId>
		<version>${httpcomponents}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.httpcomponents</groupId>
		<artifactId>httpmime</artifactId>
		<version>${httpcomponents}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.httpcomponents</groupId>
		<artifactId>fluent-hc</artifactId>
		<version>${httpcomponents}</version>
	</dependency>
	<!-- json -->
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>fastjson</artifactId>
		<version>${fastjson}</version>
	</dependency>
	<!-- xml-->
	<dependency>
		<groupId>dom4j</groupId>
		<artifactId>dom4j</artifactId>
		<version>${dom4j}</version>
	</dependency>
	<!--dom4j xpath需要此jar-->
	<dependency>
		<groupId>jaxen</groupId>
		<artifactId>jaxen</artifactId>
		<version>${jaxen}</version>
	</dependency>
	<!--mybatis -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>${mybatis}</version>
	</dependency>
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis-spring</artifactId>
		<version>${mybatis-spring}</version>
	</dependency>
	<dependency>
		<groupId>org.mybatis.generator</groupId>
		<artifactId>mybatis-generator-core</artifactId>
		<version>${mybatis.generator}</version>
	</dependency>
	<!-- mail -->
	<dependency>
		<groupId>javax.mail</groupId>
		<artifactId>mail</artifactId>
		<version>${mail}</version>
		<exclusions>
			<exclusion>
				<groupId>javax.activation</groupId>
				<artifactId>activation</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>org.apache.commons</groupId>
		<artifactId>commons-email</artifactId>
		<version>${commons-email}</version>
		<exclusions>
			<exclusion>
				<groupId>javax.activation</groupId>
				<artifactId>activation</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<!-- jedis -->
	<dependency>
		<groupId>redis.clients</groupId>
		<artifactId>jedis</artifactId>
		<version>${jedis}</version>
	</dependency>
	<!-- freemarker -->
	<dependency>
		<groupId>org.freemarker</groupId>
		<artifactId>freemarker</artifactId>
		<version>${freemarker}</version>
	</dependency>
	<!-- log -->
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-api</artifactId>
		<version>${slf4j}</version>
	</dependency>
	<!-- log4j2.0 -->
	<dependency>
		<groupId>org.apache.logging.log4j</groupId>
		<artifactId>log4j-api</artifactId>
		<version>${log4j}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.logging.log4j</groupId>
		<artifactId>log4j-core</artifactId>
		<version>${log4j}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.logging.log4j</groupId>
		<artifactId>log4j-web</artifactId>
		<version>${log4j}</version>
		<scope>runtime</scope>
	</dependency>
	<!--log4j2 异步日志 需要此jar-->
	<dependency>
		<groupId>com.lmax</groupId>
		<artifactId>disruptor</artifactId>
		<version>${disruptor}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.logging.log4j</groupId>
		<artifactId>log4j-slf4j-impl</artifactId>
		<version>${log4j}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.logging.log4j</groupId>
		<artifactId>log4j-jcl</artifactId>
		<version>${log4j}</version>
	</dependency>
	<!-- test -->
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>${junit}</version>
		<scope>test</scope>
	</dependency>
	<!-- servlet、jsp、jstl -->
	<dependency>
		<groupId>javax.servlet</groupId>
		<artifactId>javax.servlet-api</artifactId>
		<version>3.0.1</version>
		<scope>provided</scope>
	</dependency>
	<dependency>
		<groupId>javax.servlet.jsp</groupId>
		<artifactId>javax.servlet.jsp-api</artifactId>
		<version>2.2.1</version>
		<scope>provided</scope>
	</dependency>
	<dependency>
		<groupId>javax.servlet</groupId>
		<artifactId>jstl</artifactId>
		<version>1.2</version>
	</dependency>
	<!-- file upload -->
	<dependency>
		<groupId>commons-fileupload</groupId>
		<artifactId>commons-fileupload</artifactId>
		<version>${commons-fileupload}</version>
	</dependency>
	<!--validator-->
	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-validator</artifactId>
		<version>${hibernate-validator}</version>
	</dependency>
</dependencies>
```

除了一些核心的jar包外，其他jar包可以根据需要添加或删除。

##2.2 设置不同的profile

项目的开发，测试，部署到线上阶段，通常有不同的环境变量，例如数据库的host，user，password，所需请求服务的host，port以及鉴权所需的一些key，secret等。为了方便的切换，我们通常会在pom.xml文件中设置若干不同的profile

```xml
<profiles>
	<profile>
		<id>local</id>
		<properties>
			<profiles.active>local</profiles.active>
			<log4j.level>INFO</log4j.level>
		</properties>
		<dependencies>
			<!-- database -->
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<version>${mysql}</version>
				<scope>runtime</scope>
			</dependency>
			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>druid</artifactId>
				<version>${druid}</version>
			</dependency>
		</dependencies>
		<build>
			<resources>
				<resource>
					<directory>src/main/resources</directory>
					<filtering>true</filtering>
				</resource>
			</resources>
		</build>
		<activation>
			<!-- 默认启用dev环境配置 -->
			<activeByDefault>true</activeByDefault>
		</activation>
	</profile>
	<profile>
		<id>test</id>
		<properties>
			<profiles.active>test</profiles.active>
		</properties>
		<build>
			<resources>
				<resource>
					<directory>src/main/resources</directory>
					<filtering>true</filtering>
				</resource>
			</resources>
		</build>
	</profile>
</profiles>
```