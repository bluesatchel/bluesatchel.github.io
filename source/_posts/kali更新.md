---
title: kali更新
date: 2021-01-13 18:16:15
tags:
      - kail
      - linux
categories: linux
---

kali更新依次使用即可更新前先更换源/etc/apt/sources.list

- 中科大

```
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
```

- 阿里云

```
deb http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
deb-src http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
```

- 清华大学

```
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
```

apt-get update & apt-get upgrade当出现正在读取软件包列表 . . .完成 ，画面长时间未变动时，回车执行下一条语句（若如正常执行完毕更好）

apt-get dist-upgrade在执行即将结束时会有一次选择，选择是否选择安装新的还是维持旧的，输入Y，安装新的。结束后安装完毕，可再一次执行第一条命令，检查是否所有的安装包是否全部安装。

apt-get clean此时已经全部安装完毕，执行下面的命令(可复制)可查看新系统版本
查看系统版本 命令：lsb_release -a

查看内核版本 命令：uname -r
