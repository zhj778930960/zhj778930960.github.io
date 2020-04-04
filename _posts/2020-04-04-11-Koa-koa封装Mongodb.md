---
layout:     post
title:      Koa 封装 Mongodb
subtitle:   
date:       2020-04-04
author:     BY
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Node
    - Koa
---

# ` Koa`封装`Mongodb`

######       准备工作

  **单例模式**： 就是在需要多次实例化一个类的时候，其实内部只实例化了一次的方式

1.  利用`es6`的类， 结合单例模式，实例化的时候，不用重复的实例化

   ```javascript
   class Person {
       //静态方法的标记
       static getInstance(){
           
           //判断类的静态属性中有没有这个属性，如果没有，就将实例化传给这个属性
           if (!Person.instance){
               Person.instance = new Person();
           }
           //如果有 就这返回这个属性（也就是这个实例）， 如果没有 就执行上面的判断之后再次进行返回
           return Person.instance
       }
      
       constructor(){
           //在实例化的时候，可以执行开启数据库的功能
           this.openMongodb()
       }
       
       openMongodb(){
           //开启数据库
       }
   }
   
   const p1 = new Person();
   const p2 = new Person();
   
   注意： 上面两次执行实例化的操作的时候， 会开启两次服务器， 比较浪费性能，那我们怎么实例化的时候只执行一次数据库的开启就行了
         所以需要用到class的静态方法去判断就可以
   
   /*最后我们在实例化的时候，只需要执行实例化中的方法就可以了 不用直接去实例化 
   这样就自会执行一次实例化，服务也只用开启一次就好了 */
         
         const p1 = Person.getInstance();
         const p2 = Person.getInstance();
   
   ```

   

2. `mongodb`简单运用

```javascript
1. 需要下载mongodb的依赖包
  npm install mongodb --save
  
2. 引入mongodb 并且调用其中的MongoClient

const MongoClient = require("mongodb").MongoClient;

3. 设置数据库地址常量 和 数据库名称常量

//如果不知道自己电脑的数据库地址，就运行一下，在第一行就会看到地址
const dbUrl = 'mongodb://127.0.0.1:27017/';
//设置要访问的数据库名称常量
const dbName = "dataValue";

4.连接数据库
MongoClient.connect(dbUrl, (err, client) => {
    //先判断是否连接失败
    if(err) return "数据库连接失败";
    
    //利用返回的client对象，获取具体的数据库db方法  
    const db = client.db(dbName);
    
    //查询某个表中的数据
    let result = db.collection("local").find();
    //把拿到的数据转为数组传回来
    result.toArray((err, docs) => {
        console.log(docs)
    })
    
    //新增数据
    db.collection("local").insertOne({"name": "123"}, (err, result) => {
         //新增之后的回调
        
        //最后一定记得要关闭库的连接
        client.close()
    })
})
```

思考： 那我是不是每次访问数据库的时候，都要去连接一次，这样会不会太麻烦了，要怎么简化这个请求过程和时间呢？？？？

3. 将上述两者结合  封装一个`mongodb`类库

   - 先定义一个`config.js`文件，配置一些基本信息

     ```javascript
     //配置项目类库  简单的先配置一下那个数据库的名称和地址
     const app = {
          dbUrl: "mongodb://127.0.0.1:27017/",
          dbName: "koa"
     }
     
     //导出
     module.exports = app;
     ```

   - 然后建立一个`db.js`来封装一个`mongodb`库

     ```javascript
     //db库
     
     //引入config.js的配置
     
     const Config = require("./config");
     
     //引入 MongoClient
     
     const MongoClient = require("mongodb").MongoClient;
     
     //定义Db类
     
     class Db {
       //设计一个单例
       static getInstanse() {
         if (!Db.instanse) {
           Db.instanse = new Db();
         }
         return Db.instanse;
       }
     
       //构造函数
       constructor() {
         //初始化的时候 连接数据库
         this.dbClient = "";
         this.connect();
       }
     
       //定义一些方法，像新增，读取值等操作
     
       //拦截数据库
       connect() {
         return new Promise((resolve, reject) => {
           MongoClient.connect(Config.dbUrl, (err, client) => {
             if (err) return reject(err);
             resolve(client.db(Config.dbName));
           });
         });
       }
     
       find(name, value) {
         return new Promise((resolve, reject) => {
           this.connect()
             .then(db => {
               let result = db.collection(name).find(value);
               result.toArray((err, docs) => {
                 if (err) reject(err);
                 resolve(docs);
               });
             })
             .catch(err => {
               reject(err);
             });
         });
       }
     
       update() {}
     
       insert() {}
     }
     
     // var db1 =  new Db();
     
     // db1.find("local",{}).then(res => {
     //     console.log(res)
     // })
     
     // 有个问题 如果我需要实例化多个，这样每次都会执行一次构造函数，里面的方法，这样很消耗性能
     
     //所以我需要利用单例模式，多次实例化，只需要连接一次数据库就可以了
     
     //以后不需要直接去实例化Db  只需要访问其静态方法就可以了
     
     var db2 = Db.getInstanse();
     db2.find("local", {}).then(res => {
       console.log(res);
     });
     
     
     //最后 将 类`Db`导出
     
     module.exports = Db.getInstance();
     ```

   - 最后在另外别的文件导入这个`db`类  直接使用就可以了

     ```javascript
     const Koa = require("koa");
     
     const app = new Koa();
     
     const router = require("koa-router");
     
     //引入db类库
     
     const Db = require("./db");
     
     router.get("/", async (ctx) => {
         Db.find().then((result) => {
            console.log(result);
         })
         //或者
     
         let res = await Db.find();
     })
     
     app.listen(3000);
     app.use(router.routes());
     app.use(router.allowedMethods());
     ```

     

