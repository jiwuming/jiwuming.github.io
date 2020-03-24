---
title: 使用Maven完成Mybatis的逆向工程
tags: [Maven, Intellij IDEA]
date: 2018-05-28
---
今天使用 Maven 插件的方式来对 Mybatis 进行逆向操作, 自动生成dao mapper 和 po 类, 记录一下配置过程

首先先建立一个 Maven 工程, 然后创建相应的目录及`generatorConfig.xml`文件。
![](/img/mybatisR-menu.png)
然后配置需要的jar包及插件
```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.2</version>
</dependency>
 <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.11</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
```
<!-- more -->
```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.2</version>
    <dependencies>
         <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.11</version>
        </dependency>
    </dependencies>
    <configuration>
        <!--配置文件的路径-->
        <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
        <overwrite>true</overwrite>
    </configuration>
</plugin>
```
`generatorConfig.xml`文件中的内容:
```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="test" targetRuntime="MyBatis3">
        <plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin" />
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/ >
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
        <commentGenerator>
            <property name="suppressDate" value="true" />
            <!-- 是否去除自动生成的注释 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!--数据库链接URL，用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql:///mybatis"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- 非必需，类型处理器，在数据库类型和java类型之间的转换控制-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        <!-- 生成模型的包名和位置 -->
        <javaModelGenerator targetPackage="com.greg.po"
                            targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- 生成映射文件的包名和位置 -->
        <sqlMapGenerator targetPackage="com.greg.mapper"
                         targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- 生成dao的包名和位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.greg.dao" implementationPackage="com.greg.dao"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- 要生成哪些表 domainObjectName: 实体类 是否生成example这里选择不生成-->
        <table tableName="items" domainObjectName="Items" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false"
    enableSelectByExample="false" selectByExampleQueryId="false"></table>
    </context>
</generatorConfiguration>
```
接下来配置一个启动项
![](/img/mybatisR-config.png)
这个 command line `mybatis-generator:generate -e`不要写错
然后我们运行就可以了
![](/img/mybatisR-generatorStart.png)
最终得到的结果目录
![](/img/mybatisR-get.png)
