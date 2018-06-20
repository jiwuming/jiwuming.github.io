---
title: 巨坑的错误之javax.persistence.TransactionRequiredException:no transaction is in progress
tags: [Spring, Hibernate]
date: 2018-06-20
---

这个错误产生于我写的一个 SSH 项目, dao 层调用了一个 save 方法报的错。记录一下问题的解决过程。

首先, 就是测试一个把用户保存进数据库的操作:
```java
// Controller
public void addEmp() {
    EmpEntity empEntity = new EmpEntity();
    empEntity.setEmpno(7711);
    empEntity.setDeptno(10);
    empEntity.setEname("欣欣");
    empService.addEmp(empEntity);
}

// Service
public void addEmp(EmpEntity empEntity) {
    empDao.addEmp(empEntity);
}

// Dao
public void addEmp(EmpEntity empEntity) {
    sessionFactory.getCurrentSession().save(empEntity);
}
```
这个我执行之后数据并没有保存到数据库中去而且也没有报错...我到网上搜了一下, 有人说在 save 之后 flush 一下, 于是我这么做了:
```java
sessionFactory.getCurrentSession().save(empEntity);
sessionFactory.getCurrentSession().flush();
```
然后我调用了接口返回了一个500:
```bash
javax.persistence.TransactionRequiredException:no transaction is in progress
```
我就以为我的事务配置有问题, 看一下我的 `dataSource.xml` 配置:
<!-- more -->
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 读取配置文件 -->
    <context:property-placeholder location="classpath:db.properties" />

    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
        <property name="driverClass" value="${jdbc.driverClassName}" />
        <property name="user" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="jdbcUrl" value="${jdbc.url}" />
        <property name="maxPoolSize" value="${jdbc.maxPoolSize}" />
        <property name="minPoolSize" value="${jdbc.minPoolSize}" />
        <property name="initialPoolSize" value="${jdbc.initialPoolSize}" />
        <!-- 最大空闲时间 -->
        <property name="maxIdleTime" value="${jdbc.maxIdleTime}" />
    </bean>

    <!-- 配置sessionFactory -->
    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="packagesToScan" value="com.greg.ssh.entity" />
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.hbm2ddl.auto">${jdbc.hibernate.hbm2ddl.auto}</prop>
                <prop key="hibernate.dialect">${jdbc.hibernate.dialect}</prop>
                <prop key="hibernate.show_sql">${jdbc.hibernate.show_sql}</prop>
                <prop key="hibernate.format_sql">${jdbc.hibernate.format_sql}</prop>
                <prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate5.SpringSessionContext
                </prop>
            </props>
        </property>
    </bean>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <!-- 配置事务通知属性 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">

    <!-- 定义事务传播属性 -->
    <tx:attributes>
            <tx:method name="insert*" propagation="REQUIRED" rollback-for="Exception" />
            <tx:method name="update*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="upd*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="edit*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="save*" propagation="REQUIRED" rollback-for="Exception" read-only="false"/>
            <tx:method name="add*" propagation="REQUIRED" rollback-for="Exception" read-only="false"/>
            <tx:method name="query*" propagation="REQUIRED" />
            <tx:method name="new*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="set*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="remove*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="delete*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="del*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="change*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="check*" propagation="REQUIRED" rollback-for="Exception"/>
            <tx:method name="*" propagation="SUPPORTS" read-only="false"/>
        </tx:attributes>
    </tx:advice>

    <!-- 配置事务切面 -->
    <aop:config>
        <aop:pointcut id="serviceOperation"
                      expression="(execution(* com.greg.ssh.service.*.*(..)))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="serviceOperation"/>
    </aop:config>
</beans>
```
不过我也没怎么感觉我的事务管理器有什么问题...于是又去网上查了一下, 网上的说法有很多, 我看到几种说法:

1.在 `dataSource.xml` 中开启事务注解
```xml
<tx:annotation-driven transaction-manager="transactionManager" />
```
然后在 Service 上面加上 `@Transactional`
不过并没有效果, 我甚至试了在 Service 方法上面加 `@Transactional` 在 Dao 的方法和类上面都加过 `@Transactional` 这个问题还是没有解决。

2.在 `sessionFactory` 的配置中加入下面这句 说是自动提交事务:
```xml
<prop key="hibernate.connection.autocommit">true</prop>
```
不过我试过之后还是没有用...

后来我决定自己开启一个事务来试一下:
```java
Transaction transaction = sessionFactory.getCurrentSession().beginTransaction();
sessionFactory.getCurrentSession().save(empEntity);
transaction.commit();
```
果然成功了, 数据被保存进了数据库里。

那到底是什么位置出了问题呢...

后来我在[这篇博客](https://blog.csdn.net/u011217058/article/details/76076352)上找到了答案, 其实我的 `dataSource.xml` 配置文件是没有问题的, 问题出在我的 `springmvc.xml` 的配置上, 我在 `springmvc.xml` 中扫描了整个工程的注解, 而 spring 并不推荐这样做 :
>1.如果全部把注解放到spring.xml中配置： 
>当一旦采用这种方式之后，spring会将扫描的对象都会存放到spring的容器，而不会放到springmvc子容器中，当访问项目的时候，springmvc找不到处理器映射器，和其对应的Controller，进而报404错误！

>2.不用spring容器，把注解全部放到springmvc中扫描： 
>是可以的，在这个里面可以同时扫描Controller层、service层、dao层的注解，但是，子容器Controller进行扫描装配时装配了@Service注解的实例，而该实例理应由父容器进行初始化以保证事务的增强处理（因为事务管理器是配置在spring容器中的），所以此时得到的将是原样的Service（没有经过事务加强处理，故而没有事务处理能力。同理，springmvc中配置controller后也不能将事务配置在controller层，因为因为事务管理器是配置在spring容器中的，如果将事务配置在Controller层的话，spring容器就访问不了springmvc子容器，进而无法访问到事务对象。进而导致事务失效。

之后我把扫描注解分开, 在 `springmvc.xml` 中只扫描 controller 的注解, 而 service 和 dao 的注解扫描我放在了别的 spring 配置文件中, 上面的问题就都解决了。

最后看一下 web.xml
```xml
 <!-- 加载spring容器 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
         <!-- service 和 dao 的扫描我放在了这里, 其实也可以另写一个配置文件 -->
        <param-value>classpath:applicationContext-datasource.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- 加载springmvc前端控制器 -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- 这个里面只扫描 controller -->
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- session support -->
    <filter>
        <filter-name>SpringOpenSessionInViewFilter</filter-name>
        <filter-class>org.springframework.orm.hibernate5.support.OpenSessionInViewFilter</filter-class>
        <init-param>
            <param-name>flushMode</param-name>
            <param-value>AUTO</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>SpringOpenSessionInViewFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- 配置编码过滤器 -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
