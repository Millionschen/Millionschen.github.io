---
layout: post
title: 使用maven搭建spring+spring-mvc+mybatis应用
category: javaweb
tags: Essay
---


# 1 基本概念

## 1.1 spring

Spring是一个分层的JavaSE/EE full-stack轻量级开源框架。它是为了解决企业应用开发的复杂性而创建的。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。 简单来说，Spring是一个轻量级的控制反转（IoC）和面向切面（AOP）的容器框架。

## 1.2 spring-mvc

Spring MVC属于SpringFrameWork的后续产品，已经融合在Spring Web Flow里面。Spring MVC 分离了控制器、模型对象、分派器以及处理程序对象，这种分离让它们更容易进行定制。

## 1.3 mybatis

mybatis是一个轻量级的持久层框架， 包括SQL Maps和Data Access Objects（DAO）MyBatis 消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。MyBatis 使用简单的 XML或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Old Java Objects，普通的 Java对象）映射成数据库中的记录。

## 1.4 maven

Maven项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。

# 2 准备pom文件

## 2.1 引入相关jar包

使用maven可以导入并管理项目所需的jar包，我们的`<modelVersion>4.0.0</modelVersion>`，项目结构为：

![项目目录结构](http://ethanatos.qiniudn.com/maven_web%E9%A1%B9%E7%9B%AE%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)

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

## 2.2 设置不同的profile

项目的开发，测试，部署到线上阶段，通常有不同的环境变量，例如数据库的host，user，password，所需请求服务的host，port以及鉴权所需的一些key，secret等。为了方便的切换，我们通常会在pom.xml文件中设置若干不同的profile

```xml
<profiles>
	<profile>
		<id>dev</id>
		<properties>
			<profiles.active>local</profiles.active>
		</properties>
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

以上我们定义了两个不同的maven profile， 在之后的配置，代码中我们可以使用不同的profile来对项目进行区分配置。filtering指是否在编译期间使用property替换resource文件中的占位符。


## 2.3 设置build信息

之后是在pom.xml文件中配置项目的build信息：

```xml
<build>
	<finalName>${project.artifactId}</finalName>
	  <resources>
            <resource>
                <directory>src/main/webapp</directory>
                <filtering>true</filtering>
                <includes>
                    <include>WEB-INF/web.xml</include>
                </includes>
                <targetPath>${project.build.directory}/${project.build.finalName}</targetPath>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>opt/${opt}/**</include>
                    <include>config.properties</include>
                    <include>log4j2.xml</include>
                </includes>
                <filtering>true</filtering>
        </resource>
    </resources>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.5.1</version>
			<configuration>
				<source>${jdk}</source>
				<target>${jdk}</target>
				<encoding>${project.build.sourceEncoding}</encoding>
			</configuration>
		</plugin>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-resources-plugin</artifactId>
			<version>2.7</version>
			<configuration>
				<encoding>${project.build.sourceEncoding}</encoding>
			</configuration>
		</plugin>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-surefire-plugin</artifactId>
			<version>2.19.1</version>
			<configuration>
				<skip>true</skip>
			</configuration>
		</plugin>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-war-plugin</artifactId>
			<version>2.6</version>
			<configuration>
				<archive>
					<manifest>
						<addDefaultImplementationEntries>true</addDefaultImplementationEntries>
					</manifest>
					<manifestEntries>
						<Implementation-Version>${project.version}</Implementation-Version>
					</manifestEntries>
				</archive>
				<webXml>src/main/webapp/WEB-INF/web.xml</webXml>
			</configuration>
		</plugin>
</build>
```

上面是一个javaweb项目build所需的maven plugins，可以根据需要进行调整。注意我们在resources中定义了多个resource，这几个resource的定义互相叠加，构成了最终war包里的resource，首先指定了WEB_INF/web.xml的位置，然后指定除了几个文件外的resource不需要filtering，最后说明几个特殊文件需要filtering,注意resource插件可以对配置文件进行filter，但是它默认把resource放入WEB-INF/class下，所以需要指定他将WEB.xml放置到WEB-INF下。

# 3 设置web.xml

本文既然使用了spring-mvc，那么当然是一个jave web项目了，接下来，我们就进行项目的web.xml的配置。在配置前，我们先来温习一下有关j2ee和spring的一些基础知识：

>ApplicationContext是spring的核心，Context通常解释为上下文环境，用“容器”来表述更容易理解一些，ApplicationContext则是“应用的容器了”了。ServletContext 是Servlet与Servlet容器之间直接通信的接口，Servlet容器在启动一个web应用时，会为它创建一个ServletContext对 象，每个web应用有唯一的ServletContext对象，同一个web应用的所有Servlet对象共享一个ServletContext，Servlet对象可以通过它来访问容器中的各种资源

有了以上的了解，我们可以知道，将applicationContext放入ServletContext中，即可随时从其中获取applicationContext，从而获取spring中的各种bean。对于spring-mvc，我们一般将spring分为若干个“容器”，分别放入ServletContext中，如下图所示：

![spring-mvc web.xml配置](http://images.cnitblog.com/blog/698747/201502/011528042224276.png)

这样，整个应用的启动过程可以总结为：

1. servlet容器启动，为应用创建一个“全局上下文环境”：ServletContext
2. 容器调用web.xml中配置的contextLoaderListener，初始化WebApplicationContext上下文环境（即IOC容器），加载context-param指定的配置文件信息到IOC容器中。WebApplicationContext在ServletContext中以键值对的形式保存
3. 容器初始化web.xml中配置的servlet，为其初始化自己的上下文信息，并加载其设置的配置信息到该上下文中。将WebApplicationContext设置为它的父容器。
4. 此后的所有servlet的初始化都按照3步中方式创建，初始化自己的上下文环境，将WebApplicationContext设置为自己的父上下文环境。

这样在启动后，在DispatcherServlet中可以引用由ContextLoaderListener所创建的父容器ApplicationContext中的内容，而反过来不行。当Spring在执行DispatcherServlet的ApplicationContext的getBean时，如果在自己context中找不到对应的bean，则会在父ApplicationContext中去找。这也解释了为什么我们可以在DispatcherServlet中获取到由ContextLoaderListener对应的ApplicationContext中的bean。这样做区分了处理http请求的逻辑与业务逻辑，整个项目更加清晰。

随后，我们就来看看web.xml的配置，首先定义spring的profile，注意`profiles.active`在pom文件中不同的profile里已经定义过了。：

```xml
<context-param>
	<param-name>spring.profiles.active</param-name>
	<param-value>${profiles.active}</param-value>
</context-param>
```

接下来就是定义WebApplicationContext,通过context-param contextConfigLocation指定context文件的位置，然后通过spring提供的`org.springframework.web.context.ContextLoaderListener`加载配置文件生成该web应用的IOC父容器：

```xml
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath*:context/**/*.xml</param-value>
</context-param>
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

然后就是定义dispacher-servlet,注意我们使用init-param contextConfigLocation指定了其servlet-context配置文件的位置，从而生成servlet-context

```xml
<servlet>
	<servlet-name>test</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath*:servlet/**/*.xml,
            </param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>test</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
```

最后，我们配置字符编码过滤器,来将所有请求都转换为utf-8编码：

```xml
<filter>
	<filter-name>encodingFilter</filter-name>
	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	<init-param>
		<param-name>encoding</param-name>
		<param-value>UTF-8</param-value>
	</init-param>
	<init-param>
		<param-name>forceEncoding</param-name>
		<param-value>true</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>encodingFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

# 4. 配置context

## 4.1 servlet-context的配置

在servelet-context中，我们一般配置对controller包的component-scan，打开annotation-driven，以及对http的request和response的格式转换。

* component-scan 这里如我们之前所述，只scan controller
	
```xml
<context:component-scan base-package="your.app.controller"/>
```

* annotation-driven 以及 request和response的格式转换，如把object转换为json，xml等等。这里我们配置了fastJson作为json转换，marshallingConverter作为xml转换 以及org.springframework.http.converter.StringHttpMessageConverter作为string转换

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8"/>
            </bean>
            <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
                <property name="features">
                    <array value-type="com.alibaba.fastjson.serializer.SerializerFeature">
                        <value>NotWriteDefaultValue</value>
                        <value>DisableCircularReferenceDetect</value>
                    </array>
                </property>
                <property name="supportedMediaTypes">
                    <list>
                        <value>application/json;charset=UTF-8</value>
                        <value>text/html;charset=UTF-8</value>
                        <value>text/plain;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
             <bean id="marshallingConverter"
          class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
        <constructor-arg ref="jaxbMarshaller"/>
        <property name="supportedMediaTypes" value="application/xml;charset=UTF-8"/>
    			</bean>
    </mvc:message-converters>
</mvc:annotation-driven>
<bean class="org.springframework.oxm.jaxb.Jaxb2Marshaller" id="jaxbMarshaller">
    <property name="classesToBeBound">
        <list>
            <value>com.ins.life.order.dto.Message</value>
        </list>
    </property>
    <property name="marshallerProperties">
        <map>
            <entry key="jaxb.encoding" value="UTF-8"/>
            <entry key="jaxb.formatted.output">
                <bean class="java.lang.Boolean">
                    <constructor-arg value="true"/>
                </bean>
            </entry>
        </map>
    </property>
</bean>
```

* 注册multipartResolver(解析上传文件)，localResolver(用户i18n)，viewResolver(解析视图)

## 4.2 配置Application-context

application-contex是整个web应用的业务、功能组件的配置文件，可以根据需要进行拆分，放入不同的配置文件中。

### 4.2.1 配置mybatis

首先我们配置mybatis，在配置mybatis前，需要先配置一个dataSource,通常我们将dataSource所需的参数放入配置文件中，使用propertySourcePlaceHolder加载

```xml
<!-- 数据库、保险公司参数等配置文件位置 -->
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="order" value="1"/>
    <property name="ignoreUnresolvablePlaceholders" value="true"/>
    <property name="locations">
        <list>
            <value>classpath:dbconfig.properties</value>
        </list>
    </property>
</bean>
```

之后，配置dataSource,这里我们使用阿里巴巴的druid，不同dataSource的配置需要参考各自的配置文件

```xml
<bean class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close" id="dataSource" init-method="init">
    <!-- 数据库基本信息配置 -->
    <property name="driverClassName" value="${driverClassName}"/>
    <property name="url" value="${jdbcUrl}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <!-- 初始化连接数量 -->
    <property name="initialSize" value="${initialSize}"/>
    <!-- 最大并发连接数 -->
    <property name="maxActive" value="${maxActive}"/>
    <!-- 最大空闲连接数
            <property name="maxIdle" value="${maxIdle}"/>-->
    <!-- 最小空闲连接数 -->
    <property name="minIdle" value="${minIdle}"/>
    <!-- 配置获取连接等待超时的时间 -->
    <property name="maxWait" value="${maxWait}"/>
    <!-- 超过时间限制是否回收 -->
    <property name="removeAbandoned" value="${removeAbandoned}"/>
    <!-- 超过时间限制多长； -->
    <property name="removeAbandonedTimeout" value="${removeAbandonedTimeout}"/>
    <!-- 关闭abanded连接时输出错误日志 -->
    <property name="logAbandoned" value="${logAbandoned}"/>
    <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
    <property name="timeBetweenEvictionRunsMillis" value="${timeBetweenEvictionRunsMillis}"/>
    <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
    <property name="minEvictableIdleTimeMillis" value="${minEvictableIdleTimeMillis}"/>
    <!-- 用来检测连接是否有效的sql，要求是一个查询语句-->
    <property name="validationQuery" value="${validationQuery}"/>
    <!-- 申请连接的时候检测 -->
    <property name="testWhileIdle" value="${testWhileIdle}"/>
    <!-- 申请连接时执行validationQuery检测连接是否有效，配置为true会降低性能 -->
    <property name="testOnBorrow" value="${testOnBorrow}"/>
    <!-- 归还连接时执行validationQuery检测连接是否有效，配置为true会降低性能  -->
    <property name="testOnReturn" value="${testOnReturn}"/>
    <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
    <property name="poolPreparedStatements" value="${poolPreparedStatements}"/>
    <property name="maxPoolPreparedStatementPerConnectionSize" value="${maxPoolPreparedStatementPerConnectionSize}"/>
    <!--属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：
                    监控统计用的filter:stat
                    日志用的filter:log4j
                   防御SQL注入的filter:wall -->
    <property name="filters" value="${filters}"/>
</bean>
```

然后，配置事务管理,如果需要通过注解控制事务，加入tx:annotation-driven：

```xml
<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="txManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- enable transaction annotation support -->
<tx:annotation-driven transaction-manager="txManager" />
```

配置好dataSource与transactionManager后，就可以来进行mybatis的配置了，首先配置sqlSessionFactory，使用之前配置的dataSource.设置了typeAliasesPackage以后，在mapper中使用model时就可以不用加上完全限定类名了

```xml
<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
    <!-- 实例化sqlSessionFactory时需要使用上述配置好的数据源以及SQL映射文件 -->
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <!-- 自动配置别名 -->
    <property name="typeAliasesPackage" value="your.model.package"/>
    <property name="mapperLocations">
        <array>
            <value>classpath*:mapper/**/*.xml</value>
        </array>
    </property>
</bean>
```

生成一个mybatis的配置文件`mybatis-config.xml`,相关个性化配置可以查看mybatis的文档

```xml
<configuration>
	<settings>
       <setting name="lazyLoadingEnabled" value="false"/>
		<setting name="cacheEnabled" value="false"/>
	</settings>
</configuration>
```

配置一个mapper MapperScannerConfigurer，扫描mapper所在的位置

```xml
<!-- 配置扫描器 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- 扫描your.package.to.mapper这个包以及它的子包下的所有映射接口类 -->
    <property name="basePackage" value="your.package.to.mapper"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

### 4.2.2 其他bean的配置

配置其他业务逻辑和工具bean，以及component-scan,可以通过` <context:exclude-filter type="annotation"
        expression="org.springframework.stereotype.Controller"/>`进行scan的配置:

```xml
<context:component-scan base-package="package1;packge2"/>
```

# 5 总结
至此，一个基本的spring+spring-mvc+mybatis配置就搭建好了，接下来就能进行愉快的开发了。

