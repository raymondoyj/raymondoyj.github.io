---
layout:     post
title:      "windows+nginx+memcached+tomcat做负载均衡"
date:       2015-04-18
author:     "Raymond"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - 工作
    - windows
    - nginx
    - memcached
    - tomcat
---

> 最近项目要用到阿里云的SLB和OCS做负载均衡，为了在本地模拟并测试项目代码，我用nginx集群，memcached做session共享，来替代SLB和OCS服务。本文主要介绍如何安装和配置这个测试环境。

首先，我们明确目标，做Tomcat集群的目的是为了提供更高的负载能力，把访问均摊到不同的服务器上。

直观地来说，就是访问test.localhost.com时，nignx会随机将访问请求分发到tomcat1,tomcat2,为了保持session同步，使用memcached去管理session。

为此我们准备的配置清单是： windows x 1    nginx x 1    memcached x 1    tomcat x 2   mysql x 1

部署的架构图如下：

![架构图][1]

首先，我准备了一个Java Web项目、tomcat 6、jdk6。
###Step1  配置tomcat###
先将项目部署到tomcat，因为要用到两个tomcat，当然地要把其中一个tomcat的端口修改一下。  
###Step2  配置nginx###
下载安装nginx，[http://kevinworthington.com/nginx-for-windows/][2]。

在host里准备一个测试域名。打开C:\Windows\System32\drivers\etc\host, 添加域名映射  

        127.0.0.1    test.local.com

打开nginx的根目录，在conf里添加test.conf，内容大概如下。  

        upstream  test.local.com  {  
                      server   127.0.0.1:8080 weight=1;  
                      server   127.0.0.1:8083 weight=1;  
            }
        server {
                listen       80;
         	server_name test.local.com;

                error_page   500 502 503 504  /50x.html;
                location = /50x.html {
                    root   html;
                }

        root /data/projects/ycp/bill;

            # - rewrite: if( path ~ "^/assets/(.*)" ) goto "/public/assets/$1"
        #    location ~ ^/static/assets/(.*)$
        #    {
        #alias /data/projects/payment/web/public/assets/$1;
        #      access_log off;
        #      #expires 3d;
        #    }

        location / {
                    index  index.html index.htm  index.jsp;
                }

            location ~ .* {
        # proxy_pass_header Server;
         proxy_set_header Host $http_host;
        # proxy_redirect off;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Scheme $scheme;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        add_header Pragma "no-cache";
        proxy_pass http://test.local.com;
            }


            rewrite ^/admin/?$ /admin/login redirect;

            # for rewrite
            rewrite ^/(channel|admin|mobile|api|web)/(.*)$ /public/index.php/$2 last;

            #redirect to mobile wap
        #rewrite ^$ /m redirect;
        #rewrite ^/$ /mobile/user redirect;



        }

然后将test.conf引入到nginx.conf里面  

        include ycp-test.conf;  

此时，我们可以启动nginx和tomcat，访问[http://test.local.com][3]，试试效果。  
###Step3 配置和调试memcached###
上一步，我们配好了nginx集群，但是tomcat的session还没有集到一起。接下来我们会用memcached管理session。  
下载安装memcached。[http://blog.couchbase.com/memcached-windows-64-bit-pre-release-available][4]   
[点击这里直接下载][5]  
ps: [memcached官网][6]  
解压放某个盘下面，比如在c:\memcached  
在CMD下输入 "c:\memcached\memcached.exe -d install" 安装。  
再输入："c:\memcached\memcached.exe -d start" 启动。NOTE: 以后memcached将作为windows的一个服务每次开机时自动启动。  
如图：  
![memcached服务][7]  
>启动时可添加其他参数  
>-p 监听的端口  
>-l 连接的IP地址, 默认是本机   
>-d start 启动memcached服务  
>-d restart 重起memcached服务  
>-d stop|shutdown 关闭正在运行的memcached服务  
>-d install 安装memcached服务  
>-d uninstall 卸载memcached服务  
>-u 以的身份运行 (仅在以root运行的时候有效)  
>-m 最大内存使用，单位MB。默认64MB  
>-M 内存耗尽时返回错误，而不是删除项  
>-c 最大同时连接数，默认是1024  
>-f 块大小增长因子，默认是1.25   
>-n 最小分配空间，key+value+flags默认是48   
>-h 显示帮助  

另外，我们可以用telnet操作memcached  
windows如果本来没有telnet命令的话，可以在控制面板-程序和功能-启动或关闭Windows功能里面勾选telnet客户端  
![telnet客户端][8]  

        telnet 127.0.0.1 11211  
        stats

可得到描述Memcached服务器运行情况的参数。如下图：  
![stats参数][9]  
ps:网上给出的一些参数解释
>1.  pid: memcached服务进程的进程ID
2.  uptime: memcached服务从启动到当前所经过的时间，单位是秒。
3.  time: memcached服务器所在主机当前系统的时间，单位是秒。
4.  version: memcached组件的版本。这里是我当前使用的1.2.6。
5.  pointer_size：服务器所在主机操作系统的指针大小，一般为32或64.
6.  curr_items：表示当前缓存中存放的所有缓存对象的数量。不包括目前已经从缓存中删除的对象。
7.  total_items：表示从memcached服务启动到当前时间，系统存储过的所有对象的数量，包括目前已经从缓存中删除的对象。
8.  bytes：表示系统存储缓存对象所使用的存储空间，单位为字节。
9.  curr_connections：表示当前系统打开的连接数。
10. total_connections：表示从memcached服务启动到当前时间，系统打开过的连接的总数。
11. connection_structures：表示从memcached服务启动到当前时间，被服务器分配的连接结构的数量，这个解释是协议文档给的，具体什么意思，我目前还没搞明白。
12. cmd_get：累积获取数据的数量，这里是3，因为我测试过3次，第一次因为没有序列化对象，所以获取数据失败，是null，后边有2次是我用不同对象测试了2次。
13. cmd_set：累积保存数据的树立数量，这里是2.虽然我存储了3次，但是第一次因为没有序列化，所以没有保存到缓存，也就没有记录。
14. get_hits：表示获取数据成功的次数。
15. get_misses：表示获取数据失败的次数。
16. evictions：为了给新的数据项目释放空间，从缓存移除的缓存对象的数目。比如超过缓存大小时根据LRU算法移除的对象，以及过期的对象。
17. bytes_read：memcached服务器从网络读取的总的字节数。
18. bytes_written：memcached服务器发送到网络的总的字节数。
19. limit_maxbytes：memcached服务缓存允许使用的最大字节数。这里为67108864字节，也就是是64M.与我们启动memcached服务设置的大小一致。
20. threads：被请求的工作线程的总数量。

我们还可以用Java操作memcached  
首先下载相关的jar包 [Memcached-Java-Client][10]  
然后编写客户端OCSUtil.java   

        package net.bill.commons.util;

        import org.springframework.stereotype.Component;
        import com.danga.MemCached.MemCachedClient;
        import com.danga.MemCached.SockIOPool;

        @Component
        public class OCSUtil {

        	private static OCSUtil session;

        	public static OCSUtil getSession() {
        		if (session == null) {
        			synchronized(OCSUtil.class){
                        if(session==null){
                        	session=new OCSUtil();
                        }
                    }
        		}
        		return session;
        	}

            /**
             * memcached客户端
             */
            private MemCachedClient memcache = null;

            public OCSUtil(){
            	if (memcache == null) {
            		memcache =new MemCachedClient();  
                    String [] addr ={"127.0.0.1:11211"};  
                    Integer [] weights = {3};  
                    SockIOPool pool = SockIOPool.getInstance();  
                    pool.setServers(addr);  
                    pool.setWeights(weights);  
                    pool.setInitConn(5);  
                    pool.setMinConn(5);  
                    pool.setMaxConn(200);  
                    pool.setMaxIdle(1000*30*30);  
                    pool.setMaintSleep(30);  
                    pool.setNagle(false);  
                    pool.setSocketTO(30);  
                    pool.setSocketConnectTO(0);  
                    pool.initialize();  
        		}
            }

            public void setAttribute(String key, Object value){
            	memcache.set(key, value, 1000);
            }

            public  Object getAttribute(String key){
            	return memcache.get(key);
            }

            public void removeAttribute(String key){
            	memcache.delete(key);
            }

        }

再编写测试代码BaseTest.java和OCSTest.java    

        package net.test.modules;
        import org.junit.Before;
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

        	@Before
        	public void setUp() throws Exception {
        	}
        }  

- - -

        package net.test.modules;

        import net.bill.commons.util.OCSUtil;
        import net.bill.modules.pojo.User;
        import org.junit.Test;
        import org.springframework.beans.factory.annotation.Autowired;

        public class OCSTest extends BaseTest {

        	@Autowired
        	OCSUtil session;

        	/**
        	 * 通过spring注入获得客户端
        	 */
        	@Test
        	public void test1() {
        		session.setAttribute("user", new User("13355558888"));
        		System.out.println(session.getAttribute("user"));
        	}

        	/**
        	 * 通过静态方法获得客户端(单例)
        	 */
        	@Test
        	public void test2() {
        		OCSUtil session = OCSUtil.getSession();
        		System.out.println(session.getAttribute("user"));
        	}
        }

**要注意的是，User类必须实现Serializable接口，进行序列化。因为memcached的value类型是String。**  

接下来我要用memcached替换tomcat的session  
在这之前，可以先看一下[memcached-session-manager][11](google的)，或中文翻译的[memcached-session-manager配置][12]。  
session的序列化方案官方推荐的有4种：  

 - kryo-serializer
 - javolution-serializer
 - xstream-serializer
 - flexjson-serializer

我使用的是最简单的一种，就是 java serialization。这个只要Java中存入session的对象实现Serializable就可以了。
然后下载以下jar包，并放在tomcat\lib\里  

 1. memcached-session-manager-1.8.3.jar
 2. memcached-session-manager-tc6-1.8.3.jar
 3. spymemcached-2.11.1.jar

在tomcat\conf\context.xml，&lt;Context&gt;里加上以下配置：  

        <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"  
        	    memcachedNodes="n1:127.0.0.1:11211"
        	    username="root"
            	password=""  
        	    sticky="false"  
        	    sessionBackupAsync="false"  
        	    lockingMode="uriPattern:/path1|/path2"  
        	    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"  
        	    />

启动tomcat，可以看见日志输出以下信息：

        信息: --------  
        - MemcachedSessionService finished initialization:  
        - sticky: false  
        - operation timeout: 1000  
        - node ids: [n1]  
        - failover node ids: []  
        - storage key prefix: null  
        --------  

###Step4 整体测试###
现在可以测一下是否成功。

 1. 启动tomcat1
 2. 访问test.local.com,并登录
 3. 启动tomcat2，关闭tomcat1
 4. 查看登录信息是否还在

测试通过的话，就基本上没问题了。

  [1]: http://static.oschina.net/uploads/space/2015/0418/164514_kCxK_550406.png
  [2]: http://kevinworthington.com/nginx-for-windows/
  [3]: http://test.local.com
  [4]: http://blog.couchbase.com/memcached-windows-64-bit-pre-release-available
  [5]: http://s3.amazonaws.com/downloads.northscale.com/memcached-win64-1.4.4-14.zip
  [6]: http://memcached.org/
  [7]: http://static.oschina.net/uploads/space/2015/0418/191248_w0sA_550406.png
  [8]: http://static.oschina.net/uploads/space/2015/0418/194250_10b3_550406.png
  [9]: http://static.oschina.net/uploads/space/2015/0418/194644_Rbi0_550406.png
  [10]: http://www.oschina.net/p/memcached-java-client/
  [11]: https://code.google.com/p/memcached-session-manager/wiki/SetupAndConfiguration
  [12]: http://chenzhou123520.iteye.com/blog/1650212
