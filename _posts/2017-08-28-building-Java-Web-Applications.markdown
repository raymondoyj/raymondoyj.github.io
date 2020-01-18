---
layout:     post
title:      "gradle构建Java web应用"
date:       2017-08-28 12:00:00
author:     "Raymond"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - gradle
---

> 这篇文章译自[Building Java Web Applications](https://guides.gradle.org/building-java-web-applications/)

## 介绍

Gradle有一个用于构建Java web应用的`war`插件，并且社区提供了一个叫`gretty`的优秀插件，用于在Jetty或Tomcat上测试和部署Web应用。本文演示如何构建一个简单的Web应用并将其部署在Jetty上。您还将学习如何使用Mockito框架为servlet编写单元测试，以及如何使用gretty和Selenium为Web应用编写功能测试。

## 构建什么

你将用gradle默认的项目结构创建一个web应用，添加Servlet API的依赖，添加`gretty`插件，然后构建和测试应用。

## 需要什么

* 大约30分钟
* 文本编辑器或IDE
* Java7或以上
* [Gradle](https://gradle.org/install)4.0或以上

## 创建web应用的目录结构

Gradle的`war`插件记录在[web应用快速入门](https://docs.gradle.org/4.0/userguide/web_project_tutorial.html)和用户手册的[WAR插件章节](https://docs.gradle.org/4.0/userguide/war_plugin.html)中。`war`插件扩展了Java插件，增加对web应用的支持。默认情况下，它用`src/main/webapp`文件夹存放web相关的资源。

因此，新建`webdemo`项目，并创建以下目录结构：  

_示例项目的目录结构_

```
webdemo/
    src/
        main/
            java/
            webapp/
        test
            java/
```

任何servlets或其他Java文件放在`src/main/java`，测试的代码放在`src/test/java`，而其他web资源放在`src/main/webapp`。

## 添加Gradle构建文件

在项目的根目录创建`build.gradle`文件，其内容如下：

_build.gradle_


```groovy
plugins {
    id 'java'
    id 'war'
}

repositories {
    jcenter()
}

dependencies {
    providedCompile 'javax.servlet:javax.servlet-api:3.1.0'
    testCompile 'junit:junit:4.12'
}
```

`war`插件添加了configurations组的`providedCompile`和`providedRuntime`插件，类似于常见Java应用程序中的`compile`和`runtime`，来表示这些依赖在本地需要但不添加到生成的`webdemo.war`文件里。  
这些插件的语法是用在`java`和`war`插件中。不需要版本，因为它们已经包含在Gradle的发布版里。

建议通过执行`wrapper`任务生成Gradle包装：

```shell
$ gradle wrapper --gradle-version=4.0
:wrapper
```

这会产生`gradlew`、`gradlew.bat`脚本和包含包装器的jar的`gradle`文件夹，详细查看用户手册的[`wrapper`章节](https://docs.gradle.org/4.0/userguide/gradle_wrapper.html)。

> 如果你使用的是Gradle4.0或以上，则你可能看到比本文更少的控制台输出。本文使用`--console-plain`命令行参数来调整输出信息。这是为了显示Gradle正在执行的任务。

## 向项目添加servlet和元数据

定义Web应用的元数据有两个方式。在servlet3.0规范之前，元数据在项目的`WEB-INF/web.xml`中描述。3.0之后，元数据可以用注解来定义。

在src/main/java文件夹下创建一个包文件夹org/gradle/demo。添加一个servlet文件HelloServlet.java，内容如下：

_src/main/java/com/gradle/demo/HelloServlet.java_

```java
package org.gradle.demo;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "HelloServlet", urlPatterns = {"hello"}, loadOnStartup = 1) 
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
        response.getWriter().print("Hello, World!");  
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
        String name = request.getParameter("name");
        if (name == null) name = "World";
        request.setAttribute("user", name);
        request.getRequestDispatcher("response.jsp").forward(request, response); 
    }
}
```

该servlet使用`@WebServlet`注释进行配置。doGet方法通过将“Hello，World！”字符串输出到输出字符流来响应HTTP GET请求。它通过请求参数name作为新的请求参数user的值，然后转发到response.jsp页面来响应HTTP POST请求。

提示：`war`插件支持使用较旧的`web.xml`，默认放在`src/main/webapp`下的`WEB-INF`文件夹中。可以使用它替代基于注解的方法。

你现在有一个简单的servlet，来响应GET和POST请求。

## 增加JSP页面到demo

在应用的根目录`src/main/webapp`下创建首页`index.html`，内容如下：

```html
<html>
<head>
  <title>Web Demo</title>
</head>
<body>
<p>Say <a href="hello">Hello</a></p> 

<form method="post" action="hello">  
  <h2>Name:</h2>
  <input type="text" id="say-hello-text-input" name="name" />
  <input type="submit" id="say-hello-button" value="Say Hello" />
</form>
</body>
</html>
```

`index.html`用一个链接提交HTTP GET请求，用表单提交HTTP POST请求。表单的`name`文本字段就是`doPost`接受的参数。

在`doPost`方法里，servlet将请求转发到另一个JSP页面`response.jsp`。所以要在`src/main/webapp`里定义那个文件，内容如下：

_src/main/webapp/response.jsp_

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>Hello Page</title>
    </head>
    <body>
        <h2>Hello, ${user}!</h2>
    </body>
</html>
```

`response`页面从request访问 `user`参数并渲染到`h2`标签里。

## 添加`gretty`插件并运行应用

`gretty`插件是一款很出色的社区支持的插件，可以再Gradle仓库`https://plugins.gradle.org/plugin/org.akhikhl.gretty`找到它。它使得很容易地用Jetty或Tomcat运行或测试web应用。

要将它加入到我们的项目，需要将下面几行代码加入到`build.gradle`的`plugins`的模块里。

```groovy
plugins {
    id 'java'
    id 'war'
    id 'org.akhikhl.gretty' version '1.4.2' 
}
```

`gretty`插件往应用加入了很多任务，对于在Jetty或Tomcat环境中运行或测试非常有用。现在你可以用`appRun`任务构建和部署应用到默认的容器(Jetty)。

```shell
$ ./gradlew appRun
:prepareInplaceWebAppFolder
:createInplaceWebAppFolder UP-TO-DATE
:compileJava
:processResources UP-TO-DATE
:classes
:prepareInplaceWebAppClasses
:prepareInplaceWebApp
:appRun
12:25:13 INFO  Jetty 9.2.15.v20160210 started and listening on port 8080
12:25:13 INFO  webdemo runs at:
12:25:13 INFO    http://localhost:8080/webdemo
Press any key to stop the server.
> Building 87% > :appRun

BUILD SUCCESSFUL
```

你现在可以在__http://localhost:8080/webdemo__访问web应用或者点击链接执行GET请求或者提交表单执行POST请求。

虽然输出的信息说`Press any key to stop the server`，但是通常用`ctrl-C`中断停止进程。

## 使用Mockito对servlet进行单元测试

开源的[Mockito framework](http://site.mockito.org/)可以很容易地对Java应用程序进行单元测试。将Mockito的依赖项添加到`build.gradle`的`testCompile`配置里。

_添加Mockito包到`build.gradle`_

```
dependencies {
    providedCompile 'javax.servlet:javax.servlet-api:3.1.0'
    testCompile 'junit:junit:4.12'
    testCompile 'org.mockito:mockito-core:2.7.19'  
}
```

要对servlet进行单元测试，先要在`src/test/java`下创建一个包目录`org.gradle.demo`，并且创建测试类`HelloServletTest.java`，内容如下：

```java
package org.gradle.demo;

import org.junit.Before;
import org.junit.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import javax.servlet.RequestDispatcher;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;
import java.io.StringWriter;

import static org.junit.Assert.assertEquals;
import static org.mockito.Mockito.*;

public class HelloServletTest {
    @Mock private HttpServletRequest request;
    @Mock private HttpServletResponse response;
    @Mock private RequestDispatcher requestDispatcher;

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void doGet() throws Exception {
        StringWriter stringWriter = new StringWriter();
        PrintWriter printWriter = new PrintWriter(stringWriter);

        when(response.getWriter()).thenReturn(printWriter);

        new HelloServlet().doGet(request, response);

        assertEquals("Hello, World!", stringWriter.toString());
    }

    @Test
    public void doPostWithoutName() throws Exception {
        when(request.getRequestDispatcher("response.jsp"))
            .thenReturn(requestDispatcher);

        new HelloServlet().doPost(request, response);

        verify(request).setAttribute("user", "World");
        verify(requestDispatcher).forward(request,response);
    }

    @Test
    public void doPostWithName() throws Exception {
        when(request.getParameter("name")).thenReturn("Dolly");
        when(request.getRequestDispatcher("response.jsp"))
            .thenReturn(requestDispatcher);

        new HelloServlet().doPost(request, response);

        verify(request).setAttribute("user", "Dolly");
        verify(requestDispatcher).forward(request,response);
    }
}
```

这次测试创建了`HttpServletRequest`、`HttpServletResponse`和`RequestDispatcher`类的模拟对象。在`doGet`测试里，用`StringWriter`创建出`PrintWriter`，当`getWriter`被执行，模拟的请求对象则会返回。在调用`doGet`方法后，测试将会检查返回的字符串是否正确。

对于post请求，模拟请求被设置伟返回一个指定的名字，否则返回null。`getRequestDispatcher`方法则返回关联的模拟对象。调用`doPost`方法并执行请求。然后Mockito验证`setAttribute`方法是在模拟请求中用正确的参数调用的，并且在request dispatcher上调用了forward方法。

你现在可以用Gradle的`test`任务(或者其他依赖它的任务，例如`build`)测试servlet。

```shell
$ ./gradlew build
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:war
:assemble
:compileTestJava
:processTestResources UP-TO-DATE
:testClasses
:test
:check
:build

BUILD SUCCESSFUL
```

测试结果可以在`build/reports/tests/test/index.html`里看。

![test-result](/img/in-post/post-java-web-test-results.png)

## 新增功能测试

`gretty`插件与Gradle结合起来，使向Web应用程序添加功能测试变得容易。为此，将以下配置添加到build.gradle文件中：

```groovy
gretty {
    integrationTestTask = 'test'  
}

// ... rest from before ...

dependencies {
    providedCompile 'javax.servlet:javax.servlet-api:3.1.0'
    testCompile 'junit:junit:4.12'
    testCompile 'org.mockito:mockito-core:2.7.19'
    testCompile 'io.github.bonigarcia:webdrivermanager:1.6.1' 
    testCompile 'org.seleniumhq.selenium:selenium-java:3.3.1' 
}
```

`gretty`插件需要知道哪个任务需要先执行，通常是你自己的任务，但为了保持简单，只使用现有的测试任务。

[Selenium](http://www.seleniumhq.org)是用于编写功能测试的流行的开源API。2.0版基于WebDriver API。最近的版本要求测试者为他们的浏览器下载和安装一个WebDriver版本，这个版本可能很乏味而且很难自动化。 [WebDriverManager](https://github.com/bonigarcia/webdrivermanager)项目可以让Gradle轻松地为你处理这个过程。

添加功能测试到你的项目，并在`src/test/java`目录下：

_src/test/java/org/gradle/demo/HelloServletFunctionalTest.java_
```java
package org.gradle.demo;

import io.github.bonigarcia.wdm.ChromeDriverManager;
import org.junit.After;
import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

import static org.junit.Assert.assertEquals;

public class HelloServletFunctionalTest {
    private WebDriver driver;

    @BeforeClass
    public static void setupClass() {
        ChromeDriverManager.getInstance().setup(); 
    }

    @Before
    public void setUp() {
        driver = new ChromeDriver();               
    }

    @After
    public void tearDown() {
        if (driver != null)
            driver.quit();                         
    }

    @Test
    public void sayHello() throws Exception {
        driver.get("http://localhost:8080/webdemo");

        driver.findElement(By.id("say-hello-text-input")).sendKeys("Dolly");
        driver.findElement(By.id("say-hello-button")).click();

        assertEquals("Hello Page", driver.getTitle());
        assertEquals("Hello, Dolly!", driver.findElement(By.tagName("h2")).getText());
    }
}
```

`WebDriverManager`会先检查最新的版本，如果版本不存在则下载并安装。然后`sayHello`测试方法会用Chrome访问应用的根路径，填写文本字段，点击按钮，并验证目标页面的标题，并且h2标签包含期望的字符串。

`WebDriverManager`系统支持Chrome, Opera, Internet Explorer, Microsoft Edge, PhantomJS, and Firefox。查看文档获取更多信息。

## 运行功能测试

```shell
$ ./gradlew test
:prepareInplaceWebAppFolder UP-TO-DATE
:createInplaceWebAppFolder UP-TO-DATE
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:prepareInplaceWebAppClasses UP-TO-DATE
:prepareInplaceWebApp UP-TO-DATE
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:appBeforeIntegrationTest
12:57:56 INFO  Jetty 9.2.15.v20160210 started and listening on port 8080
12:57:56 INFO  webdemo runs at:
12:57:56 INFO    http://localhost:8080/webdemo
:test
:appAfterIntegrationTest
Server stopped.

BUILD SUCCESSFUL
```

`gretty`插件在默认端口启动一个嵌入式版本的Jetty 9，执行测试并关闭服务器。如果你在看，你会看到Selenium系统打开一个新的浏览器，访问网站，填写表格，点击按钮，检查新的页面，最后关闭浏览器。

集成测试通常是通过创建一个独立的源代码集和专用任务来处理的，但这不在本指南的范围之内。有关详细信息，请参阅[Gretty文档](http://akhikhl.github.io/gretty-doc/)。

## 总结

在本指南中，你学习了如何：

* 在Gradle构建中使用`war`插件来定义一个web应用程序
* 将一个servlet和JSP页面添加到一个Web应用程序
* 使用`gretty`插件来部署应用程序
* 用`Mockito`框架单元测试servlet
* 使用`gretty`和`Selenium`功能测试Web应用程序

## 下一步

`Gretty`是非常强大的API. 更多信息请查阅[[Gretty documentation]](http://akhikhl.github.io/gretty-doc/)。有关Selenium的更多详细信息可以在[Selenium website](http://www.seleniumhq.org)上找到，有关WebDriverManager系统的更多信息可以在[[WebdriverDriverManager GitHub repository]](https://github.com/bonigarcia/webdrivermanager)上找到。

如果您对功能测试感兴趣，请查看开源的[[Geb](http://www.gebish.org)，它提供了一个强大的Groovy DSL，用于浏览器自动化，位于Selenium和WebDriver之上。
