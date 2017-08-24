---
title: SpringBoot 入门笔记一（Controller）
date: 2017-08-18 18:44:48
tags: 
- java
- springboot
---

## 创建项目

这里使用[http://start.spring.io/](http://start.spring.io/)在线创建的项目，然后导入`IntelliJ`，需要将`src/main`点击右键标记为`Sources Root`，这样我们就可以开始Coding啦！

下面是生成好的`build.gradle`文件

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.6.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'

version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-rest')
    compile('org.springframework.boot:spring-boot-starter-freemarker')
    compile('org.springframework.boot:spring-boot-starter-jdbc')
    compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.0')
    compile('org.springframework.boot:spring-boot-starter-web')
    runtime('org.postgresql:postgresql')
    testCompile('org.springframework.boot:spring-boot-starter-test')
    testCompile('org.springframework.restdocs:spring-restdocs-mockmvc')
}
```

## 编写Controller

> 先随便写一个Controller

```java
package com.baobao.demo;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class IndexController {
    @RequestMapping("/")
    public String index() {
        return "Hello!";
    }
}

```

运行`./gradlew bootRun`命令或者在`gradle 面板`中双击`bootRun`都可以 ，你可能会运行失败，项目中`build.gradle`依赖了`JDBC`和`myBatis`在没有配置数据库连接的时候运行会失败，我们先将依赖注释掉，重新运行刚刚命令，打开浏览器输入[http://localhost:8080/](http://localhost:8080/)。

### Controller 的测试

```java
package com.baobao.demo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.Assert.*;
import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;

@RunWith(SpringRunner.class)
@WebMvcTest(IndexController.class)
public class IndexControllerTest {

    @Autowired
    private  MockMvc mockMvc;

    @Test
    public void index() throws Exception {
        this.mockMvc.perform(get("/"))
                .andExpect(status().isOk())
                .andDo(print()) // 打印请求结果
                .andExpect(content().string(containsString("Hello!"))); // 验证结果是否正确

    }
}
```

测试的参考：

https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html

### 生成[asciidoctor](http://asciidoctor.org/)文档

对上面的测试代码做简单的修改：

```java
package com.baobao.demo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.Assert.*;
import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;

@RunWith(SpringRunner.class)
@WebMvcTest(IndexController.class)
@AutoConfigureRestDocs(outputDir = "target/snippets")
public class IndexControllerTest {

    @Autowired
    private  MockMvc mockMvc;

    @Test
    public void index() throws Exception {
        this.mockMvc.perform(get("/"))
                .andExpect(status().isOk())
                .andDo(print()) // 打印请求结果
                .andExpect(content().string(containsString("Hello!"))) // 验证结果是否正确
                .andDo(document("index")); // 将文档输出到target/snippets/index目录

    }
}
```

其中`@AutoConfigureRestDocs`注解是用来生成[asciidoctor](http://asciidoctor.org/)文档的，与最后的`document("index")`相对应，运行测试会生成如下片段：

```
└── target
    └── snippets
        └── index
            └── httpie-request.adoc
            └── curl-request.adoc
            └── http-request.adoc
            └── http-response.adoc
```

#### [asciidoctor](http://asciidoctor.org/)文档转HTML

我们需要的不只是上面的片段，还需要一个可读的`html`，我们来看看如何做呢？

1. 修改build.gradle：

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.6.RELEASE'
	}
	repositories {
        jcenter()
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        // asciidoctor
        classpath("org.asciidoctor:asciidoctor-gradle-plugin:1.5.3")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'
apply plugin: 'org.asciidoctor.convert'

version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}
// asciidoctor 转 html
asciidoctor {
    sourceDir 'src/main/asciidoc'
    attributes \
      'snippets': file('target/snippets')
    dependsOn test // 依赖测试生成的文件
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-rest')
    compile('org.springframework.boot:spring-boot-starter-freemarker')
    compile('org.springframework.boot:spring-boot-starter-jdbc')
    compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.0')
    compile('org.springframework.boot:spring-boot-starter-web')
    runtime('org.postgresql:postgresql')
    testCompile('org.springframework.boot:spring-boot-starter-test')
    testCompile('org.springframework.restdocs:spring-restdocs-mockmvc')
}

```

2. 创建`scr/main/asciidoc/index.adoc`

```
= 请求"/"，indexController

This is an example output for a service running at http://localhost:8080:

.CURL
--
include::{snippets}/index/curl-request.adoc[]
--
.request
--
include::{snippets}/index/http-request.adoc[]
--
.response
--
include::{snippets}/index/http-response.adoc[]
--
As you can see the format is very simple, and in fact you always get the same message.
```

这里关键在于`include`指令，会将指定文件包含进来，我们运行`asciidoctor`任务就可以在`build/asciidoc/html5`找到生成好的html文件啦。

#### response字段描述

应用中常常需要返回`json`格式，对于`json`字段的含义似乎只有开发阶段才会记得，时间久了我们需要一个文档，`asciidoctor`不仅可以帮助我们生成请求和返回的结果，还可以对返回的字段添加描述，我们来写一个稍稍复杂的例子：

1. pojo

```java
package com.baobao.demo.pojo;

public class Customer {

    private int id;
    private String firstName,lastName;

    public Customer(int id, String firstName, String lastName) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String toString() {
        return String.format(
                "Customer[id=%d, firstName='%s', lastName='%s']", id, firstName, lastName);
    }

    public int getId() {
        return id;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}

```

2. 修改IndexController

```java
package com.baobao.demo;

import com.baobao.demo.pojo.Customer;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

@RestController
public class IndexController {
    @RequestMapping("/")
    public String index() {
        return "Hello!";
    }

    @RequestMapping("/customers")
    public List<Customer> getCustomerList() {
        List<Customer> list = new ArrayList<>();
        list.add(new Customer(1, "比尔", "盖茨"));
        list.add(new Customer(2,"奥", "奥巴马"));
        return list;
    }
}

```

3. 修改IndexControllerTest

```java
package com.baobao.demo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.restdocs.payload.*;
import org.springframework.http.*;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.restdocs.payload.PayloadDocumentation.responseFields;
import static org.springframework.restdocs.payload.PayloadDocumentation.fieldWithPath;


@RunWith(SpringRunner.class)
@WebMvcTest(IndexController.class)
@AutoConfigureRestDocs(outputDir = "target/snippets")
public class IndexControllerTest {

    @Autowired
    private  MockMvc mockMvc;

    @Test
    public void index() throws Exception {
        this.mockMvc.perform(get("/"))
                .andExpect(status().isOk())
                .andDo(print()) // 打印请求结果
                .andExpect(content().string(containsString("Hello!"))) // 验证结果是否正确
                .andDo(document("index")); // 将文档输出到target/snippets/index目录
    }

    @Test
    public void getCustomerList() throws Exception {
        FieldDescriptor[] customerDescriptor = new FieldDescriptor[] {
            fieldWithPath("id").description("id"),
            fieldWithPath("firstName").description("客户的名字"),
            fieldWithPath("lastName").description("客户的姓")
        };
        this.mockMvc.perform(get("/customers").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andDo(print()) // 打印请求结果
                .andExpect(content().json("\n" +
                        "[{\"id\":1,\"firstName\":\"比尔\",\"lastName\":\"盖茨\"},{\"id\":2,\"firstName\":\"奥\",\"lastName\":\"奥巴马\"}]")) // 验证结果是否正确
                .andDo(document("customers", responseFields(fieldWithPath("[]")
                                             .description("客户列表"))
                                             .andWithPrefix("[].", customerDescriptor)));
    }

}
```

增加了`FieldDescriptor`和`responseFields()`，这些会帮助我们在`target/spnippets/customers/`生成一个`response-fields.adoc`文件。

4. 创建customers.adoc

```
= 请求"/customers"，indexController

This is an example output for a service running at http://localhost:8080:

.CURL
--
include::{snippets}/customers/curl-request.adoc[]
--
.request
--
include::{snippets}/customers/http-request.adoc[]
--
.response
--
include::{snippets}/customers/http-response.adoc[]
--
.response-fields
--
include::{snippets}/customers/response-fields.adoc[]
--
As you can see the format is very simple, and in fact you always get the same message.
```

与index.adoc稍有不同的是我们在下面多引入了一个`response-fields.adoc`文件，再次运行`asciidoctor`任务，结果如下：

*response-fields*

| Path           | Type     | Description |
| -------------- | -------- | ----------- |
| `[]`           | `Array`  | 客户列表        |
| `[].id`        | `Number` | id          |
| `[].firstName` | `String` | 客户的名字       |
| `[].lastName`  | `String` | 客户的姓        |

#### 参数描述

> 这段已经不想写了，在文章最下方已经有很全的代码demo，在自己实践过程中竟然踩到了一个坑所以还是写出来的好，万一有人看呢~O(∩_∩)O

因为其余部分代码比较简单，这里只贴关键代码

```java
import ...
//import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
// 注意：这里要使用RestDocumentationRequestBuilders.get 否则生成文档时会报错
import static org.springframework.restdocs.mockmvc.RestDocumentationRequestBuilders.get;
import ...

@Test
public void getCustomer() throws Exception {
  FieldDescriptor[] customerDescriptor = new FieldDescriptor[] {
    fieldWithPath("id").description("id"),
      fieldWithPath("firstName").description("客户的名字"),
      fieldWithPath("lastName").description("客户的姓")
  };

  this.mockMvc.perform(get("/customers/{id}", 1).accept(MediaType.APPLICATION_JSON))
    .andExpect(status().isOk())
    .andDo(print()) // 打印请求结果
    .andExpect(content().json("\n" +
                              "{\"id\":1,\"firstName\":\"比尔\",\"lastName\":\"盖茨\"}")) // 验证结果是否正确
    .andDo(document("customers/get",
                    pathParameters(
                      parameterWithName("id").description("客户id")
                    ),
                    responseFields(customerDescriptor)
                   )
          );
}
```

[文档在这里](http://docs.spring.io/spring-restdocs/docs/current/reference/html5/#documenting-your-api-path-parameters)有写：

> **To make the path parameters available for documentation, the request must be built using one of the methods on `RestDocumentationRequestBuilders` rather than `MockMvcRequestBuilders`.**

我来帮大家google翻译一下：

> 要使用路径参数生成文档，必须使用``RestDocumentationRequestBuilders`构建，替换`MockMvcRequestBuilders`。

更多请参考下面：

https://spring.io/guides/gs/testing-restdocs/

https://github.com/asciidoctor/asciidoctor-gradle-plugin 插件git

http://docs.spring.io/spring-restdocs/docs/current/reference/html5/ 非常完整

https://github.com/asciidoctor/asciidoctor-gradle-examples 



## IntelliJ 快捷键

光标在接口上时：alt + enter 实现接口的方法

光标在为引入包的变量时：alt + enter 引入相应的包

ctrl + enter 自动Getter/Setter

command + shift + t 创建测试文件

alt + cmmand + o 查找方法或者类

shift 两次打开指定文件

command + > 收起代码片段

参考：

http://www.tiantianbianma.com/intellij-idea-keyshot-all.html/

http://www.ituring.com.cn/article/37792

