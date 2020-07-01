# SpringSecurity
## 简介
https://docs.spring.io/spring-security/site/docs/4.2.10.RELEASE/guides/html5/helloworld-xml.html

1. SpringSecurity融合Spring技术栈，提供JavaEE应	用的整体安全解决方案；
2. Spring Security为基于Java EE的企业软件应用提供全面的安全服务。
3. Spring Security只需要少量配置，就能构建一个强大的安全的应用系统。  
4. 目前市面上受欢迎的两个安全框架：Apache Shiro、SpringSecurity
5. SpringSecurity可以无缝整合Spring应用，具有强大的自动化web安全管控功能。而Shiro是一个轻量级强大的安全框架，可以脱离web应用来提供安全管控，但是对于web的一些定制安全需要手动编写；SpringBoot底层默认整合SpringSecurity作为安全框架，所以我们推荐web应用使用SpringSecurity来控制安全；

## 概念
### 认证
authentication：身份验证
- “身份验证”是指建立主体（principal）的过程，主体就是他们声称是谁（“主体”通常指用户、设备或在应用程序中可以执行动作的其他系统）。也就是“证明你是谁”
### 授权
authorization：授权
- 授权”是指确定主体（principal）是否被允许执行系统中某个动作的过程。 也就是“你能做什么！”
- 为了达到“授权”决策（安全框架决定你是否有权限做此事），“身份验证”（authentication）过程已经建立了主体的身份（Principal）

## 支持的身份认证模式
在身份验证级别，Spring Security支持广泛的认证模型。这些认证模型中的大部分要么由第三方提供，要么由相关标准机构（如互联网工程任务组）开发。此外，Spring Security提供了自己的一套身份验证功能。具体而言，Spring Security当前支持与所有这些技术的身份验证集成；

-	HTTP BASIC身份验证标头
HTTP BASIC authentication headers (an IETF RFC-based standard)
参考：https://blog.csdn.net/lvxinzhi/article/details/49000003
-	HTTP Digest身份验证标头
HTTP Digest authentication headers (an IETF RFC-based standard)
-	HTTP X.509客户端证书交换
HTTP X.509 client certificate exchange (an IETF RFC-based standard)
-	LDAP（一种非常常见的跨平台身份验证方法，特别是在大型环境中）
LDAP (a very common approach to cross-platform authentication needs, especially in large environments)
-	**基于表单的身份验证（用于简单的用户界面需求）
Form-based authentication (for simple user interface needs)**
-	**OpenID身份验证
OpenID authentication**
-	基于预先建立的请求标头的身份验证
Authentication based on pre-established request headers (such as Computer Associates Siteminder)
-	**Jasig中央认证服务（也称为CAS，是一种流行的开源单点登录系统）
Jasig Central Authentication Service (otherwise known as CAS, which is a popular open source single sign-on system)**
-	远程方法调用（RMI）和HttpInvoker（Spring远程协议）的透明身份验证上下文传播
Transparent authentication context propagation for Remote Method Invocation (RMI) and HttpInvoker (a Spring remoting protocol)
-	**自动“记住我”身份验证**
Automatic "remember-me" authentication (so you can tick a box to avoid re-authentication for a predetermined period of time)
-	**匿名身份验证（允许每个未经身份验证自动承担特定的安全身份）**
Anonymous authentication (allowing every unauthenticated call to automatically assume a particular security identity)
-	Runas身份验证（如果一个调用应继续使用不同的安全标识，则非常有用）
Run-as authentication (which is useful if one call should proceed with a different security identity)
-	Java身份验证和授权服务（JAAS）
Java Authentication and Authorization Service (JAAS)
-	JavaEE容器身份验证（如果需要，您仍然可以使用容器管理身份验证）
Java EE container authentication (so you can still use Container Managed Authentication if desired)
-	**Java开源单点登录（JOSSO）**
Java Open Source Single Sign-On (JOSSO) *
-	OpenNMS网络管理平台*
OpenNMS Network Management Platform *
-	**您自己的身份验证系统
Your own authentication systems (see below)**
- 其他
    - Kerberos
    - AppFuse *
    - AndroMDA *
    - Mule ESB *
    - Direct Web Request (DWR) *
    - Grails *
    - Tapestry *
    - JTrac *
    - Jasypt *
    - Roller *
    - Elastic Path *
    - Atlassian Crowd *

## 模块划分
Core - spring-security-core.jar
核心模块
 : Core - spring-security-core.jar
核心模块	核心认证、授权功能、支持jdbc-user功能、支持独立的Spring应用

Remoting - spring-security-remoting.jar
远程交互模块 : 一般不需要，可以使用Spring Remoting功能简化远程客户端交互

Web - spring-security-web.jar
web安全模块 : web项目使用，基于URL的访问控制（access-control）

Config - spring-security-config.jar
java配置模块 : 必须依赖包，包含解析xml方式和java 注解方式来使用SpringSecurity功能

LDAP - spring-security-ldap.jar
ldap（轻量目录访问协议）支持模块 : 可选依赖包，LDAP功能支持

ACL - spring-security-acl.jar
ACL支持 : ACL（Access-Control-List）访问控制列表
细粒度的资源访问控制(RBAC+ACL)

CAS - spring-security-cas.jar
CAS整合支持 : CAS（Central Authentication Service）中央认证服务。开源ApereoCAS整合

OpenID - spring-security-openid.jar
OpenID认证方式支持 : OpenID Web身份验证支持。 用于针对外部OpenID服务器对用户进行身份验证
（微信,qq，新浪微博等第三方登录）

Test - spring-security-test.jar
测试模块 : 快速的测试SpringSecurity应用

## 4种使用方式
-	一种是全部利用配置文件，将用户、权限、资源(url)硬编码在xml文件中
-	二种是用户和权限用数据库存储，而资源(url)和权限的对应采用硬编码配置
-	**三种是细分角色和权限，并将用户、角色、权限和资源均采用数据库存储，并且自定义过滤器，代替原有的FilterSecurityInterceptor过滤器， 并分别实现AccessDecisionManager、InvocationSecurityMetadataSourceService和UserDetailsService，并在配置文件中进行相应配置。**
-	四是修改springsecurity的源代码，主要是修改InvocationSecurityMetadataSourceService和UserDetailsService两个类。

# SpringSecurity-HelloWorld

## 测试环境搭建
创建普通maven-war工程:spring-security-helloworld
````xml
	<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>4.3.20.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>jsp-api</artifactId>
			<version>2.2</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
````


web.xml配置


````xml
<servlet>
<servlet-name>springDispatcherServlet</servlet-name>
<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<init-param>
<param-name>contextConfigLocation</param-name>
<param-value>classpath:spring.xml</param-value>
</init-param>
<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
<servlet-name>springDispatcherServlet</servlet-name>
<url-pattern>/</url-pattern>
</servlet-mapping>
````

spring配置:spring.xml

````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
<context:component-scan base-package="com.atguigu.security"></context:component-scan>
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
<property name="prefix" value="/WEB-INF/views/"></property>
<property name="suffix" value=".jsp"></property>
</bean>
<mvc:annotation-driven />
<mvc:default-servlet-handler />
</beans>
````

## 引入SpringSecurity框架

添加security-pom依赖
````xml
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-web</artifactId>
<version>4.2.10.RELEASE</version>
</dependency>
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-config</artifactId>
<version>4.2.10.RELEASE</version>
</dependency>
<!-- 标签库 -->
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-taglibs</artifactId>
<version>4.2.10.RELEASE</version>
</dependency>
````

web.xml中添加SpringSecurity的Filter进行安全控制

````xml
<filter>
<filter-name>springSecurityFilterChain</filter-name><!--名称固定,不能变-->
<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
<filter-name>springSecurityFilterChain</filter-name>
<url-pattern>/*</url-pattern>
</filter-mapping>
````

加入SpringSecurity配置类

- @Configuration、@Bean 注解作用
````xml
@Configuration
@EnableWebSecurity
public class AppWebSecurityConfig extends WebSecurityConfigurerAdapter {
}
````

## 启动测试效果
-	所有资源访问受限（包括静态资源）
-	需要一个默认的登录页面（框架自带的）
-	账号密码错误会有提示
-	查看登录页面的源码，发现有个hidden-input；name="_csrf" 这是SpringSecurity帮我们防止“跨站请求伪造”攻击；还可以防止表单重复提交。
-	。。。
-	http://localhost:8080/spring-security-helloworld/login?error
