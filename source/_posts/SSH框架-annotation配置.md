---
title: SSH框架-基于annotation的ssh框架整合配置   
date: 2018-08-28  <!--创建日期-->
updated: 	<!--更新日期-->
categories: 框架    <!--只能有一个分类-->  
tags: [ssh, 注解]
comments: false<!--开启文章的评论功能-->
description: 
<!--photos:
  - 图片的网络地址 可以在你的文章前面显示一张大图-->
---

## 摘要

本文简单的记录了基于annotation进行ssh框架整合配置；

### 需要的配置文件
>Strtsu2框架：src/strtus.xml
>Spring框架：src/applicationContext.xml
>关于日志：log4j.properties
>关于数据库连接：db.properties

<!-- more --> 

### web.xml配置(同基于xml的配置)

### domain下bean的注解配置
* @Entity：定义实体类
* @Table：定义表
* @Id：主键
* @GeneratedValue：注解生成策略
* @Column：定义列(可以省略)

### 整合hibernate
applicationContext.xml进行配置

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
3、开启注解扫描
```xml
<!-- 开启注解扫描 -->
<context:component-scan base-package="com.itheima"/>
```
4、配置sessionFactory
1. 加载上述步骤配置的连接池
2. 配置hibernateProperties
3. 加载注解类
```xml
<bean id="sessionFactory"
	class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
	<property name="dataSource" ref="c3p0DataSource"></property>
	<property name="hibernateProperties">
		<value>
			hibernate.show_sql=true
			hibernate.format_sql=true
			hibernate.dialect=org.hibernate.dialect.MySQLDialect
			hibernate.hbm2ddl.auto=update
		</value>
	</property>

	<!-- 加载注解类 -->
	<property name="packagesToScan">
		<list>
			<value>com.itheima.domain</value>
		</list>
	</property>
</bean>
```
5、声明事务管理器
```xml
<!-- 声明事务管理器 -->
<bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
	<property name="sessionFactory" ref="sessionFactory"></property>
</bean>
```
6、加载事务注解驱动
```xml
<!-- 事务注解驱动 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```
> bean的注册
@Component：不确定位置
@Rpository：用于DAO层
@Service：用于service层
@Controller：用于表现层

### dao注解配置
a、@Repository("userDao")：注册bean

b、注入sessionFactory
```java
@Autowired
@Qualifier("sessionFactory")
public void setSuperSessionFactory(SessionFactory sessionFactory){
	super.setSessionFactory(sessionFactory);
}
```
### service注解配置
a、@Service("userService")：注册bean	// 注意：要定义名字，否则在action注入时可能报错

b、@Transactional：开启事务

c、注入dao：需要提供set方法
```java
// 注入userDao
@Autowired
@Qualifier("userDao")
private IUserDao userDao;

public void setUserDao(IUserDao userDao) {
	this.userDao = userDao;
}
```
### action注解配置
注意：
> 1.需要struts2-spring-plugin.jar
> 2.action类需要继承`ActionSupport`;
> 3.数据封装：实现`ModelDriven<T>`接口并提供实例化bean对象

1、类上基本配置
	@Controller
	@Scope("prototype")
	@Namespace("/")
	@ParentPackage("struts-default")

2、service注入
```java
@Autowired
@Qualifier("userService")
private IUserService userService;
```
3、配置action
```java
@Action(value="user_add",results={@Result(name="success",location="/success.jsp")})
public String add() {
	userService.add(user);
	return SUCCESS;
}
```
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