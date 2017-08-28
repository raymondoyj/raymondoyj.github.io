---
layout:     post
title:      "gradle构建Java web应用"
date:       2017-08-28 12:00:00
author:     "Raymond"
header-img: "img/post-bg-rwd.jpg"
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


[source,groovy]
----
include::{samplescodedir}/webdemo/build.gradle[]
----
<1> Using the `war` plugin
<2> Current release version of the servlet API

The `war` plugin adds the configurations `providedCompile` and `providedRuntime`, analogous to `compile` and `runtime` in regular Java applications, to represent dependencies that are needed locally but should not be added to the generated `webdemo.war` file.

The `plugins` syntax is used to apply the `java` and `war` plugins. No version is needed for either, since they are included with the Gradle distribution.

It is a good practice to generate a Gradle wrapper for the project by executing the `wrapper` task:

[listing.terminal,subs="attributes"]
----
$ gradle wrapper --gradle-version={gradle-version}
:wrapper
----

This will produce `gradlew` and `gradlew.bat` scripts and the `gradle` folder with the wrapper jar inside as described in the {user-manual}gradle_wrapper.html[wrapper section] of the User Manual.

NOTE: If you are using Gradle 4.0 or later you may see less output from the console that you might see in this guide. In this guide, output is shown using the `--console-plain` flag on the command-line. This is done to show the tasks that Gradle is executing.

== Add a servlet and metadata to the project

There are two options for defining web application metadata. Prior to version 3.0 of the servlet specification, metadata resided in a deployment descriptor called `web.xml` in the `WEB-INF` folder of the project. Since 3.0, the metadata can be defined using annotations.

Create a package folder `org/gradle/demo` below the `src/main/java` folder. Add a servlet file `HelloServlet.java`, with the following contents:

.src/main/java/com/gradle/demo/HelloServlet.java
[source,java]
----
include::{samplescodedir}/webdemo/src/main/java/com/gradle/demo/HelloServlet.java[]
----
<1> Annotation-based servlet
<2> GET request returns a simple string
<3> POST request forwards to a JSP page

The servlet uses the `@WebServlet` annotation for configuration. The `doGet` method responds to HTTP GET requests by writing a "Hello, World!" string to the output writer. It reacts to HTTP POST requests by looking for a request parameter called `name` and adding it to the `request` as an attribute called `user`, then forwarding to a `response.jsp` page.

NOTE: The `war` plugin supports the use of the older `web.xml` deployment descriptor, which by default should reside in the `WEB-INF` folder under `src/main/webapp`. Feel free to use that as an alternative to the annotation-based approach.

You now have a simple servlet that responds to HTTP GET and POST requests.

== Add JSP pages to the demo application

Add an index page to the root of the application by creating the file `index.html` in the `src/main/webapp` folder, with the following contents:

.src/main/webapp/index.html
[source,html]
----
include::{samplescodedir}/webdemo/src/main/webapp/index.html[]
----
<1> Link submits GET request
<2> Form uses POST request

The `index.html` page uses a link to submit an HTTP GET request to the servlet, and a form to submit an HTTP POST request. The form contains a text field called `name`, which is accessed by the servlet in its `doPost` method.

In its `doPost` method, the servlet forwards control to another JSP page called `response.jsp`. Therefore define a file of that name inside `src/main/webapp` with the following contents:

.src/main/webapp/response.jsp
[source,html]
----
include::{samplescodedir}/webdemo/src/main/webapp/response.jsp[]
----

The `response` page accessed the `user` variable from the request and renders it inside an `h2` tag.

== Add the `gretty` plugin and run the app

The `gretty` plugin is an outstanding community-supported plugin that can be found in the Gradle plugin repository at `https://plugins.gradle.org/plugin/org.akhikhl.gretty`. The plugin makes it easy to run or test webapps on either Jetty or Tomcat.

Add it to our project by adding the following line to the `plugins` block inside `build.gradle`.

.Updating `build.gradle` to add `gretty`
[source,groovy]
----
include::{samplescodedir}/build-2.gradle[tags=add-gretty]
----
<1> Adding the `gretty` plugin

The `gretty` plugin adds a large number of tasks to the application, useful for running or testing in Jetty or Tomcat environments. Now you can build and deploy the app to the default (Jetty) container by using the `appRun` task.

.Executing the `appRun` task
----
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
----

You can now access the web app at ++http://localhost:8080/webdemo++ and either click on the link to execute a GET request or submit the form to execute a POST request.

Although the outputs says `Press any key to stop the server, standard input is intercepted by Gradle. To stop the process press `ctrl-C`.

== Unit test the servlet using Mockito

The open source http://site.mockito.org/[Mockito framework] makes it easy to unit test Java applications. Add the Mockito dependency to the `build.gradle` file under the `testCompile` configuration.

.Adding the Mockito library to `build.gradle`
[source,groovy]
----
include::{samplescodedir}/build-2.gradle[tags=add-mockito]
----
<1> Adding Mockito

To unit test the servlet, create a package folder `org.gradle.demo` beneath `src/test/java`. Add a test class file `HelloServletTest.java` with the following contents:

.src/test/java/org/gradle/demo/HelloServletTest.java
[source,java]
----
include::{samplescodedir}/webdemo/src/test/java/org/gradle/demo/HelloServletTest.java[]
----

The test creates mock objects for the `HttpServletRequest`, `HttpServletResponse`, and `RequestDispatcher` classes. For the `doGet` test, a `PrintWriter` that uses a `StringWriter` is created, and the mock request object is configured to return it when the `getWriter` method is invoked. After calling the `doGet` method, the test checks that the returned string is correct.

For the post requests, the mock request is configured to return a given name if present or null otherwise, and the `getRequestDispatcher` method returns the associated mock object. Calling the `doPost` method executes the request. Mockito then verifies that the `setAttribute` method was invoked on the mock response with the proper arguments and that the `forward` method was called on the request dispatcher.

You can now test the servlet using Gradle with the `test` task (or any task, like `build`, that depends on it).

----
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
----

The test output can be accessed from `build/reports/tests/test/index.html` in the usual manner. You should get a result similar to:

image::test-results.png[]

== Add a functional test

The `gretty` plugin combines with Gradle to make it easy to add functional tests to web applications. To do so, add the following lines to your `build.gradle` file:

.Gretty additions to `build.gradle` for functional testing
[source,groovy]
----
include::{samplescodedir}/build-3.gradle[tags=configure-gretty]

// ... rest from before ...

include::{samplescodedir}/build-3.gradle[tags=add-selenium]
----
<1> Tell gretty to start and stop the server on test
<2> Automatically installs browser drivers
<3> Uses Selenium for functional tests

The `gretty` plugin needs to know which task requires a start and stop of the server. Frequently that is assigned to your own task, but to keep things simple just use the existing `test` task.

http://www.seleniumhq.org[Selenium] is a popular open-source API for writing functional tests. Version 2.0 is based on the WebDriver API. Recent versions require testers to download and install a version of WebDriver for their browser, which can be tedious and hard to automate. The https://github.com/bonigarcia/webdrivermanager[WebDriverManager] project makes it easy to let Gradle handle that process for you.

Add the following functional test to your project, in the `src/test/java` directory:

.src/test/java/org/gradle/demo/HelloServletFunctionalTest.java
[source,java]
----
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
        ChromeDriverManager.getInstance().setup(); // <1>
    }

    @Before
    public void setUp() {
        driver = new ChromeDriver();               // <2>
    }

    @After
    public void tearDown() {
        if (driver != null)
            driver.quit();                         // <3>
    }

    @Test
    public void sayHello() throws Exception {      // <4>
        driver.get("http://localhost:8080/webdemo");

        driver.findElement(By.id("say-hello-text-input")).sendKeys("Dolly");
        driver.findElement(By.id("say-hello-button")).click();

        assertEquals("Hello Page", driver.getTitle());
        assertEquals("Hello, Dolly!", driver.findElement(By.tagName("h2")).getText());
    }
}
----
<1> Downloads and installs browser driver, if necessary
<2> Start the browser automation
<3> Shut down the browser when done
<4> Run the functional test using the Selenium API

The WebDriverManager portion of this test checks for the latest version of the binary, and downloads and installs it when it is not present. Then the `sayHello` test method drives a Chrome browser to the root of our application, fills in the input text field, clicks the button, and verifies the title of the destination page and that the `h2` tag contains the expected string.

The WebDriverManager system supports Chrome, Opera, Internet Explorer, Microsoft Edge, PhantomJS, and Firefox. Check the project documentation for more details.

== Run the functional test

Run the test using the `test` task:

----
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
----

The `gretty` plugin starts up an embedded version of Jetty 9 on the default port, executes the tests, and shuts down the server. If you watch, you'll see the Selenium system open a new browser, access the site, complete the form, click the button, check the new page, and finally shut down the browser.

Integration tests are often handled by creating a separate source set and dedicated tasks, but that is beyond the scope of this guide. See the http://akhikhl.github.io/gretty-doc/[Gretty documentation] for details.

== Summary

In this guide, you learned how to:

* Use the `war` plugin in Gradle builds to define a web application
* Add a servlet and JSP pages to a web app
* Use the `gretty` plugin to deploy the application
* Unit test a servlet using the Mockito framework
* Functionally test the web app using `gretty` and Selenium

== Next steps

Gretty is a very powerful API. See the http://akhikhl.github.io/gretty-doc/[Gretty documentation] for details. Further details about Selenium can be found on the http://www.seleniumhq.org[Selenium website], and more about WebDriverManager system is available on the https://github.com/bonigarcia/webdrivermanager[WebdriverDriverManager GitHub repository].

If you are interested in functional testing, check out the open source http://www.gebish.org[Geb] library, which provides a powerful Groovy DSL for browser automation that rests on top of Selenium and WebDriver.


include::contribute[]
