---
title: "初学 SpringBoot"
linktitle: "初学 SpringBoot"
date: 2022-05-05T14:51:48+08:00
tags: ['SpringBoot', 'Spring', 'JSON', 'HTTP']
---

## 从后端零基础开始

由于本学期 J2EE 课程作业需要，我们小组需要前后端分离去学习不同的东西。前端关于 `HTML`,`CSS`,`JSP` 的内容已经有些许了解了，但是后端部分的知识却是一点都不知道。出于想学习新东西的心态，我便选择开始学习后端部分的知识。  
  
可能是觉得老师教的东西有些许过时了，我们的组长便决定不完全按照老师所教的东西来，而是选择了 `SpringBoot` 和  `MyBatis` 作为我们的后端选型，数据库也不是 `MYSQL`，而是 `MariaDB` （我们组长到底是怎么知道这么多东西的真的好厉害必须狠狠夸一波）。  
  
于是便有了后面一些关于 SpringBoot 的简单学习。  
  
## [SpringBoot](https://spring.io/) 是什么？  

要弄懂 SpringBoot 是什么或许得先了解 Spring 应用开发的相关知识。简单来说，它是一种为了解决企业应用程序开发复杂性而创建的开源的框架。框架的主要优势之一就是其分层架构，分层架构允许我们使用哪一种组件，同时为 J2EE 应用程序开发提供了集成的框架。  
  
而 SpringBoot 呢？它也是一种框架。它被设计出来的目的便是**为了简化 Spring 应用的创建，运行，调试，部署等功能**。它打包了许多本来需要我们自己去配置的功能，按照人们的使用习惯解决了依赖问题，这样便可以降低开发人员对于框架的关注点，可以把更多的精力放在自己的业务代码上。总的来说，其目的就是为了 Java Web 的开发进行"简化"和"加快速度"，简化开发过程中引入或启动相关 Spring 功能的配置。使用 SpringBoot 我们就可以不用或者只需很少的 Spring 配置就可以让企业项快速运行起来。

## 简单尝试

打开 IDEA，Generator 选择的是 Spring Initializr，取名默认为 demo，再选择了 `Gradle` 作为此次的项目结构便开始了我的第一次尝试（网上大部分的教程好像都是使用了 MAVEN，但是据说是使用 Gradle 的人越来越多了，便试着使用了它）。这里要注意在第二步中需要勾选 Spring Web 才能顺利进行这个框架的构建~  
  
之后便是愉快的贴代码环节了。通过 `src/main/java/com/example/demo` 路径打开已生成的 `DemoApplication.java` 文件，并将下面这段代码直接贴上去：

```java
package com.example.demo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
              
@SpringBootApplication
@RestController
public class DemoApplication {
                               
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
                  
    @GetMapping("/hello")
    public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
        return String.format("Hello %s!", name);
    }
                
}          
```

在代码中，自定义的`hello()`函数表示其中被定义的内容会以字符串的形式被投射在 web 上。 以`@`符号开头的我们将其称为注释，它会告诉 Spring 框架这里面隐含了一大段代码，他们也分别有各自的作用：

- `@RestController`：表示接下来的代码的内容将会被投放到 web 上。
- `@GetMapping("/hello")`：将我们定义的`hello()`函数发送到我们之后将访问的端口的页面上。
- `@RequestParam`：将 Sting 的值设为`name`，默认值为`World`。  

最后通过 main 函数执行程序之后我们可以访问本机中的`http://localhost:8080/hello`地址，便可以看到我们预期之中的结果了！  
  
当然，前面我们也说了这里的 name 只是个默认值，所以我们如果要进行修改的话，可以在地址后面增加 `?name=GuanGuan`，这样就会得到另外你想要的结果了。

## 进阶

接下来试试创建一个 RESTful Web Service。  
这里先将访问端口摆出来：`http://localhost:8080/greeting`.  
展示出以下的 JSON 格式是我们的最终目的。

```json
{"id":1,"content":"Hello, World!"}
```

因为我们需要获得这种格式，所以我们需要创建一个可以表示这种资源格式的实体类而去建立它的模型。首先，我们需要`id`（ greeting 的次数）和 `content`（greeting 的文本方式）这两个数据，其次，我们还需要构造函数，以及获取他们的值的方式。该类的定义如下，我将其命名为 Greeting 后存在了`src/main/java/com/example/restservice/Greeting.java`路径下。这个实体类可以被序列化成我们想要的 JSON 格式。

```java
package com.example.restservice;

public class Greeting {

 private final long id;
 private final String content;

 public Greeting(long id, String content) {
  this.id = id;
  this.content = content;
 }

 public long getId() {
  return id;
 }

 public String getContent() {
  return content;
 }
}
```

由于在 Spring 构建 RESTful web 服务的方法中，HTTP 的请求通常由控制器处理，因此我们还需要创建一个控制器。它和我们之前编写的 hello() 其实十分相像：

```java
package com.example.restservice;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

 private static final String template = "Hello, %s!";
 private final AtomicLong counter = new AtomicLong();

 @GetMapping("/greeting")
 public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
  return new Greeting(counter.incrementAndGet(), String.format(template, name));
 }
}
```

代码中以`@`开头的注释和之前的意思是一样的。这段代码和之前的区别只有两个，一个我们使用了`AtomicLong`类定义了一个`counter`，而用它去记录我们的`id`的数值，除此之外，`greeting()`函数返回的是我们自己创造的`Greeting`类型。

### 好处

它和传统 MVC 的区别在于 **创建 HTTP 响应体的方式**

- RESTful Web 服务控制器填充并返回一个 Greeting 对象，并将对象数据作为 JSON 直接写入 HTTP 相应
- 传统 MVC 依赖于试图转换，组装成 HTML 的服务器端呈现
- RESTful Web 服务的每个方法均返回`领域对象`而不是`视图`。  
  
最后依旧是通过访问`http://localhost:8080/greeting`得到被返回的 JSON 字符串结果。至于为什么通过创建这个实体类就可以得到 JSON 的格式，是因为由于 Spring 的 HTTP 消息转换器，我们无需手动进行这个操作去进行转换。因为 Jackson 2 位于类路径上，所以会自动选择 Spring 的`MappingJackson2HttpMessageConvert`将`Greeting`转化为 JSON。

### 拓展（Build an executable JAR）

SpringBoot 有很棒的一点就是我们可以通过将其打包成一个`jar`文件而后直接运行，这样便可以变得十分简洁，且方便我们使用。它需要包含所有的依赖项，类和资源。而这样生产的可执行的 jar 可以轻松地将一个服务器作为应用程序进行发布，版本化和部署。
