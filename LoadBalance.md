# 负载均衡（load balance）

## 1、为什么用它？



负载均衡 建立在现有网络结构之上，它提供了一种廉价有效透明的方法扩展[网络设备](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C%E8%AE%BE%E5%A4%87)和[服务器](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8)的带宽、增加[吞吐量](https://baike.baidu.com/item/%E5%90%9E%E5%90%90%E9%87%8F)、加强网络数据处理能力、提高网络的灵活性和可用性。



```doc
在互联网高速发展的时代，大数据量、高并发等是互联网网站提及最多的。如何处理高并发带来的系统性能问题，最终大家都会使用负载均衡机制。它是根据某种负载策略把请求分发到集群中的每一台服务器上，让整个服务器群来处理网站的请求。

指由多台服务器以对称的方式组成一个服务器集合，每台服务器都具有等价的地位，都可以单独对外提供服务而无须其他服务器的辅助。通过某种 负载分担技术，将外部发送来的请求均匀分配到对称结构中的某一台服务器上，而接收到请求的服务器独立地回应客户的请求。负载均衡能够平均分配客户请求到服 务器阵列，借此提供快速获取重要数据，解决大量并发访问服务问题，这种集群技术可以用最少的投资获得接近于大型主机的性能。
负载均衡分为软件负载均衡和硬件负载均衡，前者的代表是阿里章文嵩博士研发的LVS，后者则是均衡服务器比如F5
```

将外部发送来的请求均匀分配到对称结构中的某一台服务器上



## 2、负载均衡算法

### 轮询法（Round Robin）

   轮询算法按顺序把每个新的连接请求分配给下一个服务器，最终把所有请求平分给所有的服务器。

   优点：绝对公平

   缺点：无法根据服务器性能去分配，无法合理利用服务器资源。

```java
/**
 * Title:轮询
 * Description:
 *
 * @author Created by Julie
 * @version 1.0
 * @date on 15:49 2017/10/26
 */
package com.test.loadbalance;


import java.util.*;
import java.util.concurrent.ConcurrentHashMap;


public class TestRoundRobin {



    //    1.定义map, key-ip,value-weight
    static Map<String,Integer> ipMap=new HashMap<>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",1);
        ipMap.put("192.168.13.3",1);

    }

//    Integer sum=0;
    Integer  pos = 0;

    public String RoundRobin(){
        Map<String,Integer> ipServerMap=new ConcurrentHashMap<>();
        ipServerMap.putAll(ipMap);

        //    2.取出来key,放到set中
        Set<String> ipset=ipServerMap.keySet();

        //    3.set放到list，要循环list取出
        ArrayList<String> iplist=new ArrayList<String>();
        iplist.addAll(ipset);

        String serverName=null;

        //    4.定义一个循环的值，如果大于set就从0开始
        synchronized(pos){
            if (pos>=ipset.size()){
                pos=0;
            }
            serverName=iplist.get(pos);
            //轮询+1
            pos ++;
        }
        return serverName;

    }

    public static void main(String[] args) {
        TestRoundRobin testRoundRobin=new TestRoundRobin();
        for (int i=0;i<10;i++){
            String serverIp=testRoundRobin.RoundRobin();
            System.out.println(serverIp);
        }
    }

}
```



### 加权轮询法

```
该算法中，每个机器接受的连接数量是按权重比例分配的。这是对普通轮询算法的改进，比如你可以设定：第三台机器的处理能力是第一台机器的两倍，那么负载均衡器会把两倍的连接数量分配给第3台机器。加权轮询分为：简单的轮询、平滑的轮询。
```

```
 什么是平滑的轮询，就是把每个不同的服务，平均分布。在Nginx源码中，实现了一种叫做平滑的加权轮询（smooth weighted round-robin balancing）的算法，它生成的序列更加均匀。5个请求现在分散开来，不再是连续的。
```

```java
package com.test.loadbalance;


import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * Title:
 * Description:加权轮询
 *
 * @author Created by Julie
 * @version 1.0
 * @date on 18:05 2017/10/26
 */
public class TestWeightRobin {
    //    1.map, key-ip,value-weight
    static Map<String,Integer> ipMap=new HashMap<>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);


    }
    Integer pos=0;
    public String WeightRobin(){
        Map<String,Integer> ipServerMap=new ConcurrentHashMap<>();
        ipServerMap.putAll(ipMap);

        Set<String> ipSet=ipServerMap.keySet();
        Iterator<String> ipIterator=ipSet.iterator();

        //定义一个list放所有server
        ArrayList<String> ipArrayList=new ArrayList<String>();

        //循环set，根据set中的可以去得知map中的value，给list中添加对应数字的server数量
        while (ipIterator.hasNext()){
            String serverName=ipIterator.next();
            Integer weight=ipServerMap.get(serverName);
            for (int i = 0;i < weight ;i++){
                ipArrayList.add(serverName);
            }
        }
        String serverName=null;
        if (pos>=ipArrayList.size()){
            pos=0;
        }
        serverName=ipArrayList.get(pos);
        //轮询+1
        pos ++;


        return  serverName;
    }

    public static void main(String[] args) {
        TestWeightRobin testWeightRobin=new TestWeightRobin();
        for (int i =0;i<10;i++){
            String server=testWeightRobin.WeightRobin();
            System.out.println(server);
        }


    }
}
```



### 随机法

```
负载均衡方法随机的把负载分配到各个可用的服务器上，通过随机数生成算法选取一个服务器。毕竟随机，，有效性受到了质疑。
```

```java
package com.test.loadbalance;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * Title:
 * Description:随机
 *
 * @author Created by Julie
 * @version 1.0
 * @date on 18:25 2017/10/26
 */
public class TestRandom {
    //    1.定义map, key-ip,value-weight
    static Map<String,Integer> ipMap=new HashMap<>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);


    }
    public String Random() {
        Map<String,Integer> ipServerMap=new ConcurrentHashMap<>();
        ipServerMap.putAll(ipMap);

        Set<String> ipSet=ipServerMap.keySet();

        //定义一个list放所有server
        ArrayList<String> ipArrayList=new ArrayList<String>();
        ipArrayList.addAll(ipSet);

        //循环随机数
        Random random=new Random();
        //随机数在list数量中取（1-list.size）
        int pos=random.nextInt(ipArrayList.size());
        String serverNameReturn= ipArrayList.get(pos);
        return  serverNameReturn;
    }

    public static void main(String[] args) {
        TestRandom testRandom=new TestRandom();
        for (int i =0;i<10;i++){
            String server=testRandom.Random();
            System.out.println(server);
        }


    }
}

```



### 加权随机法

```
 获取带有权重的随机数字，随机这种东西，不能看绝对，只能看相对。
```

```java
package com.test.loadbalance;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * Title:
 * Description:加权随机
 *
 * @author Created by Julie
 * @version 1.0
 * @date on 18:42 2017/10/26
 */
public class TestRobinRandom {

    //    1.定义map, key-ip,value-weight
    static Map<String,Integer> ipMap=new HashMap<>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);


    }
    public String RobinRandom(){
        Map<String,Integer> ipServerMap=new ConcurrentHashMap<>();
        ipServerMap.putAll(ipMap);

        Set<String> ipSet=ipServerMap.keySet();
        Iterator<String> ipIterator=ipSet.iterator();

        //定义一个list放所有server
        ArrayList<String> ipArrayList=new ArrayList<String>();

        //循环set，根据set中的可以去得知map中的value，给list中添加对应数字的server数量
        while (ipIterator.hasNext()){
            String serverName=ipIterator.next();
            Integer weight=ipServerMap.get(serverName);
            for (int i=0;i<weight;i++){
                ipArrayList.add(serverName);
            }
        }

        //循环随机数
        Random random=new Random();
        //随机数在list数量中取（1-list.size）
        int pos=random.nextInt(ipArrayList.size());
        String serverNameReturn= ipArrayList.get(pos);
        return  serverNameReturn;
    }

    public static void main(String[] args) {
        TestRobinRandom testRobinRandom=new TestRobinRandom();
        for (int i =0;i<10;i++){
            String server=testRobinRandom.RobinRandom();
            System.out.println(server);
        }


    }
}

```



### IP_Hash算法



```
hash(object)%N算法，通过一种散列算法把请求分配到不同的服务器上。
```

```java
package com.test.loadbalance;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

/**
 * Title:
 * Description:
 *
 * @author Created by Julie
 * @version 1.0
 * @date on 18:35 2017/10/26
 */
public class ipHash {
    //    1.定义map, key-ip,value-weight
    static Map<String,Integer> ipMap=new HashMap<>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);
    }
    public String ipHash(String clientIP){
        Map<String,Integer> ipServerMap=new ConcurrentHashMap<>();
        ipServerMap.putAll(ipMap);

        //    2.取出来key,放到set中
        Set<String> ipset=ipServerMap.keySet();

        //    3.set放到list，要循环list取出
        ArrayList<String> iplist=new ArrayList<String>();
        iplist.addAll(ipset);

        //对ip的hashcode值取余数，每次都一样的
        int hashCode=clientIP.hashCode();
        int serverListsize=iplist.size();
        int pos=hashCode%serverListsize;
        return iplist.get(pos);

    }

    public static void main(String[] args) {
        ipHash iphash=new ipHash();
        String servername= iphash.ipHash("192.168.21.2");
        System.out.println(servername);
    }

}
```





**集群（Cluster）**

用N台服务器构成一个松耦合的多处理器系统(对外来说，他们就是一个服务器)，它们之间通过网络实现通信。让N台服务器之间相互协作，共同承载一个网站的请求压力。





**高可用（HA）**



在集群服务器架构中，当主服务器故障时，备份服务器能够自动接管主服务器的工作，并及时切换过去，以实现对用户的不间断服务。



**session复制/共享**

 在访问系统的会话过程中，用户登录系统后，不管访问系统的任何资源地址都不需要重复登录，这里面servlet容易保存了该用户的会话(session)。如果两个tomcat(A、B)提供集群服务时候，用户在A-tomcat上登录，接下来的请求web服务器根据策略分发到B-tomcat，因为B-tomcat没有保存用户的会话(session)信息，不知道其登录，会跳转到登录界面。
这时候我们需要让B-tomcat也保存有A-tomcat的会话，我们可以使用tomcat的session复制实现或者通过其他手段让session共享。

# Nginx



## Nginx+Tomcat搭建高性能负载均衡集群

### 一、       工具

　　nginx-1.8.0

　　apache-tomcat-6.0.33

### 二、    目标

　　实现高性能负载均衡的Tomcat集群：

```doc

client1
client2----------------------|                                       |-----tomcat1
client3----------------------|                                       |
client4----------------------|                                       |-----tomcat2
client5----------------------|                                       |
                             ---》Nginx服务器-------分发请求--------》                                                            
client6----------------------|                                       |-----tomcat3
client7----------------------|                                       |
client8----------------------|                                       |
client9----------------------|                                       |-----tomcat4
client10---------------------|                                       |
```

### 三、    步骤



#### 1、下载nginx

http://nginx.org/en/download.html

下载解压后放到C:\nginx-1.0.4（官网这样要求的，不知道放其它盘有没有问题）

启动nginx.exe，然后在浏览器输入127.0.0.1即可

使用了默认的nginx.conf 。Nginx的配置文件都存于目录conf文件下，其中nginx.conf是它的主配置文件。

#### 2、解压两个Tomcat

分别命名为apache-tomcat-6.0.33-1和apache-tomcat-6.0.33-2：



#### 3、分别修改启动端口

然后修改这两个Tomcat的启动端口，分别为18080和28080，下面以修改第一台Tomcat为例，打开Tomcat的conf目录下的server.xml

#### 4、server.xml

```xml
<?xml version='1.0' encoding='utf-8'?>

<Server port="18005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JasperListener" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">
    <Connector port="18080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="18443" />
  
    <Connector port="18443" protocol="org.apache.coyote.http11.Http11Protocol" keystoreFile="D:\mykey.keystore" keystorePass="123456"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" />
    

    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="18009" protocol="AJP/1.3" redirectPort="8443" />


    <Engine name="Catalina" defaultHost="localhost">

      <Realm className="org.apache.catalina.realm.LockOutRealm">
        
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

      
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
			   
			   <Context debug="0" docBase="D://file" path="/photo" reloadable="true"/>
      </Host>
    </Engine>
  </Service>
</Server>

```



#### 5、修改页面并测试

然后修改上面两个Tomcat的默认页面（为了区分下面到底访问的是那一台Tomcat，随便改一下即可）：



#### 6、配置 nginx.conf  

OK，现在我们可以开始配置Nginx来实现负载均衡了，其实非常的简单，只需要配置好Nginx的配置文件即可：

```xml
worker_processes  1;#工作进程的个数，一般与计算机的cpu核数一致

events {
    worker_connections  1024;#单个进程最大连接数（最大连接数=连接数*进程数）
}

http {
    include       mime.types; #文件扩展名与文件类型映射表
    default_type  application/octet-stream;#默认文件类型

    sendfile        on;#开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    
	keepalive_timeout  65; #长连接超时时间，单位是秒

    gzip  on;#启用Gizp压缩
	
	#服务器的集群    netitcast.com当前搭建了nginx 服务器主机的域名
    upstream  netitcast.com {  #服务器集群名字	
		server    127.0.0.1:18080  weight=1;#服务器配置   weight是权重的意思，权重越大，分配的概率越大。
		server    127.0.0.1:28080  weight=2;
	}	

	#当前的主机Nginx的配置
    server {
        listen       80;#监听80端口，可以改成其他端口
        server_name  localhost;##############	当前服务的域名
#反向代理配置，将所有请求为http://netitcast.com的请求全部转发到upstream中定义的目标服务器中。
	location / {
#此处配置的域名必须与upstream的域名一致，才能转发。
            proxy_pass http://netitcast.com;  #这里要和upstream 域名一样
            proxy_redirect default;
        }
		
#错误页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```



#### 7、启动Nginx

start nginx     // 开启服务

nginx -s stop   // 关闭服务

#### 8、小结

然后我们即可输入：localhost/index.jsp查看运行状况了

第一次访问，发现访问的是Tomcat2上的程序：

然后刷新，访问的还是Tomcat2上的程序：

　到此，我们利用Nginx已经实现了负载均衡的Tomcat集群。我们不断的刷新，发现访问Tomcat2的概率大概是Tomcat1的2倍，这是因为我们在Nginx中配置的两台Tomcat的权重起的作用。

## Nginx配置upstream实现负载均衡

Nginx能够配置代理多台服务器。当一台服务器宕机之后。仍能保持系统可用。

### 1、默认依照轮询

在http节点下，加入upstream节点。


```xml
  upstream linuxidc { 
  server 10.0.6.108:7080; 
  server 10.0.0.85:8980;
 }
```
2、将server节点下的location节点中的proxy_pass配置为：http:// + upstream名称，即“
http://linuxidc”.


```xml
location / { 
		root  html; 
        index  index.html index.htm; 
        proxy_pass http://linuxidc;    
     }
```
3、如今负载均衡初步完毕了。upstream依照轮询（默认）方式进行负载，每一个请求按时间顺序逐一分配到不同的后端服务器。假设后端服务器down掉。能自己主动剔除。尽管这样的方式简便、成本低廉。但缺点是：可靠性低和负载分配不均衡。适用于图片服务器集群和纯静态页面服务器集群。

### 4、分配策略

```doc
    除此之外，upstream还有其他的分配策略，分别例如以下：

    weight（权重）

    指定轮询几率，weight和訪问比率成正比，用于后端服务器性能不均的情况。例如以下所看到的。10.0.0.88的訪问比率要比10.0.0.77的訪问比率高一倍。

upstream linuxidc{ 
      server 10.0.0.77 weight=5; 
      server 10.0.0.88 weight=10; 
}

    ip_hash（訪问ip）

    每一个请求按訪问ip的hash结果分配。这样每一个訪客固定訪问一个后端服务器，能够解决session的问题。

upstream favresin{ 
      ip_hash; 
      server 10.0.0.10:8080; 
      server 10.0.0.11:8080; 
}

    fair（第三方）

    按后端服务器的响应时间来分配请求。响应时间短的优先分配。

与weight分配策略相似。

 upstream favresin{      
      server 10.0.0.10:8080; 
      server 10.0.0.11:8080; 
      fair; 
}

url_hash（第三方）

按訪问url的hash结果来分配请求，使每一个url定向到同一个后端服务器。后端服务器为缓存时比較有效。

注意：在upstream中加入hash语句。server语句中不能写入weight等其他的參数，hash_method是使用的hash算法。
 upstream resinserver{ 
      server 10.0.0.10:7777; 
      server 10.0.0.11:8888; 
      hash $request_uri; 
      hash_method crc32; 
}

upstream还能够为每一个设备设置状态值，这些状态值的含义分别例如以下：

down 表示单前的server临时不參与负载.

weight 默觉得1.weight越大，负载的权重就越大。

max_fails ：同意请求失败的次数默觉得1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误.

fail_timeout : max_fails次失败后。暂停的时间。

backup： 其他全部的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

upstream bakend{ #定义负载均衡设备的Ip及设备状态 
      ip_hash; 
      server 10.0.0.11:9090 down; 
      server 10.0.0.11:8080 weight=2; 
      server 10.0.0.11:6060; 
      server 10.0.0.11:7070 backup; 
}
```



## 负载均衡-session共享的三种处理方法



# Redis

### 一、下载

官方下载地址：[https://redis.io/download](https://redis.io/download)

redis 64位下载地址：https://github.com/ServiceStack/redis-windows

### 二、配置

redis.windows.conf

```xml
################################## SECURITY ###################################

# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
# 
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
# requirepass foobared
requirepass 123456
  
  
  
# NOTE: since Redis uses the system paging file to allocate the heap memory,
# the Working Set memory usage showed by the Windows Task Manager or by other
# tools such as ProcessExplorer will not always be accurate. For example, right
# after a background save of the RDB or the AOF files, the working set value
# may drop significantly. In order to check the correct amount of memory used
# by the redis-server to store the data, use the INFO client command. The INFO
# command shows only the memory used to store the redis data, not the extra
# memory used by the Windows process for its own requirements. Th3 extra amount
# of memory not reported by the INFO command can be calculated subtracting the
# Peak Working Set reported by the Windows Task Manager and the used_memory_peak
# reported by the INFO command.
#
# maxmemory <bytes>
maxmemory 1024000000
```



### 三、启动

dos窗口在redis 解压目录下 执行   redis-server.exe  redis.windows.conf



### 四、特性

```doc
Redis 有三个主要使其有别于其它很多竞争对手的特点：
Redis是完全在内存中保存数据的数据库，使用磁盘只是为了持久性目的； 
Redis相比许多键值数据存储系统有相对丰富的数据类型； 
Redis可以将数据复制到任意数量的从服务器中； 
优点：
异常快速 : Redis是非常快的，每秒可以执行大约110000设置操作，81000个/每秒的读取操作。
支持丰富的数据类型 : Redis支持最大多数开发人员已经知道如列表，集合，可排序集合，哈希等数据类型。
这使得在应用中很容易解决的各种问题，因为我们知道哪些问题处理使用哪种数据类型更好解决。
操作都是原子的 : 所有 Redis 的操作都是原子，从而确保当两个客户同时访问 Redis 服务器得到的是更新后的值（最新值）。

MultiUtility工具：Redis是一个多功能实用工具，可以在很多如：缓存，消息传递队列中使用（Redis原生支持发布/订阅），在应用程序中，如：Web应用程序会话，网站页面点击数等任何短暂的数据；
```



Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

redis的5种基本数据类型的最基本操作：string、list、set、hash、zset，以及简单的redis消息队列的使用。

 jedis是官方推荐的，使用java操作redis的类库，目前官网的最新稳定版是2.9

    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>
 其实这个依赖只有3个jar包，jedis-2.9.0.jar，commons-pool-1.6.jar，commons-pool2-2.4.2.jar，后面的两个common的pool包，是配置jedis的连接池用的，如果不想使用连接池，那么只要导入jedis-2.9.0.jar就够了。

### 五、实例

#### 单机

pom.xml

```xml
<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- redis Spring 基于注解配置 --> 
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-redis</artifactId>
			<version>1.0.2.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
		</dependency>
		<!-- Redis客户端 -->  
		<dependency>
			<groupId>redis.clients</groupId>
			<artifactId>jedis</artifactId>
		</dependency>

	</dependencies>
```

applicationContext-redis-single.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<context:property-placeholder location="classpath:redis.properties"
		ignore-unresolvable="true" />

	<!-- 配置单机版的连接 -->
	<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
		<constructor-arg name="host" value="${redis.host}" />
		<constructor-arg name="port" value="${redis.port}" />
		<!--<constructor-arg name="poolConfig" ref="jedisPoolConfig"/> -->
	</bean>

	<bean id="jedisClientSingle" class="org.teluoyi.redis.commons.JedisClientSingle">
		<property name="jedisPool" ref="jedisPool" />
	</bean>
</beans>
```



JedisClient

```java
package org.teluoyi.redis.commons;

public interface JedisClient {

	String set(String key, String value);

	String get(String key);

	Boolean exists(String key);

	Long expire(String key, int seconds);

	Long ttl(String key);

	Long incr(String key);

	Long hset(String key, String field, String value);

	String hget(String key, String field);

	Long hdel(String key, String... field);
}

```

JedisClientSingle.java

```java
package org.teluoyi.redis.commons;


import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
/**
 * 
* @ClassName: JedisClientPool
* @Description: TODO(单机版实现类)
* @author wenjin.zhu
* @date 2018年5月9日
*
 */
public class JedisClientSingle implements JedisClient {
	
	private JedisPool jedisPool;

	public void setJedisPool(JedisPool jedisPool) {
		this.jedisPool = jedisPool;
	}

	@Override
	public String set(String key, String value) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.set(key, value);
		jedis.close();
		return result;
	}

	@Override
	public String get(String key) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.get(key);
		jedis.close();
		return result;
	}

	@Override
	public Boolean exists(String key) {
		Jedis jedis = jedisPool.getResource();
		Boolean result = jedis.exists(key);
		jedis.close();
		return result;
	}

	@Override
	public Long expire(String key, int seconds) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.expire(key, seconds);
		jedis.close();
		return result;
	}

	@Override
	public Long ttl(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.ttl(key);
		jedis.close();
		return result;
	}

	@Override
	public Long incr(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.incr(key);
		jedis.close();
		return result;
	}

	@Override
	public Long hset(String key, String field, String value) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hset(key, field, value);
		jedis.close();
		return result;
	}

	@Override
	public String hget(String key, String field) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.hget(key, field);
		jedis.close();
		return result;
	}

	@Override
	public Long hdel(String key, String... field) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hdel(key, field);
		jedis.close();
		return result;
	}

}
```



测试类

```java
package org.teluoyi.redis.single;


import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.teluoyi.redis.commons.JedisClient;

/**
 * 
* @ClassName: TestJedis2
* @Description: TODO(单机版使用连接池基于spring配置)
* @author wenjin.zhu
* @date 2018年5月9日
*
 */
public class TestJedisSingle {

	public static void main(String[] args) {
		@SuppressWarnings("resource")
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext-redis-single.xml");
		/*JedisPool jedisPool = (JedisPool) applicationContext.getBean("jedisPool");
		Jedis jedis = jedisPool.getResource();
		jedis.set("name", "amygood");
		String name = jedis.get("name");
		System.out.println(name);
		jedis.close();*/
		
		 JedisClient jedisClient = applicationContext.getBean(JedisClient.class);  
		//JedisClient jedisClient = (JedisClient) applicationContext.getBean("jedisClientSingle");
		jedisClient.set("age", "100");  
	    String result = jedisClient.get("age");  
	    System.out.println(result);  
	}


}

```



# Redis集群





































































