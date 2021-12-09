---
title: hexo多设备操作
date: 2021-12-04 02:03:57
tags: 
    - hexo 
    - github
    - github公钥设置
categories: hexo
typora-root-url: ..
---

在解决掉hexo图片路径的问题后,着手解决hexo多设备更新文章的问题<!--more-->

首先在新设备上面安装Node.js和Git

接着在项目中新建一个分支,或者将main分支改名为source分支并设置为默认分支

![image-20211204031522619](/images/hexo%E5%A4%9A%E8%AE%BE%E5%A4%87%E6%93%8D%E4%BD%9C/image-20211204031522619.png)

确保该分支中文件为空

新建一个hexo文件夹,在该文件夹内打开git bash,生成SSH公钥添加到Github

`git config --global user.name "yourname"`

`git config --global user.email "youremail"`

可以用这两条命令检查有没有输对

`git config user.name`

`git config user.email`

确保正确后

输入`ssh-keygen -t rsa -C "youremail"`

会提示已经生成SSH秘钥,打开提示的文件夹,打开id_rsa.pub(公钥),复制内容在`头像->setting->SSH and GPG keys`下点击`New SSH key`,将复制的公钥粘贴进去,即完成了SSH公钥的添加

在父目录打开git bash

`git clone git@github.com:yourname/yourname.github.io.git hexo` 

就会拉取source的文件到hexo文件夹下,cd hexo目录,删除所有除.git以外的文件

接着将原来设备中的Hexo目录下的所有文件复制新建的hexo目录

接着安装hexo 

`npm install -g hexo cli`

安装一些依赖,不知道哪些有用,反正都装上吧

```bash
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save
npm install hexo-generator-search --save
npm install hexo-generator-searchdb --save
```

接下来在有.git文件夹的目录下打开git bash

开始准备push文件,提交前将.gitignore文件中的node_modules依赖库删掉,把它也传上去

`git init`

将当前目录下的文件放到暂存区(上膛)

`git add .`

可以用`git status`查看当前文件状态

接着用`git commit -m "本次提交的描述信息"`提交描述信息

因为之前已经拉取过,所以这句可以省略`git remote add origin "远程仓库地址"`

`git push origin source`这样就成功将文件push到了source目录上面

提交时考虑以下注意将themes目录以内中的主题的.git目录删除（如果有），因为一个git仓库中不能包含另一个git仓库，否则提交主题文件夹会失败

提交成功之后,source分支就是保存博客的部署文件,让自己更新维护,master保存博客的静态页面,提供访问

,新电脑只需要拉取source分支并安装需要的软件就能写博客了

需要注意的点是,每当需要切换设备更新博客时候,一定要先将原设备中source分支下的内容push到github上,再由新设备拉取才能撰写博客push到master分支

如果不嫌麻烦的话,每次写完文章

git bash 五连击

```bash
hexo g
hexo d
git add .
git commit -m "new page"
git push origin source
```

换设备之后只需要在hexo目录下git pull即可写文章了
