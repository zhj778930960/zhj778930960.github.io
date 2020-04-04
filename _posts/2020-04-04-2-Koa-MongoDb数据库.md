---
layout:     post
title:      MongoDb 数据库
subtitle:   数据库
date:       2020-04-04
author:     BY
header-img: img/post-bg-iWatch.jpg
catalog: true
tags:
    - Node
    - Koa
---

# `MongoDb `数据库

1. 安装

   ```javascript
   //Mongdb 网站下载 工具
   www.mongodb.com
   ```

   - 下载完成后选择`custom`自定义，然后选择安装的目录即可。
   - 安装时间较长，请耐心等待

2. 环境配置

   - 右键我的电脑  -  高级设置  -  高级 -  环境变量  -  系统变量PATH
   - 将`mongodb`所在`bin`目录路径放到变量里面去就好了

3. 启动

   - 在任意盘新建一个文件夹

   - 运行 在`cmd` 界面 `mongod --dbpath  文件路径+名称`  这样就启动成功了
   - 怎么访问，如果自己的电脑上既是服务端，也是客户端，直接`mongo` 开启访问就可以了
   - 如果需要访问另一个地址的 就需要`mongo  ip地址`

4. 命令

   1. 查看数据库数据 `show dbs`  就可以看当前有哪些数据库

   2. 进入已经存在的数据库 `use  xxx(数据库名称)`  就可以进入到选择的数据库之中了

   3. 创建：

      - 先使用一个不存在数据库 `use itying` 这样就是会提示你这个数据库不存在，没关系
      - 然后 `db.user.insert({"name": "123"})`  这样就是既创建了一个`itying` `数据库，也创建了一个`user`表，最后还给里面加入了一条数据
      - 展示所有的表 `show collections`  当前数据库中有哪些表

   4. 查询

      我这里默认使用`user`这个表，当然你可以建立别的表，不影响啥

      - 查询所有记录 `db.user.find()`  查询`user`表之中所有的数据
      - 查询指定数据`db.user.find({"age": 22})`查找年龄为22的所有数据
      - 查询一个 范围 例如 `db.user.find({"age": {$gt: 20}})`查找年龄大于20的数据
      - 查询一个 范围 例如 `db.user.find({"age": {$lt: 20}})`查找年龄小于20的数据
      - 查询一个 范围 例如 `db.user.find({"age": {$gte: 20}})`查找年龄大于等于20的数据
      - 查询一个 范围 例如 `db.user.find({"age": {$lte: 20}})`查找年龄小于等于20的数据
      - 查询多个条件，里面就需要多个值`db.user.find({"age": 20, name: "张三"})`
      - 查询一个中间范围 `db.user.find({"age": {$gt:20, $lt:35})`查找一个数据大于20， 小于23
      - **模糊查询  `db.user.find({"name": /san/})`查询名字之中带`san`这个字符的数据**
      - **查询以什么开头的   `db.user.find({"name":/^zh/})`以`zh`开头的`name`数据都会被查询出来** 
      - **查询指定列的数据 `db.user.find({}, {"age":1})`**   按照正序排列  `-1`按降序排列
      - 只查询前几条数据 `db.user.find().limit(5)`  查询到前五条数据
      - 查询指定位置的数据 第几条数据   `db.user.find().skip(2)` 跳过前两条数据，从第三条开始查
      - `or`条件查询，两个条件满足一个即可  `db.user.find({$or:[{"age": 20}, {"age": 24}]})` 查询年龄为`20`或者为`24`的数据
      - 查询表中所有数据的数量  做分页时可以返回给前端 `db.user.find().count()`  
      - **上面的条件可以任意组合的查询，都是可以的**

   5. 修改

      - 删除`user`表   `db.user.drop()`  这样就把当前数据库中`user`表删除了 
      - 删除当前所在的数据库 `db.dropDatabase()` 就把当前所在的数据库就给删除了
      - 删除某一条数据 `db.user.remove({"id": 123})`将`id`等于123的数据删除掉
        - 只删除一条，如果有多条的话 `db.user.remove({"name": "wangmazi"}, {justOne: true})`

   6. 修改

      - 修改数据  `db.user.update({"name": "张三"}, {$set: {"age": 33}})` 将`name`为张三的这条数据的`age`属性变为33

   7. 清屏

      - `cls`命令可以清屏

