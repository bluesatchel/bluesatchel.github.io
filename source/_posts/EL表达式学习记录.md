---
title: EL表达式学习记录
date: 2021-09-22 18:25:41
tags:
      - el表达式
      - jsp
categories: java
---

# EL表达式

EL(expression language)是为了让JSP写起来更加方便，他提供了在jsp中简化表达式的方法，让jsp的代码更加简化

## EL表达式语法

语法结构较为简单,就是${expression}

## 域的概念

域对象的概念在jsp中共有4中,分别是`pageContext,request,session,application`  范围依次是: `本页面 ->一次请求->一次会话->整个应用程序`

el表达式一般操作的都是域对象,操作不了局部变量

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>EL表达式</title>
</head>
<body>
<%
    pageContext.setAttribute("username","zhangsan");
    request.setAttribute("username","lisi");
    session.setAttribute("username","wangwu");
    application.setAttribute("username","zhaoliu");
%>
<pre>
        获取作用域中username： ${username}<br><%-- 默认从小到大的范围中找，找到的第一个返回 --%>
        不在作用域中的： ${password}
        <%--获取request作用域中的username： ${requestScope.username}
        获取session作用域中的username： ${sessionScope.username}
        获取application作用域中的username： ${applicationScope.username}--%>
  </pre>
</body>
</html>
```

![image-20220922183714222](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220922183714222.png)

不在作用域中,显示为空,而不是null

### 从指定范围取值,以及默认取值规则

- 当需要从某个特定的域对象中查找数据的时候可以使用四个域对象对应的空间对象分别为:

  `pageScope,requestScope,sessionScope,applicationScope`

- EL默认的查找方式为:从小到大查找,找到了即返回,若未查找到则返回空字符串

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>EL表达式</title>
</head>
<body>
<%
    pageContext.setAttribute("username","zhangsan");
    request.setAttribute("username","lisi");
    session.setAttribute("username","wangwu");
    application.setAttribute("username","zhaoliu");

%>
<pre>
         获取pageContext作用域中的username:  ${pageScope.username}
         获取request作用域中的username： ${requestScope.username}
         获取session作用域中的username： ${sessionScope.username}
         获取application作用域中的username： ${applicationScope.username}
  </pre>
</body>
</html>
```

![image-20220922184459693](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220922184459693.png)



#### 获取作用域中的集合

```jsp
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>EL表达式</title>
</head>
<body>
  <%
      List<String> list=new ArrayList<String>();
      list.add("aaa");
      list.add("bbb");
      list.add("ccc");
      request.setAttribute("list",list);
  %>
  <pre>
           获取list中指定下标的数据：${list[1]}--${list[2]}
           获取集合的长度：${list.size()}
           list代表的是存在域对象中的变量名(限域变量名)
  </pre>
</body>
</html>
```

![image-20220922184636850](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220922184636850.png)

#### 获取javaBean对象

```jsp
<%@ page import="com.test.User" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>EL表达式</title>
</head>
<body>
<%
    User user=new User();
    user.setUsername("zhangsan");
    user.setSex(true);
    user.setUserId(1);
    request.setAttribute("user",user);//设置域对象属性
%>
<pre>
         获取JavaBean中的username  ${user.username}
         获取JavaBean中的userId    ${user.userId}
         获取JavaBean中的sex       ${user.sex}
  </pre>
</body>
</html>
```

![image-20220922185046439](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220922185046439.png)

## EL表达式运算

#### 判断是否为空

![image-20220922185231903](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220922185231903.png)

#### 判断是否相等

![image-20220922185319138](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220922185319138.png)

#### 算数运算

![image-20220922185339887](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220922185339887.png)

#### 大小比较

![image-20220922185400875](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220922185400875.png)
