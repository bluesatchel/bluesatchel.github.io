---
title: 关于blog迁移到不同设备
date: 2024-09-29 14:08:20
tags:
	  -hexo
categories: hexo
---

#### 安装nodejs和git

这里没啥问题一般

安装好git之后需要配置一下用户名和电子邮件用于后续commit

```bash
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱地址"
```



#### 配置ssh秘钥

如果已经有秘钥了则跳过此步骤,否则在命令行输入如下

```bash
ssh-keygen -t rsa -b 4096
```

生成的秘钥在

`C:\Users\用户名\.ssh`

复制公钥`.pub`添加到github的`settings->SSH and GPG keys`位置的ssh秘钥处即可

#### 克隆仓库到本地

没啥问题

#### 用npm安装js的模块

##### 先给npm换源

```bash
npm config set registry https://registry.npmmirror.com
//验证是否成功
npm config get registry
```

##### 安装hexo所需的模块

```bash
npm install
```

hexo命令需要添加到环境变量path中

把`hexo\node_modules\.bin`添加到环境变量path中

这里也可以全局安装hexo

```bash
npm install hexo-cli -g
```



到这里一般就成功了,后序有啥问题再记录一下