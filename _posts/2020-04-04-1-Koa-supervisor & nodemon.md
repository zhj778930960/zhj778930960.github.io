---
layout:     post
title:      Supervisor & Nodemon
subtitle:   启动插件
date:       2020-04-04
author:     BY
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - Node
    - Koa
---

# `Supervisor `& `Nodemon`

当`js`文件有更新的时候，会立马重新启动服务，相当于`webpack`配置项目中的热更新功能.

这个是`node.js`自带的包，安装完成之后，就直接可以用了.

1. 安装

   ```javascript
   npm install supervisor -g  //安装到全局 所有的项目都可以使用
   ```

   

2. 使用

   ```javascript
   supervisor xxx.js  //代替 node xxx.js
   ```

   

3. 注意

   安装完之后， 有时候，`vscode`需要重启一下，才能生效这个命令。





#### 与`supervisor`相同 不过更推荐使用`nodemon`

````javascript
npm install nodemon -g 
````

````javascript
nodemon xxx.js
````




