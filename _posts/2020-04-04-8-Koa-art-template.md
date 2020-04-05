---
layout:     post
title:      art-template
subtitle:   渲染模板
date:       2020-04-04
author:     BY
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Node
    - Koa
---

# `art-template`模板引擎

优点：

- `art-template`支持`ejs`的语法
- 也可以使用自己的类似`vue`的语法 {{变量}}

#### 如何使用？

1. 安装

   ```javascript
   //需要安装两个模块
   npm install art-template --save  //先安装art-template
   npm install koa-art-template --save //再安装koa-art-template
   ```

2. 使用

   ```javascript
   const Koa = require("koa");
   const app = new Koa();
   const router = require("koa-router")();
   
   //1   引入koa-art-template 模块
   const render = require("koa-art-template")
   const path = require("path");
   
   //2 配置render函数， 也就是配模板的基本信息
   
   render(app, {
       root: path.join(__dirname, "views"), //配置文件的所在路径，在当前文件内的views目录下
       extname: ".html", //文件后缀名称
       debug: process.env.NODE_ENV !== "production" //是否开启调试模式， 不在生产环境开启调试， 意思就是在开发环境下开启调试模式
   });
   
   3.// 监听接口 将模板渲染到浏览器上
   router.get("/news", async (ctx) => {
       await ctx.render("index"); //index为views文件夹下的index.html  这里index就是文件夹的名称
   })
   
   4. //如何在html运用变量显示，如何将值传入到html中
   router.get("/content", async (ctx) => {
       await ctx.render("content", {
            //要传值给content.html的值
           name: "张三",
           content: "你好啊",
           h: "<h1>你怎么来了呢</h1>"
       })
   })
   
   
   5. //html中如何显示传过去的值呢？？  跟vue一样就可以了 不过有一点差别是显示传过去的dom节点
   
        正常显示  {{ name }}   {{ content }}
        显示dom节点  {{ @h }}
   
   //常规的 开启服务
   
   app.listen(3000, () => {
       //开启服务成功
   });
   
   //开启路由
   app.use(router.routes());
   app.use(router.allowedMethods());
   
   ```
