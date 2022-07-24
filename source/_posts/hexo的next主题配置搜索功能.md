---
title: hexo的next主题配置搜索功能
date: 2021-12-04 23:39:33
tags:
    -hexo
    -Next主题搜索功能
categories: hexo
---

1.安装插件

`npm install hexo-generator-searchdb --save`

2.修改next主题的_config.yml配置文件,找到local_search,将其中的enable设置为true

3.将下列参数添加到hexo的配置文件中(_config.yml)

hexo的搜索功能是基于public下的search.xml文件实现的

```
search:
  path: search.xml   #在public目录的根目录下生成search.xml 文件，用于存储网站文章的文字数据.
  field: post
  format: html
  limit: 10000
```

然后

```
hexo clean
hexo g
hexo d
```

即可在线使用搜索功能

#### 注意事项:

安装searchdb插件后

如果不配置第三部的参数,在在线页面中会出现一直转圈的搜索框

