---
title: SSH框架-基于xml的ssh框架整合配置  
date: 2018-08-28 15:09:16  <!--创建日期-->
updated: 2018-08-28 15:09:16	<!--更新日期-->
categories: 框架    <!--只能有一个分类-->  
tags: [框架, ssh]
comments: false <!--开启文章的评论功能-->
description: 
<!--photos:
  - 图片的网络地址 可以在你的文章前面显示一张大图-->
---

## 摘要

学习过程中记录了基于xml的ssh框架整合的一些简单配置

### 需要的配置文件

>Strtsu2框架：src/strtus.xml
Hibernate框架：src/hibernate.cfg.xml；在domain有 Xxx.hbm.xml
Spring框架：src/applicationContext.xml
关于日志：log4j.properties
关于数据库连接：db.properties

<!-- more --> 

### web.xml配置(必须)

#### 配置spring监听器及指定配置文件位置
~~~xml
<!-- 配置spring的监听器 -->
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!-- 指定spring框架配置文件所在位置 -->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:applicationContext.xml</param-value>
</context-param>
~~~

#### 配置struts2的filter
~~~xml
<!-- 配置structs2 -->
<filter>
	<filter-name>struts2</filter-name>
	<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<filter-mapping>
	<filter-name>struts2</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
~~~

### spring整合hibernate
#### 方式一：零障碍整合
注意：需要hibernate.cfg.xml配置文件，引用时根据需求变动处为：1.配置数据库相关信息；2.mapping标签映射实体类
```xml
<bean id="sessionFactory"
	class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
	<property name="configLocation" value="classpath:hibernate.cfg.xml"></property>
</bean>
```
#### 方式二：spring管理hibernate配置(建议使用)
> 该方式不需要hibernate.cfg.xml文件，所有关于hibernate.cfg.xml文件中的配置都在spring的配置文件中来配置
详细配置见applicationContext.xml文件

==注意：==

1、需要引入`db.properties`文件，并且数据库相关信息都在`db.properties`文件中进行配置
```xml
<!-- 引入外部 的properties文件 -->
<context:property-placeholder location="classpath:db.properties" />
```
2、配置连接池
```xml
<!-- 创建c3p0连接池 -->
<bean id="c3p0DataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="driverClass" value="${jdbc.driverClass}" />
	<property name="jdbcUrl" value="${jdbc.url}" />
	<property name="user" value="${jdbc.username}" />
	<property name="password" value="${jdbc.password}" />
</bean>
```
3、配置`sessionFactory`
1. 加载上述步骤配置的连接池
2. 配置hibernateProperties
3. 加载domain下的`类名.hbm.xml`
```xml
<bean id="sessionFactory"
	class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
	<!-- 加载连接池 -->
	<property name="dataSource" ref="c3p0DataSource"></property>
	<property name="hibernateProperties">
		<value>
			hibernate.show_sql=true
			hibernate.format_sql=true
			<!-- hibernate的方言，必须配置 -->
			hibernate.dialect=org.hibernate.dialect.MySQLDialect
			hibernate.hbm2ddl.auto=update
		</value>
	</property>
	
	<!-- 加载hibernate的Xxx.hbm.xml配置文件 -->
	<property name="mappingResources">
		<list>
			<value>com/itheima/domain/User.hbm.xml</value>
		</list>
	</property>
</bean>
```
4、声明dao
==注意==：dao需要继承`HibernateDaoSupport`类，此时注入`sessionFactory`才可以获得`HibernateTemplate`进行数据持久化操作
```xml
<!-- 声明UserDao -->
<bean id="userDao" class="com.itheima.dao.UserDaoImpl">
	<!-- 注入sessionFactory -->
	<property name="sessionFactory" ref="sessionFactory" />
</bean>
```
5、声明service
==service类中需要提供dao的set方法以便dao的注入==
```xml
<bean id="userService" class="com.itheima.service.UserServiceImpl">
	<property name="userDao" ref="userDao" />
</bean>
```
6、事务管理
```xml
<!-- 声明事务管理器 -->
<bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
	<property name="sessionFactory" ref="sessionFactory"></property>
</bean>
<!-- 通知 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<tx:method name="find*" read-only="true"/>
		<tx:method name="add"/>
		<tx:method name="update"/>
		<tx:method name="delete"/>
	</tx:attributes>
</tx:advice>
<!-- 声明切面 -->
<aop:config>
	<!-- 声明切点 -->
	<aop:pointcut expression="execution(* com.itheima.service.*..*(..))" id="myPointcut"/>
	<aop:advisor advice-ref="txAdvice" pointcut-ref="myPointcut"/>
</aop:config>
```
### Spring整合struts2框架
注意：
> 1.需要struts2-spring-plugin.jar
> 2.action类需要继承`ActionSupport`;
> 3.数据封装：实现`ModelDriven<T>`接口并提供实例化bean对象

配置struts.xml
```xml
<struts>
	<package name="default" namespace="/" extends="struts-default">
		<action name="user_*" class="userAction"
			method="{1}">
			<result name="success">/success.jsp</result>
		</action>
	</package>
</struts>
```
#### 方式一：spring管理action(建议使用)
1、`applicationContext.xml`中配置
```xml
<!-- 配置action -->
<bean id="userAction" class="com.itheima.action.UserAction" scope="prototype">
	<property name="userService" ref="userService"/>
</bean>
```
2、action类中需要提供service的set方法以便service的注入

3、上述struts.xml中action标签中class的配置使用bean中的id

4、这种方式必须配置`scope="prototype"`，因为action是多例的

#### 方式二：action自动注入service
applicationContext.xml中不需要配置action标签，struts.xml文件中class使用全类名，此时使用自动注入方案

### 延迟加载的问题
在web.xml中配置openSessionInViewFilter
注意:openSessionInViewFilter一定要在Struts2 Filter前配置
	
```xml
<!-- openSessionInViewFilter -->
<filter>
	<filter-name>openSessionInView</filter-name>
	<filter-class>org.springframework.orm.hibernate5.support.OpenSessionInViewFilter</filter-class>
</filter>

<filter-mapping>
	<filter-name>openSessionInView</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```