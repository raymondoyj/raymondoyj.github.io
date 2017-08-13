---
layout:     post
title:      "在linux准备Java开发环境"
date:       2017-08-10 11:41:18
author:     "Raymond"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 工作
    - linux
    - Java
---

> 这套Java开发环境包括**JDK**、项目管理工具-**gradle**、IDE-**idea**。  
> 以前准备方式都是在官网下载安装包，手动配置环境。  
> 本文介绍如何通过**SDKMAN!**和**JetBrains Toolbox App**来管理这套环境。  

## SDKMAN!

### 什么是SDKMAN!

[SDKMAN!][SDKMAN]是用来在类Unix 系统中管理多个版本的开发环境的工具。提供命令行接口来安装、切换、删除、列出候选版本。  

### 安装SDKMAN!

打开终端并输入:  

```shell
$ curl -s "https://get.sdkman.io" | bash
```

按照屏幕上的说明完成安装。  
接下来，打开新终端或直接输入：

```shell
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

最后，运行以下代码段来确保安装成功:

```shell
$ sdk version
```

如果一切顺利，应该会显示版本，就像是:

```shell
sdkman 5.0.0+51
```

### 安装Java

打开终端并输入:  

```shell
$ sdk install java
```

运行以下代码段确认安装成功:

```shell
$ java -version
```

若成功，则屏幕显示版本信息，如下:

```shell
openjdk version "1.8.0_144"
OpenJDK Runtime Environment (Zulu 8.23.0.3-linux64) (build 1.8.0_144-b01)
OpenJDK 64-Bit Server VM (Zulu 8.23.0.3-linux64) (build 25.144-b01, mixed mode)
```

### 安装gradle

打开终端并输入:  

```shell
$ sdk install gradle
```

运行以下代码段确认安装成功:

```shell
$ gradle -v
```

若成功，则屏幕显示版本信息，如下:

```shell
------------------------------------------------------------
Gradle 3.0
------------------------------------------------------------

Build time:   2016-08-15 13:15:01 UTC
Revision:     ad76ba00f59ecb287bd3c037bd25fc3df13ca558

Groovy:       2.4.7
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_102 (Oracle Corporation 25.102-b14)
OS:           Linux 4.4.0-89-generic amd64
```

## Jetbrains Toolbox

### 什么是Jetbrains Toolbox

[Jetbrains Toolbox][Toolbox]是用来管理Jetbrains工具和项目的应用。

### 安装Jetbrains Toolbox

从[官网][Toolbox]下载Jetbrains Toolbox App并安装。

### 安装idea community

在Jetbrains Toolbox App面板上选择idea community并安装。

## 参考链接

1. [http://sdkman.io/index.html][SDKMAN]
2. [https://www.jetbrains.com/][jetbrains]
3. [https://www.jetbrains.com/toolbox/app/][Toolbox]
4. [https://gradle.org/][gradle]
5. [http://www.oracle.com/technetwork/java/javase/archive-139210.html][Java]

[SDKMAN]: http://sdkman.io/index.html  "SDKMAN!"
[jetbrains]: https://www.jetbrains.com/  "jetbrains"
[Toolbox]: https://www.jetbrains.com/toolbox/app/  "Jetbrains Toolbox"
[gradle]: https://gradle.org/ "gradle"
[Java]: http://www.oracle.com/technetwork/java/javase/archive-139210.html "Java"
