# 创建服务消费者（Feign）

## 概述

Feign 是一个声明式的伪 Http 客户端，它使得写 Http 客户端变得更简单。使用 Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用 Feign 注解和 JAX-RS 注解。Feign 支持可插拔的编码器和解码器。Feign 默认集成了 Ribbon，并和 Eureka 结合，默认实现了负载均衡的效果

* Feign 采用的是基于接口的注解
* Feign 整合了 ribbon

## 创建服务消费者

创建一个工程名为 `hello-spring-cloud-web-admin-feign` 的服务消费者项目，`pom.xml` 配置如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.duo</groupId>
        <artifactId>hello-spring-cloud-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-web-admin-feign</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-web-admin-feign</name>
    <url>http://www.duo.com</url>
    <inceptionYear>2018-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- Spring Cloud End -->

        <!-- 解决 thymeleaf 模板引擎一定要执行严格的 html5 格式校验问题 -->
        <dependency>
            <groupId>net.sourceforge.nekohtml</groupId>
            <artifactId>nekohtml</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.duo.hello.spring.cloud.web.admin.feign.WebAdminFeignApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

主要是增加了 Feign 的依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### Application

通过 `@EnableFeignClients` 注解开启 Feign 功能

```
package com.duo.hello.spring.cloud.web.admin.feign;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class WebAdminFeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebAdminFeignApplication.class, args);
    }
}
```

### application.yml

设置程序端口号为：`8765`

```
spring:
  application:
    name: hello-spring-cloud-web-admin-feign
  thymeleaf:
    cache: false
    mode: LEGACYHTML5
    encoding: UTF-8
    servlet:
      content-type: text/html

server:
  port: 8765

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

### 创建 Feign 接口

通过 `@FeignClient("服务名")` 注解来指定调用哪个服务。代码如下：

```
package com.duo.hello.spring.cloud.web.admin.feign.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(value = "hello-spring-cloud-service-admin")
public interface AdminService {

    @RequestMapping(value = "hi", method = RequestMethod.GET)
    public String sayHi(@RequestParam(value = "message") String message);
}
```

### 创建测试用的 Controller

```
package com.duo.hello.spring.cloud.web.admin.feign.controller;

import com.duo.hello.spring.cloud.web.admin.feign.service.AdminService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AdminController {

    @Autowired
    private AdminService adminService;

    @RequestMapping(value = "hi", method = RequestMethod.GET)
    public String sayHi(@RequestParam String message) {
        return adminService.sayHi(message);
    }
}
```

### 测试访问

在浏览器上多次访问 http://localhost:8765/hi?message=HelloFeign

浏览器交替显示：

```
Hi，your message is :"HelloFeign" i am from port：8762
Hi，your message is :"HelloFeign" i am from port：8763
```

请求成功则表示我们已经成功实现了 Feign 功能来访问不同端口的实例





## feign通过拦截器携带Cookie进行远程调用



![image-20210412093344577](/Users/zeleishi/Documents/工作/授课/2101/第四阶段/day20springcloud/资料/img/image-20210412093344577.png)

### 1）通过创建拦截器

```java
package com.qf.my.product.feign.service.interceptor;

import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.apache.http.HttpEntity;
import org.apache.http.protocol.RequestContent;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.ws.rs.core.HttpHeaders;

public class FeignInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate requestTemplate) {
        //1.获取当前的request对象
        RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();
        //2.从request对象中获取cookie
        Cookie[] cookies = request.getCookies();
        String cart_token = "";
        String login_token = "";
        if(cookies!=null&&cookies.length!=0)
        for (Cookie cookie : cookies) {
            if("cart_token".equals(cookie.getName())){
                cart_token = cookie.getValue();
                continue;
            }
            if("login_token".equals(cookie.getName())){
                login_token = cookie.getValue();
                continue;
            }
        }
        //3.封装成请求头里的cookie的格式
        String cookie = "cart_token="+cart_token+";"+"login_token="+login_token;
        //4.设置cookie键值对放到请求头
        requestTemplate.header(HttpHeaders.COOKIE,cookie);
    }
}

```



### 2)在Feign注解中配置该拦截器

```java
package com.qf.my.product.feign.service.api;

import com.qf.common.entity.User;
import com.qf.my.product.feign.service.interceptor.FeignInterceptor;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;

import java.util.Objects;

//配置拦截器，让每次请求都先经过拦截器
@FeignClient(value="USER-SERVICE",configuration = FeignInterceptor.class)
public interface ProductFeignAPI {


    @RequestMapping("/user/show")
    public String userShow();

    @PostMapping("/addUser")
    public String addUser(@RequestBody User user);

    @GetMapping("/getUserByMap")
    public User getUserByMap(@RequestParam(name = "id") Long id,@RequestParam(name = "name") String name);

    @PostMapping("/addUserWithHeader")
    public String addUserWithHeader(@RequestBody User user);


}

```

