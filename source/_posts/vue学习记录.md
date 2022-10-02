---
title: vue学习记录
date: 2022-07-18 20:38:40
tags:
      - vue
categories: vue
description: 写第一个vue项目时遇到的一些问题和解决方法
---

## vue学习记录

这篇文章主要用于自己开发[背忘录]项目中遇到的一些问题和解决方法

项目开发中使用的是vue 和element-ui

### 界面问题

在本项目中对于界面采用的是element-ui的container容器

在该组件中的`<el-main>`组件里嵌套`<router-view></router-view>`标签用来作为整个单页面应用（spa）的显示窗口

![image-20220718204211612](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220718204211612.png)

同时头部和footer都自定义组件并引入从而实现统一性,需要注意的是引入组件后标签需要采用小写和-的方式来替代驼峰命名

![image-20220718204332102](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220718204332102.png)

比如原来是`CommonHeader`现在就需要写成`<common-header></common-header>`

#### 默认页面

![image-20220718204613496](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220718204613496.png)

由于本项目是单页面应用,只需要通过路由跳转让主页面的`<router-view></router-view>`替换内容即可,但是由于`/`需要引入`Main.vue`这个总的布局模板,而`<router-view>`就写在`Main.vue`中,所以不能直接在`Main.vue`中写主页面的内容,这时候就需要给`/`这个路由添加一个重定向属性'redirect'让其重定向到`/`的子路由`/index`

### axios使用

先npm安装axios

在main.js中引入axios后还需要将其绑定到Vue的prototype属性上才能用

```vue
import axios from 'axios'
Vue.prototype.$axios=axios
```

或者不绑定prototype直接在每个需要使用axios的地方再引入一次

对于请求的格式,按照这样的格式来写,就可以在最后的then里面访问到返回的res和做赋值给return里面的变量值的操作

如果把`data`改为`params`就会将参数拼接到url后面,类似get传参,但是好像不会进行url编码

至于header头,则按需添加

```vue
axios({
        method: 'post',
        url: 'api',
        headers: { 'content-type': 'application/x-www-form-urlencoded' },
        data: this.$qs.stringify({
          q: "12312312"
        }),
      }).then((res) => {
        console.log(res)
      })
```



### axios配置跨域请求

仅适用于使用vue-cli创建的vue项目中

在vue.config.js文件中输入内容如下,并且之后请求的时候,使用`api`来替代target

```js
module.exports = {

  devServer: {

    proxy: {
      '/api': {
        target: 'https://openapi.youdao.com/api',
        //ws:true,
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }

      }

    }
  }
}
```

![image-20220719185901835](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220719185901835.png)

axios配置传统的post键值对传值方式

需要引入`qs`插件

在项目目录下`npm install qs`

在`main.js`中

```js
import qs from 'qs'
Vue.prototype.$qs = qs
```

接下来就可以使用`this.$qs.stringify({})`将值包裹再传递给data,这样发送的就不再是json数据了

![image-20220719190156526](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220719190156526.png)

### 在vue中使用md5等哈希函数

直接npm install js-md5 , js-sha256等

一样绑定到prototype上

```js
Vue.prototype.$md5 = md5
Vue.prototype.$sha256 = sha256
```

调用的时候直接,

```js
this.$md5(plaintext)
this.$sha256()
```

### vue项目打包,生成dist文件

由于是使用vue-cli生成的项目

所以需要在根目录下面新建vue.config.js

不过前面配置跨域的时候弄过了,所以直接加一段进去就行

```js
module.exports = {
  assetsDir: 'static',
  parallel: false,
  publicPath: './',
}
```

然后直接`npm run build`

### 通过iframe内嵌B站视频的问题

需要给链接`src`加上`https://`,否则请求路径有问题



### 配置按键对应事件

main.js

```js
Vue.prototype.$keyBoard = function (vm, methodName, code) {
  document.onkeydown = function () {
    let key = window.event.keyCode;
    if (key == code) {
      vm[methodName](code); // 触发methodName事件
    }
  };
}

```

具体的vue代码中

```vue
mounted() {
    this.$keyBoard(this, 'onClickEnter', 13)     //13是enter按键  其他按键码自己查
}


methods: {
    onClickEnter() {

		//按下按键后要实现的代码写在这里

	},    //这里使用绑定的按键事件
}

```



### vue中页面布局需要刷新才正常显示的问题

其实这个问题的起因是自己前端代码写的太烂,只需要在style添加scoped即可

因为每个vue文件都有自己对应的css样式。
因此需要在每个页面的style标签上加上scoped属性以表示它的样式仅对于当前页面生效。否则在从别的页面回退或跳转时因为相同的class 样式而导致冲突



