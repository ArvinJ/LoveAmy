# EL表达式的使用

## **1、什么是EL**

EL是JSP表达式语言，全称是ExpressionLanguage.

EL表达式是一种JSP技术，能够代替JSP中原本要用Java语言进行显示的语句，使得代码更容易编写与维护。

最基本的语法是${express}。

使用EL的目的是简化在JSP中访问变量的方式，简单静态HTML与Java代码的耦合。

传统写法例：

```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"  
    pageEncoding="ISO-8859-1"%>  
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">  
<html>  
<head>  
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">  
<title>Insert title here</title>  
</head>  
<body>  
<%  
String a=request.getParameter("a");  
String b=request.getParameter("b");  
out.println("a+b="+(Integer.parseInt(a)+Integer.parseInt(b)));  
 %>  
</body>  
</html>  
```

下面用EL来写：

```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"  
    pageEncoding="ISO-8859-1"%>  
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">  
<html>  
<head>  
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">  
<title>Insert title here</title>  
</head>  
<body>  
 ${param.a}+${param.b}=${param.a+param.b}  
</body>  
</html>  
```

代码相对原本少了很多，而且很明了。

## 2、JSP EL 表达式用于以下情形

静态文本
标准标签和自定义标签
安装支持Servlet2.4/JSP2.0的Container

## 3、基本语法格式

${ EL Expression}

示例：
${ “Helloworld” }  //输出字符串常量
${ str }  //输出字符串变量str的值
${ 3 + 2 }  //输出3+2的结果
${ user.name} //输出user对象的name属性
${user[“name”] }  //同上
${ sessionScope[“user”].name } //同上
${user.name}
访问对象user的getName()方法以得到name成员的值。
${list[1]}
访问list对象的第二项。
${map[“key”]}
访问map指定键的值。
再来看个例子
EL的语法很简单，他最大的特点就是使用上很方便

${sessionScope.user.sex}   

等于

${sessionScope.user["sex"]}  

**所有EL都是以 ${ 为起始、以} 为结尾的**。

<% User user = (User)session.getAttribute("user");   String sex = user.getSex( ); %>

## 4、“.”与”[ ]”的相同点和差别

都可以访问对象有属性。
差别：
当属性的名字包含有空格，点号等复杂符号时。使用”.”来访问对象有属性将会出现异常



## 5.获取并显示数据

从四个域中通过key找到简单数据并显示出来。表达式代码：

${name}      <!-- 类比于<%=pageContext.findAttribute("name") %> -->  
四个域的寻找顺序是 page，request，session，application。用EL表达式还有个好处，若找不到键值为name的属性值，不会显示null，会显示空字符串。若是确定键值是在request域中，则可以用如下EL表达式代码：

${requestScope.name}  



从存储在WEB域中的封装了数据的JavaBean中得到对象的某个属性值并显示出来。

```jsp
<%
	Person p = new Person();
	Address address = new Address();
	address.setCity("浙江");//Adress类中有个私有String属性为city
	p.setName("mike");//Person类中有私有String属性为name
	p.setAddress(address);//Person类中有私有Adress属性为address
	session.setAttribute("person", p);//存入sessionScope
%>
${person.name}<!-- 从WEB域中找到键值为person的对象，然后再person对象中找到name属性 -->
${person.address.city}
```

${person['name']}<!-- 也可以用[]方式 -->
除了用.方式获得对象的属性，也可以用[ ]方式，当然遇到键值名字中有-的，如mike-abc，或者key值为数字开头，则只能用[ ]方式。



从List集合对象中获取某个值并显示。



```
<%  
List<Person> list = new ArrayList<Person>();  
list.add(new Person("kkk"));  
list.add(new Person("sss"));  
list.add(new Person("jjj"));  
application.setAttribute("list_1", list);  
%>  

${list_1[1].name } 
```

 从Map中获取某个值并显示。

```jsp
<%
	Map map = new HashMap();
	map.put("a", new Person("aaa"));
	map.put("b", new Person("bbb"));
	map.put("1", new Person("ccc"));
	request.setAttribute("map", map);
%>
${map['1'].name }<!-- 是数字的话只能用括号，就算put进去的key值是字符串类型-->
${map.a.name }
```



## 6.执行运算

语法：${运算表达式}

常见运算符：==(eq)   !=(ne)    <(lt)    >(gt)    <=(le)    >=(ge)    &&(and)   ||(or)   !(not)

判断是否为空：**${empty name }**

三目运算符：${name == null?"null":name }

## 7.获取常用对象

1)语法：${隐式对象名称}

2)隐式对象表

| 对象名          | 用法                                 | 等价JSP代码或作用                               |
| ------------ | ---------------------------------- | ---------------------------------------- |
| param        | ${param.name}                      | request.getParameter(name)               |
| paramValues  | ${paramValues.name}                | request.getParameterValues(name)//返回一个字符串数组 |
| header       | ${header.name}                     | request.getHeader(name)                  |
| headerValues | ${headerValues.name}               | request.getHeaderValues(name)            |
| cookie       | ${cookie.name.value}               | request.getCookie()                      |
| initParam    | ${initParam.name}                  | ServletContext.getInitparameter(name)    |
| pageContext  | ${pageContext.request.contextPath} | 获取WEB应用名                                 |
| pageContext  | ${pageContext.request.sessionid}   | 获取sessionId                              |
| pageContext  | ${pageContext.request.remoteAddr}  | 获取主机名                                    |

## 8.EL表达式原理及注意点

原理：EL表达式在获取某个对象的属性值时，先将某个属性值首字母变成大写，然后加上get前缀，拼接成getter方法，通过反射将该对象构建出来，然后再对该对象执行getter方法，这与私有属性并没有关系，所以要注意，JavaBean的属性名要小写，且要有getter方法，不然会报错。







# JSTL核心标签库使用

```dockerfile
JSTL 核心标签库标签共有13个，功能上分为4类：

1.表达式控制标签：out、set、remove、catch

2.流程控制标签：if、choose、when、otherwise

3.循环标签：forEach、forTokens

4.URL操作标签：import、url、redirect
```

使用标签时，一定要在jsp文件头加入以下代码：

```xml
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```



## **1.c:out**

 用来显示数据对象（字符串、表达式）的内容或结果

使用Java脚本的方式为：<% out.println("hello") %>  <% =表达式 %>

使用JSTL标签：<c:out value="字符串">，例如：

```xml
<body>
  <c:out value="&lt要显示的数据对象（未使用转义字符）&gt" escapeXml="true" default="默认值"></c:out><br/>
  <c:out value="&lt要显示的数据对象（使用转义字符）&gt" escapeXml="false" default="默认值"></c:out><br/>
  <c:out value="${null}" escapeXml="false">使用的表达式结果为null，则输出该默认值</c:out><br/>
</body>
```



## 2.c:set

用于将变量存取于 JSP 范围中或 JavaBean 属性中

下面的例子中假设已经有 Person.java 这个类文件。

```jsp
<%@ page language="java" import="java.util.*" pageEncoding="gb2312"%>
<%@page contentType="text/html; charset=utf-8" %>

<jsp:useBean id="person" class="lihui.Person"></jsp:useBean>

<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title>JSTL测试</title>
  </head>
  
  <body>
  <c:set value="张三" var="name1" scope="session"></c:set>
  <c:set var="name2" scope="session">李四</c:set>
  <c:set value="赵五" target="${person}" property="name"></c:set>
  <c:set target="${person}" property="age">19</c:set>
  <li>从session中得到的值：${sessionScope.name1}</li>
  <li>从session中得到的值：${sessionScope.name2}</li>
  <li>从Bean中获取对象person的name值：<c:out value="${person.name}"></c:out></li>
  <li>从Bean中获取对象person的age值：<c:out value="${person.age}"></c:out></li>
  </body>
</html>
```

一**共有四种语法格式，前两种是给jsp的范围变量赋值，后两个是给 javabean 变量赋值**

## 3.c:remove

主要用来从指定的 jsp 范围内移除指定的变量。

<c:remove var="变量名" [scope="page|request|session|application"]></c:remove>

## 4.c:catch

用来处理 JSP 页面中产生的异常，并存储异常信息

<c:catch var="name1">

```
  容易产生异常的代码
```

</c:catch>

如果抛异常，则异常信息保存在变量 name1 中。

## 5.c:if

<c:if test="条件1" var="name" [scope="page|request|session|application"]></c:if>

例：

```jsp
<body>
  <c:set value="赵五" target="${person}" property="name"></c:set>
  <c:set target="${person}" property="age">19</c:set>
  <c:if test="${person.name == '赵武'}" var="name1"></c:if>
  <c:out value="name1的值：${name1}"></c:out><br/>
  <c:if test="${person.name == '赵五'}" var="name2"></c:if>
  <c:out value="name2的值：${name2}"></c:out>
  </body>
```

## 6. c:choose  c:when   c:otherwise

三个标签通常嵌套使用，第一个标签在最外层，最后一个标签在嵌套中只能使用一次

```jsp
<c:set var="score">85</c:set>
    <c:choose>
    <c:when test="${score>=90}">
    你的成绩为优秀！
    </c:when>
    <c:when test="${score>=70&&score<90}">
    您的成绩为良好!
    </c:when>
    <c:when test="${score>60&&score<70}">
    您的成绩为及格
    </c:when>
    <c:otherwise>
    对不起，您没有通过考试！
    </c:otherwise>
    </c:choose>
```

## 7.c:forEach

```jsp
<c:forEach var="name" items="Collection" varStatus="statusName" begin="begin" end="end" step="step"></c:forEach>
```

该标签根据循环条件遍历集合 Collection 中的元素。

 var 用于存储从集合中取出的元素；

items 指定要遍历的集合；

varStatus 用于存放集合中元素的信息。

varStatus 一共有4种状态属性

例：

```jsp
<%@ page contentType="text/html;charset=GBK" %>
<%@page import="java.util.List"%>
<%@page import="java.util.ArrayList"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>JSTL: -- forEach标签实例</title>
</head>
<body>
<h4><c:out value="forEach实例"/></h4>
<hr>
    <% 
        List a=new ArrayList();
        a.add("贝贝");
        a.add("晶晶");
        a.add("欢欢");
        a.add("莹莹");
        a.add("妮妮");
    request.setAttribute("a",a);
%>
    <B><c:out value="不指定begin和end的迭代：" /></B><br>
    <c:forEach var="fuwa" items="${a}">
    &nbsp;<c:out value="${fuwa}"/><br>
    </c:forEach>
    <B><c:out value="指定begin和end的迭代：" /></B><br>
    <c:forEach var="fuwa" items="${a}" begin="1" end="3" step="2">
    &nbsp;<c:out value="${fuwa}" /><br>
    </c:forEach>
    <B><c:out value="输出整个迭代的信息：" /></B><br>
    <c:forEach var="fuwa" items="${a}" begin="3" end="4" step="1" varStatus="s">
    &nbsp;<c:out value="${fuwa}" />的四种属性：<br>
    &nbsp;&nbsp;所在位置，即索引：<c:out value="${s.index}" /><br>
    &nbsp;&nbsp;总共已迭代的次数：<c:out value="${s.count}" /><br>
    &nbsp;&nbsp;是否为第一个位置：<c:out value="${s.first}" /><br>
    &nbsp;&nbsp;是否为最后一个位置：<c:out value="${s.last}" /><br>
    </c:forEach>
</body>
</html>
```

## 8.c:forTokens

用于浏览字符串，并根据指定的字符串截取字符串

<c:forTokens items="stringOfTokens" delims="delimiters" [var="name" begin="begin" end="end" step="len" varStatus="statusName"]></c:forTokens>

例

```jsp
<%@ page contentType="text/html;charset=GBK"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
    <head>
        <title>JSTL: -- forTokens标签实例</title>
    </head>
    <body>
        <h4>
            <c:out value="forToken实例" />
        </h4>
        <hr>
        <c:forTokens items="北、京、欢、迎、您" delims="、" var="c1">
            <c:out value="${c1}"></c:out>
        </c:forTokens>
        <br>
        <c:forTokens items="123-4567-8854" delims="-" var="t">
            <c:out value="${t}"></c:out>
        </c:forTokens>
        <br>
        <c:forTokens items="1*2*3*4*5*6*7" delims="*" begin="1" end="3"
            var="n" varStatus="s">
    &nbsp;<c:out value="${n}" />的四种属性：<br>
        &nbsp;&nbsp;所在位置，即索引：<c:out value="${s.index}" />
            <br>
        &nbsp;&nbsp;总共已迭代的次数：<c:out value="${s.count}" />
            <br>
        &nbsp;&nbsp;是否为第一个位置：<c:out value="${s.first}" />
            <br>
        &nbsp;&nbsp;是否为最后一个位置：<c:out value="${s.last}" />
            <br>
        </c:forTokens>
    </body>
</html>
```



## 9.c:import

URL 操作标签

把其他静态或动态文件包含到 JSP 页面。

与<jsp:include>的区别是后者只能包含同一个web应用中的文件，前者可以包含其他web应用中的文件，甚至是网络上的资源。

```jsp
<%@ page contentType="text/html;charset=GBK"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
    <head>
        <title>JSTL: -- import标签实例</title>
    </head>
    <body>
        <h4>
            <c:out value="import实例" />
        </h4>
        <hr>
        <h4>
            <c:out value="绝对路径引用的实例" />
        </h4>
        <c:catch var="error1">
            <c:import url="http://www.baidu.com" />
        </c:catch>
        <c:out value="${error1}"></c:out>
        <hr>
        <h4>
            <c:out value="相对路径引用的实例，引用本应用中的文件" />
        </h4>
        <c:catch>
            <c:import url="a1.txt" charEncoding="gbk" />
        </c:catch>
        <hr>
        <h4>
            <c:out value="使用字符串输出、相对路径引用的实例，并保存在session范围内" />
        </h4>
        <c:catch var="error3">
            <c:import var="myurl" url="a1.txt" scope="session" charEncoding="gbk"></c:import>
            <c:out value="${myurl}"></c:out>
            <c:out value="${myurl}" />
        </c:catch>
        <c:out value="${error3}"></c:out>
    </body>
</html>
```

URL路径有个绝对路径和相对路径。相对路径：<c:import url="a.txt"/>那么，a.txt必须与当前文件放在同一个文件目录下。如果以"/"开头，表示存放在应用程序的根目录下，如Tomcat应用程序的根目录文件夹为 webapps。导入该文件夹下的 b.txt 的编写方式： <c:import url="/b.txt">。如果要访问webapps管理文件夹中的其他Web应用，就要用context属性。例如访问demoProj下的index.jsp，则：<c:import url="/index.jsp" context="/demoProj"/>.



## 10.c:redirect

该标签用来实现请求的重定向。例如，对用户输入的用户名和密码进行验证，不成功则重定向到登录页面。或者实现Web应用不同模块之间的衔接

```jsp
 <%@ page contentType="text/html;charset=GBK"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<c:redirect url="http://127.0.0.1:8080">
    <c:param name="uname">lihui</c:param>
    <c:param name="password">11111</c:param>
 </c:redirect>
```

## 11.c:url

用于动态生成一个 String 类型的URL，可以同上个标签共同使用，也可以使用HTML的<a>标签实验超链接。

```jsp
<%@ page contentType="text/html;charset=GBK"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<c:out value="url标签使用"></c:out>
<h4>
    使用url标签生成一个动态的url，并把值存入session中.
</h4>
<hr>
<c:url value="http://127.0.0.1:8080" var="url" scope="session">
</c:url>
<a href="${url}">Tomcat首页</a>
```

# EasyUI

http://www.jeasyui.net/tutorial/147.html

jQuery EasyUI 是一个基于 jQuery 的框架，集成了各种用户界面插件。

使用EasyUI之前必须先声明UI控件，有两个方法声明它。

1. 直接在 HTML 声明组件。

```html
<div class="easyui-dialog" style="width:400px;height:200px"
    data-options="title:'My Dialog',collapsible:true,iconCls:'icon-ok',onOpen:function(){}">
        dialog content.
</div>
```

 2.编写 JavaScript 代码来创建组件。

```javascript
<input id="cc" style="width:200px" />
  
  $('#cc').combobox({
	url: ...,
	required: true,
	valueField: 'id',
	textField: 'text'
});
```



easyui的简单使用步骤

step1: easyui下载；

step2:引入必要的js和css样式文件

```jsp
<!-- 引入JQuery -->
  <script type="text/javascript" src="${pageContext.request.contextPath}/jquery-easyui-1.4.1/jquery.min.js"></script>
  <!-- 引入EasyUI -->
  <script type="text/javascript" src="${pageContext.request.contextPath}/jquery-easyui-1.4.1/jquery.easyui.min.js"></script>
  <!-- 引入EasyUI的中文国际化js，让EasyUI支持中文 -->
  <script type="text/javascript" src="${pageContext.request.contextPath}/jquery-easyui-1.4.1/locale/easyui-lang-zh_CN.js"></script>
  <!-- 引入EasyUI的样式文件-->
  <link rel="stylesheet" href="${pageContext.request.contextPath}/jquery-easyui-1.4.1/themes/default/easyui.css" type="text/css"/>
  <!-- 引入EasyUI的图标样式文件-->
  <link rel="stylesheet" href="${pageContext.request.contextPath}/jquery-easyui-1.4.1/themes/icon.css" type="text/css"/>
```

step3:应用到jsp中

```jsp
<%@ page language="java" pageEncoding="UTF-8"%>
<!DOCTYPE HTML>
<html>
  <head>
    <title>EasyUI入门——创建EasyUI的Dialog</title>
  <!-- 引入JQuery -->
  <script type="text/javascript" src="${pageContext.request.contextPath}/jquery-easyui-1.4.1/jquery.min.js"></script>
  <!-- 引入EasyUI -->
  <script type="text/javascript" src="${pageContext.request.contextPath}/jquery-easyui-1.4.1/jquery.easyui.min.js"></script>
  <!-- 引入EasyUI的中文国际化js，让EasyUI支持中文 -->
  <script type="text/javascript" src="${pageContext.request.contextPath}/jquery-easyui-1.4.1/locale/easyui-lang-zh_CN.js"></script>
  <!-- 引入EasyUI的样式文件-->
  <link rel="stylesheet" href="${pageContext.request.contextPath}/jquery-easyui-1.4.1/themes/default/easyui.css" type="text/css"/>
  <!-- 引入EasyUI的图标样式文件-->
  <link rel="stylesheet" href="${pageContext.request.contextPath}/jquery-easyui-1.4.1/themes/icon.css" type="text/css"/>
  
  <script type="text/javascript">
      $(function(){
          //console.info($('#dd2'));
          /*使用JavaScript动态创建EasyUI的Dialog的步骤：
          1、定义一个div，并给div指定一个id
          2、使用Jquery选择器选中该div，然后调用dialog()方法就可以创建EasyUI的Dialog
          */
          $('#dd2').dialog();//使用默认的参数创建EasyUI的Dialog
          //使用自定义参数创建EasyUI的Dialog
          $('#dd3').dialog({
              title: '使用JavaScript创建的Dialog',
              width: 400,
              height: 200,
              closed: false,
              cache: false,
              modal: true
          });
      });
  </script>
  
  </head>
  
  <body>
      <%--使用纯html的方式创建创建EasyUI的Dialog的步骤：
      1、定义一个div
      2、将div的class样式属性设置成easyui-dialog，这样就可以将普通的div变成EasyUI的Dialog了
       --%>
    <div class="easyui-dialog" id="dd1" title="EasyUI Dialog" style="width: 500px;height: 300px;">
        Hello World!
    </div>
    <div id="dd2">Dialog Content</div>
    <div id="dd3">Dialog Content</div>
  </body>
</html>
```

# SiteMesh



## 1 . Sitemesh 3 简介

Sitemesh 是一个网页布局和修饰的框架，基于 Servlet 中的 Filter。

SiteMesh 是一个网页布局和修饰的框架，利用它可以将网页的内容和页面结构分离，以达到页面结构共享的目的。

主要是用来统一页面风格，减少重复编码的。

它定义了一个过滤器，然后把页面都加上统一的头部和底部。

## 2 . Sitemesh  HelloWorld

最近在搭建新的后台管理系统框架的时候需要用到三栏布局，如下图所示，之前采用的  frameset  框架，据说有很多的缺陷，所以今天就试着用SiteMesh进行布局。



### step1下载jar

首先就是加Jar包，我用的是sitemesh-3.0.0.jar，

```xml
<dependency>
   <groupId>org.sitemesh</groupId>
   <artifactId>sitemesh</artifactId>
   <version>3.0.0</version>
 </dependency>
```

### step2 web.xml配置 过滤器

然后在web.xml中增加过滤器：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"  
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
 xsi:schemaLocation="http://java.sun.com/xml/ns/javaee   
 http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">  
 <welcome-file-list>  
  <welcome-file>index.jsp</welcome-file>  
 </welcome-file-list>  
 <filter>  
  <filter-name>sitemesh</filter-name>  
  <filter-class>  
   com.opensymphony.module.sitemesh.filter.PageFilter  
  </filter-class>  
 </filter>  
 <filter-mapping>  
  <filter-name>sitemesh</filter-name>  
  <url-pattern>/*</url-pattern>  
 </filter-mapping>  
</web-app>  
```

### step3:配置文件decorators.xml

增加SiteMesh配置文件decorators.xml，该文件放在WEB-INF下：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<decorators defaultdir="/layouts/">  
 <!-- 不需要过滤的请求 -->  
 <excludes>  
  <pattern>/static/*</pattern>  
  <pattern>/remote/*</pattern>  
 </excludes>  
 <!-- 定义装饰器要过滤的页面 -->  
 <decorator name="default" page="default.jsp">  
  <pattern>/*</pattern>  
 </decorator>  
</decorators>  
```

### step4:创建文件示例

在WEB-INF根目录下新建文件夹layouts，然后新建三个JSP，一个是默认，一个输出头，一个输出尾，默认页面引用其他两个。

默认页面default.jsp：

```jsp
<%@ page contentType="text/html;charset=UTF-8"%>  
<%@ taglib prefix="sitemesh" uri="http://www.opensymphony.com/sitemesh/decorator" %>    
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>  
<c:set var="ctx" value="${pageContext.request.contextPath}" />
<html>  
<head>  
<title>SiteMesh示例-<sitemesh:title/></title>  
  <meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
<meta http-equiv="Cache-Control" content="no-store" />
<meta http-equiv="Pragma" content="no-cache" />
<meta http-equiv="Expires" content="0" />

<link type="image/x-icon" href="${ctx}/static/images/favicon.ico" rel="shortcut icon">
<link href="${ctx}/sc/bootstrap/2.3.0/css/bootstrap.min.css" type="text/css" rel="stylesheet" />
<link href="${ctx}/sc/jquery-validation/1.11.0/validate.css" type="text/css" rel="stylesheet" />
<link href="${ctx}/css/base/default.css" type="text/css" rel="stylesheet" />
<script src="${ctx}/sc/jquery/jquery-1.9.1.min.js" type="text/javascript"></script>
<script src="${ctx}/sc/jquery-validation/1.11.0/jquery.validate.min.js" type="text/javascript"></script>
<script src="${ctx}/sc/jquery-validation/1.11.0/messages_bs_zh.js" type="text/javascript"></script>

<sitemesh:head/>  
</head>  
<body>  
 <%@ include file="/layouts/header.jsp"%>  
  <div id="content">  
   <sitemesh:body/>  
  </div>  
 <%@ include file="/layouts/footer.jsp"%>  
</body>  
</html>  
```

简单说明：

- 引入了SiteMesh标签。
- <sitemesh:title/> 会自动替换为被过滤页面的title。
- <sitemesh:head/> 会把被过滤页面head里面的东西（除了title）放在这个地方。
- <sitemesh:body/> 被过滤的页面body里面的内容放在这里。
- 头部引入js和css，都可以在其他重用。



头页面header.jsp：

<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>  
菜单信息  



尾页面footer.jsp：

<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>  
版权信息  



在根下新建一个文件夹static，用于实验是否拦截，在该文件夹下新建JSP：

```jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>  
<%  
String path = request.getContextPath();  
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  
%>  
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">  
<html>  
  <head>  
    <base href="<%=basePath%>">  
    <title>有人拦截我吗？</title>  
  </head>  
  <body>  
    有人拦截我吗？  
  </body>  
</html>  
```

访问：http://127.0.0.1:8080/sitemesh/index.jsp这个会拦截
访问：http://127.0.0.1:8080/sitemesh/static/index.jsp则不会拦截处理



# **Tomcat**

## 1.tomcat 访问manager

http://localhost:8080/manager/status  查看JVM运行内存

```xml
请找到您的tomcat\conf目录下的tomcat-user.xml文件，打开。
<role rolename="manager-gui"/>
<user username="tomcat" password="tomcat" roles="manager-gui"/>
manager-gui是最高权限，username是用户名，password是密码！
提示：修改完毕，保存，记得重启tomcat，否则不生效！
```

## 2.

tomcat绿色版修改方法。

在bin目录下找到catalina.bat；在:gotHome后输入

set "JAVA_OPTS=-Xms512m -Xmx1024m" 

```xml
set "CURRENT_DIR=%cd%"
if not "%CATALINA_HOME%" == "" goto gotHome
set "CATALINA_HOME=%CURRENT_DIR%"
if exist "%CATALINA_HOME%\bin\catalina.bat" goto okHome
cd ..
set "CATALINA_HOME=%cd%"
cd "%CURRENT_DIR%"
:gotHome
set "JAVA_OPTS=-Xms512m -Xmx1024m"  
if exist "%CATALINA_HOME%\bin\catalina.bat" goto okHome
echo The CATALINA_HOME environment variable is not defined correctly
echo This environment variable is needed to run this program
goto end
:okHome
```



注意：内存大小根据实际情况自己调整。







# Quartz

## quartz可以用来做什么？

Quartz是一个**任务调度框架**。比如你遇到这样的问题

想每月25号，信用卡自动还款
想每年4月1日自己给当年暗恋女神发一封匿名贺卡


在某一个有规律的时间点干某件事。并且时间的触发的条件可以非常复杂（比如每月最后一个工作日的17:50），复杂到需要一个专门的框架来干这个事。 Quartz就是来干这样的事，你给它一个触发条件的定义，它负责到了时间点，触发相应的Job起来干活。

## Quartz 的几个概念类

### 1、org.quartz.Job 

它是一个抽象接口，

表示一个工作，也就是我们要执行的具体内容他只定义了一个方法： 

void execute(JobExecutionContext context) 
作用等同Spring的： 
**org.springframework.scheduling.quartz.QuartzJobBean**

### **2、org.quartz.JobDetail** 

JobDetail表示一个具体的可执行的调度程序，

Job是这个可执行程调度程序所要执行的内容，它包含了这个任务调度的方案和策略。 

他的实现类： 
**org.quartz.impl.JobDetailImpl** 
作用等同Spring： 
**org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean**

### **3、org.quartz.Trigger** 

它是一个抽象接口，

表示一个调度参数的配置，

通过配置它，来告诉调度容器什么时候去调用JobDetail。 
他的两个实现类: 
**org.quartz.impl.triggers.SimpleTriggerImpl** 
**org.quartz.impl.triggers.CronTriggerImpl** 
等同于Spring的: 
**org.springframework.scheduling.quartz.SimpleTriggerBean** 
**org.springframework.scheduling.quartz.CronTriggerBean** 
前者只支持按照一定频度调用任务，如每隔30分钟运行一次。 
后者既支持按照一定频度调用任务，又支持定时任务。

### **4、org.quartz.Scheduler** 

代表一个调度容器，一个调度容器中可以注册多个JobDetail和Trigger。当Trigger与JobDetail组合，就可以被Scheduler容器调度了。它的方法有start()、shutdown()等方法，负责管理整个调度作业。 
等同Spring的： **org.springframework.scheduling.quartz.SchedulerFactoryBean**





## Spring注解配置定时任务<task:annotation-driven/>

首先在配置文件头部的必须要有：

xmlns:task="http://www.springframework.org/schema/task"

其次xsi:schemaLocation必须为其添加：

http://www.springframework.org/schema/task 
http://www.springframework.org/schema/task/spring-task.xsd

然后spring扫描过程必须涵盖定时任务类所在的目录：

<context:component-scan base-package="com.xx.xx" />

com.xx.xx属于定时任务类的父级甚至更高级 
然后设置动作启用定时任务

<task:annotation-driven/>



最后设置任务类

```java
import org.springframework.context.annotation.Lazy;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;


@Component
@Lazy(value=false)
public class MyQuartzs {

    @Scheduled(cron = "*/5 * * * * ?")//每隔5秒执行一次
    public void test() throws Exception {
        System.out.println("Test is working......");
    }


    //@Scheduled(cron = "0 0 1 * * ?")//每天凌晨1点整
    //@Scheduled(cron = "0 30 0 * * ?")//每天凌晨0点30分
    //@Scheduled(cron = "0 */60 * * * ?")//1小时处理一次
}
```









# SpringMVC



# **分布式**

## 1.1 什么是分布式

1. 分布式系统一定是由多个节点组成的系统。其中，节点指的是计算机服务器，而且这些节点一般不是孤立的，而是互通的。
2. 这些连通的节点上部署了我们的节点，并且相互的操作会有协同。分布式系统对于用户而言，他们面对的就是一个服务器，提供用户需要的服务而已，而实际上这些服务是通过背后的众多服务器组成的一个分布式系统，因此分布式系统看起来像是一个超级计算机一样。

## 1.2 分布式与集群的区别

1.  集群是同一个系统部在不同的服务器上,例如一个登陆系统部在不同的服务器上.
2.  分布式是不同的系统部在不同的服务器上,服务器之间相互调用.

小饭店原来只有一个厨师，切菜洗菜备料炒菜全干。后来客人多了，厨房一个厨师忙不过来，又请了个厨师，两个厨师都能炒一样的菜，这两个厨师的关系是集群。为了让厨师专心炒菜，把菜做到极致，又请了个配菜师负责切菜，备菜，备料，厨师和配菜师的关系是分布式，一个配菜师也忙不过来了，又请了个配菜师，两个配菜师关系是集群.

## 1.3什么是SOA

SOA是一种设计方法，其中包含多个服务,而服务之间通过配合最终会提供一系列功能。一个服务通常以独立的形式存在于操作系统进程中。服务之间通过网络调用，而非采用进程内调用的方式进行通信。

## 1.4Dubbo 是什么

一款分布式服务框架
高性能和透明化的RPC远程服务调用方案
SOA服务治理方案

Provider：服务提供方 
Consumer：服务消费者 
Registry：注册中心 
Monitor：统计服务调用次数和调用时间的监控中心

调用流程 
0.启动服务提供者。 
1.服务提供者在启动时，向注册中心注册自己提供的服务。 
2.服务消费者在启动时，向注册中心订阅自己所需的服务。 
3.注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。 
4.服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。 
5.服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。



注册中心 
服务提供方：针对所提供的服务到注册中心发布。 
服务消费方：到服务中心订阅所需的服务。 
对于任何一方，不论服务提供方或者服务消费方都有可能同时兼具两种角色，即需要提供服务也需要消费服务。

## 1.5ZooKeeper下载并安装

下载
 Apache官方最新版本为：3.4.8
下载地址：https://mirrors.cnnic.cn/apache/zookeeper/

  安装

      解压到指定目录下    D:\soft\zookeeper-3.4.8
      修改zoo_sample.cfg 文件名(D:\soft\zookeeper-3.4.8\conf) 为 zoo.cfg
      主要修改一下日志位置，具体配置文件如下：
```xml
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=D:\\zookeeper\\data
dataLogDir=D:\\zookeeper\\log
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

```

配置文件简单解析
1、tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
2、dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
3、dataLogDir：顾名思义就是 Zookeeper 保存日志文件的目录
4、clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。



启动进入到bin目录，并且启动zkServer.cmd，这个脚本中会启动一个java进程

启动后jps可以看到QuorumPeerMain的进程       jps -l -v





## Docker











## 数据库连接池





## RMI



## WebService



# ActiveMQ

## 1、下载

ActiveMQ官网下载地址：http://activemq.apache.org/download.html

　ActiveMQ 提供了Windows 和Linux、Unix 等几个版本，楼主这里选择了Linux 版本下进行开发。

下载完安装包，解压之后的目录：

从它的目录来说，还是很简单的： 

bin存放的是脚本文件
conf存放的是基本配置文件
data存放的是日志文件
docs存放的是说明文档
examples存放的是简单的实例
lib存放的是activemq所需jar包
webapps用于存放项目的目录

## 2、启动ActiveMQ　

windows 下执行  D:\Arvin\soft\activemq\apache-activemq-5.15.3-bin\apache-activemq-5.15.3\bin\win64 目录下的activemq.bat 文件启动服务。

　进入到ActiveMQ 安装目录的Bin 目录，linux 下输入 ./activemq start 启动activeMQ 服务。

　输入命令之后，会提示我们创建了一个进程IP 号，这时候说明服务已经成功启动了。



ActiveMQ默认启动时，启动了内置的jetty服务器，提供一个用于监控ActiveMQ的admin应用。 
　admin：http://127.0.0.1:8161/admin/

　我们在浏览器打开链接之后输入账号密码（这里和tomcat 服务器类似）

　　默认账号：admin

　　密码：admin

## 3、创建一个ActiveMQ工程













## Redis

##  

## Mongodb



## 分库分表