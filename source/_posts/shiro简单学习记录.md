---
title: shiro简单学习记录
date: 2022-09-27 15:47:02
tags:	
      - shiro
      - java
categories: java
---

先来一个简单的登录认证小例子

编一个ini文件模拟数据库中获取的数据

![image-20220927171547699](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220927171547699.png)

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.subject.Subject;

public class ShiroRun {

    public static void main(String[] args) {
        //1.初始化获取SecurityManager
        IniSecurityManagerFactory factory =new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager=factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        //2.获取subject对象
        Subject subject=SecurityUtils.getSubject();
        //3.创建token对象,web应用用户名密码从页面传递
        AuthenticationToken token=new UsernamePasswordToken("zhangsan","123");
        //4.完成登录
        try {
            subject.login(token);
            System.out.println("登录成功");
        } catch (AuthenticationException e) {
            System.out.println("登录失败");
            e.printStackTrace();
        }
        //异常还有很多种,这里笼统的写一个就行了
    }
}

```

### 授权概念

1.授权,也叫访问控制,即在应用中控制谁访问哪些资源(如访问页面/编辑数据/页面操作等).在授权中需了解的几个关键对象:主体(subject),资源(Resource),权限(permission),角色(role)

2.主体(Subject):访问应用的用户,在Shiro中使用subject代表该用户,用户只有授权后才能访问相应的资源

3.资源(Resource):在应用中用户可以访问的URL,比如访问JSP页面,查看/编辑某些数据,访问某个业务方法,打印文本等等都是资源,用户需要授权后才能访问

3.权限(permission):安全策略中的原子授权单位,通过权限我们可以表示在应用中用户有没有操作某个资源的权利,即权限表示在应用中用户能不能访问某个资源,如:访问用户列表页面查看/新增/修改/删除用户数据(即很多时候都是CRUD)增删改查式权限控制等

5.shiro支持粗粒度权限(如用户模块的所有权限)和细粒度权限(操作某个用户的权限,即实例级别的)

6.角色(role):权限的集合,一般情况下会赋予用户角色而不是权限,即这样用户可以拥有一组权限,赋予权限时比较方便,典型的如:项目经理,技术总监,CTO,开发工程师等都是角色,不同的角色拥有一组不同的权限

#### 授权方式

##### 1.编程方式

通过写if/else授权代码块完成

```java
if(subject.hasRole("admin")){
    //有权限
    
}else{
    //无权限
}
```

##### 2.注解式

通过在执行的java方法上防止相应的注解完成,没有权限将抛出相应的已换成那个

```java
@RequireRoles("admin")
public void hello(){
}
```

##### 3.JSP/GSP标签

在jsp/gsp页面通过相应的标签完成

```jsp
<shiro:hasRole name="admin">
    有权限
</shiro:hasRole>
```

#### 授权流程

subject.isPermitted/hasRole





### 自定义登录认证

### 与SpringBoot整合

3.1框架整合
