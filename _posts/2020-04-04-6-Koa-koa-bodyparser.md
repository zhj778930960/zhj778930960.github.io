---
layout:     post
title:      koa-bodyparser
subtitle:   路由请求
date:       2020-04-04
author:     BY
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Node
    - Koa
---

# `koa-bodyparser`

如何接收`post`请求带过来的数据，有两种方式

- 利用`node.js`原生的`querystring.parse()`方式拿到前端传过来的数据。

  - ```javascript
    const http = require("http");
    http.createServer((req, res) => {
        if (req.method === "POST"){
            let params = ""
            req.on("data", (chunk) => {
                params += chunk;
            });
            
            req.on("end", () => {
               let data = querystring.parse(params);
          })
        }
    })
    ```
    
    

- 利用第三方中间件`koa-bodyparser`。

##### `koa-bodyparser`

1. 安装

   ```javascript
   npm install koa-bodyparser --save
   ```

2. 使用

   ```javascript
   const Koa = require("koa");
   const app = new Koa();
   
   const router = require("koa-router")();
   const bodyParser = require("koa-bodyparser");
   
   var views = require('koa-views')
   app.use(bodyParser())
   
   
   //ejs 第三方模板渲染引擎
   app.use(views('views',{
       extension:'ejs'
   }))
   
   router.get('/',async (ctx)=>{
       await ctx.render('index'); //index 就是文件的名称
   })
   
   router.post("/new", async (ctx) => {
       console.log(ctx)
       ctx.body = ctx.request.body
   })
   
   
   app.listen(3000)
   app.use(router.routes());
   app.use(router.allowedMethods());
   ```

   