---
layout:     post
title:      nodejs 使用 mssql 模块连接数据库
subtitle: 
description: 使用 nodejs 去连接 mssql
date:       2019-06-14
author:     Sam
header-img: 
catalog: true
tags:
    - nodejs
    - mssql
    - Microsoft SQL Server
---
### nodejs 使用 mssql 模块连接数据库
目前项目组负责的项目是一个九十年代的产品，收集用户的数据然后在浏览器上面浏览报告。但是，因为我们产品的平台是针对 Windows，所以浏览器只支持 IE 5 6 7 8 9。现在有一个支持现在浏览器的需求，需要一个 demo。

为了可以快速开发，给产品经理提供一个报告的 demo，我选用了现成的开源的后台管理系统 vue-element-admin。产品收集的数据存放在 mssql，所以就需要使用 nodejs 去连接 mssql。

这里新建一个 nodejs 项目。

#### 1. 新建项目

```cmd
mkdir test      \\ 新建目录

cd test         \\ 转到项目目录

npm init        \\ 初始化 nodejs 项目
```

#### 2. 安装所需模块

这里需要安装 mssql 模块

```
npm install mssql --save        // 安装 mssql 模块
```

#### 3. 准备配置文件

这里需要连接到数据库的配置文件。

```javascript
const config = {
    user: 'sa',
    password: '123456',
    server: 'localhost', // You can use 'localhost\\instance' to connect to named instance
    database: 'test',
 
    // options: {
    //     encrypt: true // Use this if you're on Windows Azure
    // }
}

// 使用连接字符串连接
// const config = 'mssql://sa:123456@localhost/test'

// 指定端口的连接字符串
// const config = 'mssql://sa:123456@locahost:1433/test'

module.exports = config
```

#### 4. 编写

编写 test.js，连接到数据库，并检索所有用户。

```javascript
const sql = require('mssql')
const config = require('./config')

sql.connect(config, err => {
    // ... error checks
    if(err){
        console.log('connect fail')
    }
 
    new sql.Request().query('select * from [User];', (err, result) => {
        // ... error checks
        if(err){
            console.log(err)
        }
        console.log(result)
    })
})
 
sql.on('error', err => {
    // ... error handler
    if(err){
        console.log(err)
    }
})

```
运行
```cmd
node test.js
```

这里需要注意的是，mssql 的表名需要用 [ ] 括起来不然会出现 **Error: Incorrect syntax near the keyword 'User'** 的错误。这是我遇到的一个错误。


#### 参考
https://www.jianshu.com/p/0f2f7db5d9fc

http://yijiebuyi.com/blog/69ebad54c394487547921ff6b3239622.html

https://segmentfault.com/a/1190000010324061

https://www.npmjs.com/package/mssql
