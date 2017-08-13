---
layout:     post
title:      "使用mybatis+SQLServer做持久层入门"
subtitle:   "本篇文章介绍如何用mybatis连接SQLServer数据库"
date:       2015-04-02 12:00:00
author:     "Raymond"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - 工作
    - mybatis
    - sqlserver
---

> 本篇文章介绍如何用mybatis连接SQLServer数据库。


1、在http://www.microsoft.com/en-us/server-cloud/products/sql-server-editions/sql-server-express.aspx 下载SQL Server 2014 Express 免费版，用于学习用。

然后，安装并配置好默认管理员sa的密码。

2、在Sql Server Configuration Manager里面，将TCP/IP启用，并重启SQLServer。然后我们可以使用一些数据库客户端尝试连接数据库。
![TCP/IP启用][1]
![客户端尝试连接][2]

3、在http://www.microsoft.com/zh-CN/download/details.aspx?id=11774 下载SQLServer 的jdbc驱动。里面有简单的例子和三个版本的jar包。

![jdbc驱动][3]

4、现在我拿到了一个spring+springmvc+mybatis的项目，之前这个项目使用的数据库是mysql，现在我要换成SQLServer。
首先我将其中一个版本的jdbc.jar包引进项目中，这里我使用的是sqljdbc4.jar。
（调试中出现
 Cannot load JDBC driver class 'com.microsoft.sqlserver.jdbc.SQLServerDriver'
或Cannot create PoolableConnectionFactory (不支持此服务器版本。目标服务器必须是 SQL Server 2000 或更高版本。
等异常，那可能就是使用的jar包版本不对）


然后修改jdbc.properties:

    driver=com.microsoft.sqlserver.jdbc.SQLServerDriver
    url=jdbc\:sqlserver\://127.0.0.1\:1433;DatabaseName=bill
    user=sa
    password=

修改mybatis-config.xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <properties>
            <property name="dialect" value="sqlserver"/>
        </properties>
    </configuration>

修改这两个文件就基本配好了，接着用junit调试一下。

在AccountMapper.xml里，写入

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="net.bill.modules.dao.AccountMapper">

    <insert id="insert" parameterType="Account">
    INSERT INTO b_account
    	(
    	userId,
    	password
    	)
    VALUES
    	(
    	#{userId},
    	#{password}
    	)
    </insert>

    </mapper>


并建立相应的pojo和数据库表。

在Test.java里写以下调试代码：

    package net.test.modules;
    import javax.annotation.Resource;

    import net.bill.modules.dao.AccountMapper;
    import net.bill.modules.pojo.Account;

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.BeansException;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ApplicationContextAware;
    import org.springframework.test.context.ContextConfiguration;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations="/applicationContext.xml")
    public class BaseTest implements ApplicationContextAware{
    	public ApplicationContext ctxt;

    	public void setApplicationContext(ApplicationContext arg0)
    			throws BeansException {
    		this.ctxt = arg0;
    	}

    	[@Resource](https://my.oschina.net/u/929718)
    	private AccountMapper accountMapper;

    	[@Test](https://my.oschina.net/azibug)
    	public void testSql(){
    		accountMapper.insert(new Account("16", "haha"));
    	}
    }


**测试通过了。**

![测试成功][4]


  [1]: http://static.oschina.net/uploads/space/2015/0402/171628_bOL9_550406.png
  [2]: http://static.oschina.net/uploads/space/2015/0402/171902_KZOE_550406.png
  [3]: http://static.oschina.net/uploads/space/2015/0402/172030_YeYT_550406.png
  [4]: http://static.oschina.net/uploads/space/2015/0402/172945_cdGG_550406.png
