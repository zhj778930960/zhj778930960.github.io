---
layout:     post
title:      koa-multer
subtitle:   用户上传文件
date:       2020-04-04
author:     BY
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Node
    - Koa
---

# `@koa/multer`

1. 什么是`koa-multer`

    `koa-multer` 是用于文件上传的，当客户端，选择文件进行上传的时候，可以利用这个中间件，接收到上传的文件

2. 安装

   ```javascript
   // @koa/multer 是基于 multer开发的 这里我们也需要引入他 
   
   npm install --save @koa/multer multer
   ```

3. 使用， 官网示例代码

   ```javascript
   const Koa = require("koa");
   const app = new Koa();
   //引入
   const multer = require("@koa/multer");
   //使用中间件
   
   
   //option 的配置有几个属性，需要了解一下
    const option = {
        dest: "./uploads/", //指定文件存储的位置  文件保存在内存中，并不会写入磁盘
        filFilter: "", //文件过滤器，控制哪些文件可以被接收
    }
    
    
    //推荐使用下面的storage 的方式，  这样更加强大 multer.diskStorage({}) 方式可以直接将文件存储到磁盘中  可控制磁盘存储引擎
    
    const storage = multer.disStorage({
          destination: function(req, file, cb) {
              //回调函数 固定用法
              cb(null, "要存储到的地址")
          },
          filename: function(req, file, cb) {
            //这里可以设置 上传之后的文件的名称，我们为了重复就给这里设置了时间戳
            cb(null, file.filename + "_" + Date.now())
          }
    })
    
    const limits = {
            // 对上传的文件做一些限制，
            fields: 10, //非文字字段的数量
            fileSize: 500 * 1024, //文件夹大小的限制， 单位b
            files: 1  //文件数量的限制
    }
    
    
    //执行multer  得到其内部的返回，就可以的到其，需要定制好的上传方法
       const upload = multer({
              storage,
               limits
       })
       
       
       
    //上传之后 我们都可以拿到哪些文件的信息 也就是ctx.file 和 ctx.files 中的信息是什么？
       
       以多文件上传举例 我们得到的其实就是一个数组 ctx.files
        [
            { 
                fieldname: 'file', //上传时，对应的接口key
                originalname: 'xm.jpg', //上传前文件，也就是用户选择时文件的名称+后缀
                encoding: '7bit', //文件的编码方式
                mimetype: 'image/jpeg', //文件的类型
                destination: './static/images', //文件存储的位置
                filename: 'file-1582378596464', //存储后，名字改变，改变之后，文件的名称
                path: 'static\\images\\file-1582378596464', //文件在磁盘上存储的路径
                size: 961027  //文件大小
            }
           
       ]
       
       
       
   ```

4. 单文件上传`upload.single(fieldname)`

   ```javascript
   //"file" 是前端在上窜你的时候，定义的接口字段key  value 就是文件， 这样才能获取到对应的文件
   
   1. 先引入
   
   const Koa = require("koa");
   const app = new Koa();
   const router = require("koa-router")();
   const render = require("koa-art-template");
   const multer = require("@koa/multer");
   const path = require("path");
   render(app, {
       root: path.join(__dirname, "views"),
       extname: ".html",
       debug: process.env.NODE_DEV === "develop"
   })
   
   
   2. 配置上传设置
     const storage = multer.diskStorage({
         //配置存储地址
          destination: function(req, file, cb) {
              cb(null, "./static/")
          },
           //配置上传后的文件名
          filename: function(req, file, cb) {
                cb(null, `${Date.now()}_${file.originalname}`)
          }
     })
     const limits = {
          // 对上传的文件做一些限制，
            fields: 10, //非文字字段的数量
            fileSize: 500 * 1024, //文件夹大小的限制， 单位b
            files: 1  //文件数量的限制
     }
     
     const upload = multer({
           storage,
           limits
     })
     
   3. 调用接口
   
    router.post("/upload",upload,single("avatar"), async ctx => {
          console.log(ctx.file)
        ctx.body = {
             code: 0,
             msg: "成功"
        }
    })
   
   
   app.listen(5000);
   app.use(router.routes()).use(router.allowedMethods())
   
   //html的代码
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Document</title>
   </head>
   <body>
         <form action="/upload" method="post" enctype="multipart/form-data">
           <input name="avatar" type="file">
            <button type="submit">提交</button>
         </form>
   </body>
   </html>
   
   ```

5. 多文件上传`upload.array(fieldname, maxCount)`  一个上传组件， 可以同时上传多个文件

    ```javascript
    const Koa = require("koa");
    const app = new Koa();
    const router = require("koa-router")();
    const path = require("path");
    const multer = require("@koa/multer");
    const storage = multer.diskStorage({
         destination: (req, file, cb) => {
              cb(null, "./static/")
         },
        filename: (req, file, cb) => {
             cb(null, `${Date.now()}-${file.originalname}`)
        }
    })
    
    const limits = {
         fields: 10， //其实就是在配置，有几个可以上传的文件按钮，如果有多个，就需要用到upload.fields([]) 来上传， 如果有一个 用 upload.single("") 或者 upload.array("", maxcont)
         fileSize: 500 * 1024 * 1024, //文件大小
         files: 3, //文件数量
    }
    
    //如果文件不符合上面limits 中的任何一个条件 就会报错
    
    
    const upload = multer({
         storage,
         limits
    })
    //upload.array(fieldsname, maxCount)第二个参数如果，配置了  还是以limits 中的files 为优先
    
    //limits.files > maxCount  
    //3 优先于10
    
    router.post("/upload", upload.array("avatar", 10), async ctx = > {
         ctx.body = ctx.files
    })
          
          
    app.listen(5000);
    app.use(router.routes()).use(router.allowedMethods())      
    
    
    
    //html的代码
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
    </head>
    <body>
          <form action="/upload" method="post" enctype="multipart/form-data">
            <input name="avatar" type="file" multiple="multiple">
             <button type="submit">提交</button>
          </form>
    </body>
    </html>
    
    ```

6.  `upload.fields([])`  存在多个上传组件，可同时上传多个文件

     

    ```javascript
    const Koa = require("koa");
    const app = new Koa();
    const router = require("koa-router")();
    const path = require("path");
    const multer = require("@koa/multer");
    const storage = multer.diskStorage({
         destination: (req, file, cb) => {
              cb(null, "./static/")
         },
        filename: (req, file, cb) => {
             cb(null, `${Date.now()}-${file.originalname}`)
        }
    })
    
    const limits = {
         fields: 10， //其实就是在配置，有几个可以上传的文件按钮，如果有多个，就需要用到upload.fields([]) 来上传， 如果有一个 用 upload.single("") 或者 upload.array("", maxcont)
         fileSize: 500 * 1024 * 1024, //文件大小
         files: 3, //文件数量
    }
    
    //如果文件不符合上面limits 中的任何一个条件 就会报错
    
    
    const upload = multer({
         storage,
         limits
    })
    //upload.array(fieldsname, maxCount)第二个参数如果，配置了  还是以limits 中的files 为优先
    
    //limits.files > maxCount
    //3 优先于10
    
    //avatar 在form-data 格式文件中作为 接口的key值   value 对应文件
    const options = [
         {
             name: "avatar",
             maxCount: 10
         },
         {
              name: "avatar",
              maxCount: 8
         }
    ]
    
    router.post("/upload", upload.fields(options), async ctx = > {
         ctx.body = ctx.files
    })
          
          
    app.listen(5000);
    app.use(router.routes()).use(router.allowedMethods())      
    
    
    
    //html的代码
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
    </head>
    <body>
          <form action="/upload" method="post" enctype="multipart/form-data">
            <input name="avatar" type="file" multiple="multiple">
            <input name="zhj" type="file" multiple="multiple">
             <button type="submit">提交</button>
          </form>
    </body>
    </html>
    ```

    

