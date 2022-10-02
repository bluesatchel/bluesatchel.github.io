---
title: springboot防御xss的方法学习记录
date: 2022-08-31 19:09:49
tags:
      - xss
      - java
      - springBoot
description: 对springBoot防御xss的一般方法的学习
---

大多数流程是:

1. 拦截请求
2. 重新包装请求
3. 重写**HttpServletRequest**中的获取参数的方法
4. 将获得的参数进行XSS处理
5. 拦截器放行

可以重写Filter的将request中的请求参数进行html实体编码

### 主要使用的方法HtmlUtils.htmlEscape

该方法可以将字符串中的一些特殊字符进行转义,进行转义处理后再存入数据库,这样前端请求之后获取到的字符串中可能会导致xss的特殊字符都会转义成html实体编码

![image-20220901174343546](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220901174343546.png)

### [mica-xss](https://gitee.com/596392912/mica/tree/master/mica-xss)

有一个特别好用的插件,`mica-xss`

##### 对应springboot版本信息

![image-20220831191455575](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220831191455575.png)

使用起来也及其简单,只需要两步

##### 1.导入依赖

由于我这里使用的springBoot版本为2.5.7,所以导入2.5.8版本

```xml
		<dependency>
            <groupId>net.dreamlu</groupId>
            <artifactId>mica-core</artifactId>
            <version>2.5.8</version>
        </dependency>
        <dependency>
            <groupId>net.dreamlu</groupId>
            <artifactId>mica-xss</artifactId>
            <version>2.5.8</version>
        </dependency>
```

##### 2.配置

导入依赖之后就已经可以运行了,可以在配置文件中进行一些简单的设置

| 配置项                         | 默认值 | 说明                                                         |
| ------------------------------ | ------ | ------------------------------------------------------------ |
| mica.xss.enabled               | true   | 开启xss                                                      |
| mica.xss.trim-text             | true   | 【全局】是否去除文本首尾空格                                 |
| mica.xss.mode                  | clear  | 模式：clear 清理（默认）、escape 转义、validate 校验（3.7.4新增） |
| mica.xss.pretty-print          | false  | `clear 专用` prettyPrint，默认关闭： 保留换行                |
| mica.xss.enable-escape         | false  | `clear 专用` 转义，默认关闭                                  |
| mica.xss.path-patterns         | `/**`  | 拦截的路由，例如: `/api/order/**`                            |
| mica.xss.path-exclude-patterns |        | 放行的路由，默认为空                                         |

同时也提供了对应的可以自定义过滤规则的方式
