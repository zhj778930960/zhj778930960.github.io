---
layout:     post
title:      Mongoose
subtitle:   用于操控mongodb
date:       2020-04-04
author:     BY
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Node
    - Koa
---

# Mongoose

1.  与官网的`mongdb`插件一样的，都是用于连接数据库用的。
2. 比自带的会简便很多，用法更加简单

- 安装

  ````javascript
  npm install mongoose --save
  ````

  

- 使用

  ```javascript
  1.引入
  const mongoose = require("mongoose");
  
  2. 连接数据库
  mongoose.connect("数据库地址+具体的数据库名字")；//不清出的话 本地运行mongo 就可以看到
  //例如 mongodb://127.0.0.1:27017/itying  
  
  //地址： mongodb://127.0.0.1:27017/
  //数据库名  itying
  
  当然还要求这么配置  实际配置这么写： 后面这个对象是mongoose 要求的， 不然启动会报错
  
  mongoose.connect(
      "mongodb://127.0.0.1:27017/itying"，
      { useNewUrlParser: true,  useUnifiedTopology: true }
  })；
  
  
  
  
  3.定义表（集合）mongoose.Schema()
  //先要定义一个schema schema需要传入一个对象，里面设置一些属性，要与要操作的表中的字段对应
  
  //可以将Schema理解为为表结构的定义，类似于TS中的interface, 每一个schema会映射到mongodb中的一个集合（表）， 它不具备操作数据库的能力。只是一种表结构中字段的定义
  var userSchema = mongose.Schema({
      name: {
          type: String,
          required: true,
          default: "123"
      }
      
  })
  
  //想要获取数据库表的操作能力，需要用到模型model  model是由定义好的schema来生成的，意思就是说如果想生成model, 必须先有schema 
  
  4. 操作表（集合）mongoose.model()
  
  //mongoose.model()可以接受两个值，也可以接受三个值
  //设置模型名称时， 模型名称要首字母大写，并且要与数据库中的表（集合）名称对应
  
  /*   
    这个模型就回和数据库中表复数名称相同的建立连接  例如：数据库表users   model名称就要是User  
  这样才能建立连接
  */
     参数1：模型名称（首字母大写） 要与数据库的表（集合）要对应  ；
     参数2：Schema  例如上面定义好的userSchema;
     参数3： 表（集合）的名称， 指定要连接的表（集合）
  
     
    //这样就与指定的表admin建立起来了连接，可以操作表admin了
     var Admin = mongoose.model("Admin", userSchema, "admin")
     
     例如：查询数据
     router.get("/test", async ctx =>{
         //查询表中的数据
         let res = await Admin.find();
         console.log(res);
     });
  
     增加数据分为两步
        ①： 实例化model
        ②： 实例.save()方法增加数据
     
     router.post("/add", async ctx => {
         //先实例化model + 数据
         
         let user1 = new Admin({
             name: "张豪杰"
         });
         
         //通过实例化的save方法 增加数据
         
         user1.save((err) => {
              if (err) return console.log(err)；
              console.log("成功")
         })       
         
     })
  
  
  //数据更新
router.put("/update", async ctx => {
      //第一个参数为查询参数，查询到对应的数据，
    //第二个参数为这一条数据中，要修改的属性，和新的值
      //最后为一个回调函数
      
      //Admin 为mongoose.model()
      Admin.updateOne({"id": 123}, {"title": "woaini"}, (err) => {
          if (err) return ctx.body = "修改错误";
          ctx.body = "修改成功"
      })
  })
  
  //数据删除
  router.delete("/update", async ctx => {
      //第一个参数为查询参数，查询到对应的数据，并删除
      //最后为一个回调函数
      //Admin 为mongoose.model()
      
      Admin.deleteOne({"id": 123}, (err) => {
          if (err) return ctx.body = "删除错误";
          ctx.body = "成功"
      })
  })
  
  
    
  
  ```
  
  所以我们写代码的时候，可以将所有的表都封装在一个`db.js`(随便起名字)的文件中。然后将创建好的model 都导出
  
  需要用到的地方  自己导入model就可以了 

单独起一个文件

```javascript
const mongoose = require("mongoose"); //引入

const config = {
    dbUrl: "mongodb://127.0.0.1:27017/itying",
    dbUse: { useNewUrlParser: true,  useUnifiedTopology: true }
}
mongoose.connect(config.dbUrl, config.dbUse);

//定义Schema
let adminSchema = mongoose.Schema({
    name: {
        type: String,
        required: true
    }
})

let Admin = mongoose.model("Admin", adminSchema, "admin");

let userSchema = mongoose.Schema({
    name: {
        type: String,
        required: true
    }
})

let User = mongoose.model("User", userSchema, "user");

model.exports = {
    Admin,
    User
}

//在需要的地方，导入就可以了
```

####  数据修饰符

```javascript
let userSchema = mongoose.Schema({
     name: {
        type: String,
        default: "123",
        required: true,
        
        //修饰符trim  将name数据的左右空格都去除  "  123   "  => "123"
        trim: true,
        
        //将字符串变成大写
        uppercase: true,
        
        //将字符串变成小写
        lowercase: true
     }

})
```



#### 自定义修饰符

- `set`在设置数据，也就是存入数据的时候，自定义修饰符，来改变数据之后，进行存储。
- `get`不建议使用，其实是在new一个model实例之后，save()之前，去访问数据的时候，在前端获取数据的时候，将数据改变之后给前端  无实际意义， 用处不大

```javascript
还是在schema中去设置

let userSchema = mongoose.Schema({
     url: {
        type: String,
        required: true,
        default: "http://www.baidu.com",
        
        //放入一个set()方法 用于改变数据，其实就是跟对象中的setter 一样
        //存入数据时，将数据改变
        set(params){
           //params 就是存储数据的时候，传过来的值  方法最后需要return 
           //如果不以http://开头 我们就给加上 如果有 就不用管了
           if (!params) return "";
           if (params.startWith("http://")) return params;
           //如果没有 就拼接上
           return "http://" + params;
        }
     }

})
```



#### 增加索引，查询数据

```javascript
let userSchema = mongoose.Schema({
     url: {
        type: String,
        required: true,
        default: "http://www.baidu.com",
         
         //设置索引字段 加快查询速度，切记不要给所有的数据都加，只给重点的加就好了。因为会降低新增数据的时间
        index: true

     }

})
```



#### Schema的静态方法扩展， 使用在model上使用

````javascript
//  在Schema()上扩展静态方法， 在model()上就可以使用这个方法

let userSchema = mongoose.Schema({
     url: {
        type: String,
        required: true,
        default: "http://www.baidu.com",
     }
})
//自定义一个静态方法    userSchema.statics.xxx 来定义静态方法
userSchema.statics.findUrl = function(url, callback) {
     //this 指向Schema
     
     this.find({"url": url}, function(err, docs) {
         callback(err, docs)
     })
}


//怎么使用呢？？ 直接就可以访问model上面的方法

let UserModel = mongoose.model("user", userSchema, "user");

UserModel.findUrl("http://xxxx", function(err, docs) {
      if (err) return "xxx";
       ...
})

````





#### Schema 实例方法， 同样也是在model上使用

```javascript
//  在Schema()上扩展实例方法， 在model()上就可以使用这个方法

let userSchema = mongoose.Schema({
     url: {
        type: String,
        require: true,
        default: "http://www.baidu.com",
     }
})
//自定义一个实例方法    userSchema.methods.xxx 来定义实例方法
userSchema.methods.findUrl = function(url, callback) {
     //this 指向Schema
     
     this.find({"url": url}, function(err, docs) {
         callback(err, docs)
     })
}


//怎么使用呢？？ 因为是实例方法，所以必须是model 实例化之后才能访问的方法

let UserModel = mongoose.model("user", userSchema, "user");
let user = new UserModel({url: "123"})
user.findUrl("http://xxxx", function(err, docs) {
      if (err) return "xxx";
       ...
})
```



### 数据校验（内置验证器）

```javascript
//在Schema 中定义的数据 如果实例化模型时，多增加的属性  不会被增加到数据库 ，只有定义的会被增加

//如何对传入的数据进行校验呢？？？？

let userSchema = mongoose.Schema({
    name: {
         type: String, //传入的数据必须为string类型
         required: true, //默认为false  是否必须传入， 为true时， 必须传入这个参数
         enum: ["1","2"], //字符串那类型特有， 枚举值， 传入的值 只能是这里面的一个
         minlength: 10, //字符串特有， 传入字符串的最小长度限制
         maxlength: 100, //字符串特有， 传入字符串的最大长度限制
         match: /^\d{11}$/ // 验证字符串是否符合正则
    }，
    age: {
        type: Number, //传入的数据必须为number类型
        min: 0, // number类型特有  数据的最小值不能小于0 
        max: 150, //number类型特有  数据的最大值  不能大于150
        match: /^\d{11}$/, //验证数字是否符合正则
    }
})
```

#### 数据校验（自定义验证）

```javascript
let userSchema = mongoose.Schema({
    name: {
         type: String, //传入的数据必须为string类型
         required: true, //默认为false  是否必须传入， 为true时， 必须传入这个参数
         enum: ["1","2"], //字符串那类型特有， 枚举值， 传入的值 只能是这里面的一个
         minlength: 10, //字符串特有， 传入字符串的最小长度限制
         maxlength: 100, //字符串特有， 传入字符串的最大长度限制
         match: /^\d{11}$/, // 验证字符串是否符合正则
        
        //自定义校验 validate方法与用于自定义的校验
        validate： function(docs){
            return docs.length >= 10
         }
         
    }，
    age: {
        type: Number, //传入的数据必须为number类型
        min: 0, // number类型特有  数据的最小值不能小于0 
        max: 150, //number类型特有  数据的最大值  不能大于150
        match: /^\d{11}$/, //验证数字是否符合正则
    }
})
```

