---
title: SpringBoot 入门笔记二 （mybatis集成）
date: 2017-08-19
tags: 
- java
- springboot
---


## 数据库连接

在`src/main/resources/application.properties`中配置：

```properties
spring.datasource.url=jdbc:postgresql://localhost/game
spring.datasource.username=dbuser
spring.datasource.password=passw0rd!
spring.datasource.driver-class-name=org.postgresql.Driver
```

详情看[这里](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-connect-to-production-database)

## service代码：

```java
package com.baobao.demo.repository;

import com.baobao.demo.pojo.Customer;
import org.apache.ibatis.annotations.*;

import java.util.List;

@Mapper
public interface CustomerRepository {
    @Insert("INSERT INTO customer(id, \"firstName\", \"lastName\") VALUES (#{id}, #{firstName}, #{lastName})")
    void insert(Customer customer);

    @Select("SELECT * FROM customer WHERE id = #{id}")
    Customer findById(int id);

    @Select("SELECT * FROM customer")
    List<Customer> findAll();

    @Delete("DELETE from customer WHERE id= ${id}")
    void removeById(@Param("id") int id);

    @Update("UPDATE customer SET \"firstName\" = #{firstName} , \"lastName\" = #{lastName} WHERE id = #{id}")
    void update(Customer customer);

    @Delete("DELETE from customer")
    void removeAll();
}

```

`@Mapper`和`@Insert、@Select、@Delete、@Update`都是`mybatis`提供的`annotation`，我们不需要去写service的实现类，只要在接口上加上`annotation`就可以了，只是我们需要`CURD`所有`sql`都写出来才行，貌似`mybatis`并没有提供一个接口帮助我们去完成这项操作。

字段映射不友好，这一点不如[Nutz](http://nutzam.com/)的设计，在方法上配置`@Results`，或者在`xml`中配置，个人觉得在实体类上写注解更加合理。

## 文档：

http://www.mybatis.org/mybatis-3/java-api.html

[mybatis 与 springboot集成](http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)