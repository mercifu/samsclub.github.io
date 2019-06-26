---
layout:     post
title:      nodejs + express + mssql 数据操作
subtitle:
description:  上一篇写了使用 mssql 模块访问数据库数据，这一篇就将 express 和 mssql 模块结合起来，写一个小的 web 应用。虽然这个 web 应用距离 微服务 应用还差很多...
date:       2019-06-20
author:     Sam
header-img: 
catalog: true
tags:
    - nodejs
    - mssql
    - express
    - Microsoft SQL Server
---
### nodejs + express + mssql 数据操作
上一篇写了使用 mssql 模块访问数据库数据，这一篇就将 express 和 mssql 模块结合起来，写一个小的 web 应用。虽然这个 web 应用距离 微服务 应用还差很多，但还是先写起来吧。

#### 1. 新建项目及安装所需模块
新建项目和安装项目所需模块与上篇一样，因为懒得想项目叫什么，这里还叫 test ...
```cmd
mkdir test

cd test

npm init

npm install mssql --save

npm install express --save

nmp install q --save
```

#### 2. 编写配置文件
为了 web 应用的健壮性，这里还是要写个配置文件

```javascript
// db-config.js

const config = {
  user: 'sa',
  password: '123456',
  server: 'localhost', // You can use 'localhost\\instance' to connect to named instance
  database: 'test',
  pool: {
    max: 10,
    min: 0,
    idleTimeoutMillis: 30000
  },
  // options: {
  //   encrypt: true // Use this if you're on Windows Azure
  // },
  debug: false
}

module.exports = config

```

#### 3. 编写数据库操作基类
同样为了健壮性，写一个简单的数据库连接与执行脚本的基类。这里的基类是参考一介布衣的文章，下面参考有链接。

这里使用到了一个叫 q 的模块，同样还有方法里面使用到了很多 return 关键字。这些涉及到 nodejs 异步函数的问题，我也是被 nodejs 的异步调用问题搞到焦头烂额，以后另开一篇讲一下 nodejs 的异步编程。
```javascript
// db-base.js

var config = require('./db-config')
var sql = require('mssql')
var Q = require('q')

function _Base() {
}

_Base.prototype._connect = function(callback) {
  var defer = Q.defer()
  var connection = new sql.ConnectionPool({
    user: config.user,
    password: config.password,
    server: config.server,
    database: config.database
  })
  defer.resolve(connection)
  return defer.promise.nodeify(callback)
}

_Base.prototype._query = function(sqlstr, callBack) {
  if (!sqlstr) {
    var err = new Error('sql is empty!')
    var defer = Q.defer()
    return defer.reject(err)
  }

  if (config.debug) {
    console.log('[SQL:]', sqlstr, '[:SQL]')
  }

  return this._connect().then(function(connection) {
    return connection.connect().then(function() {
      var request = new sql.Request(connection)
      return request.query(sqlstr).then(function(recordset) {
        connection.close()
        return callBack(err, recordset)
      }).catch(function(err) {
        connection.close()
        return err
      })
    })
  })
}

module.exports = _Base
```

#### 4. 编写对象类
编写一个简单的 User 对象类继承 _Base 类，在 nodejs 中有多种编写类以及继承的方式，这里使用了 prototype 的方式去继承。
```javascript
// user.js

const _Base = require('./db-base')

function User() {
  _Base.call(this)
}

User.prototype = new _Base()

User.prototype.SelectAll = function(callBack){
  User.prototype._query('select * from [User]', function(err, result){
    if(err){
      console.log(err)
    }
    return callBack(result.recordset)
  })
}

module.exports = User
```

#### 5. 编写 web 服务实例
这里编写一个 web 服务实例，在实例中引用 express 模块，并在客户端发出请求后响应。

这里写了两个路由去响应客户端的请求，一个是 / ，另一个是  /user/all。
```javascript
// test.js

const express = require('express')
const app = express()

var User = require('./user')

app.get('/',function (req,res){
    res.send('hello world')
})

app.get('/user/all',function (req,res){
    var user = new User()
    user.SelectAll(function(result){
        res.send(result)
    })
})

var server = app.listen(3000, function(){
    var host = server.address().address
    var port = server.address().port

    console.log("server running on http://%s:%s", host, port)
})
```
启动服务
```
node test.js
```

#### 6. 验证
在浏览器中输入 http://localhost:3000/ 界面显示 hello world。

输入 http://localhost:3000/user/all 界面显示获取的数据。

成功！！！


#### 参考
https://segmentfault.com/a/1190000010324061

http://yijiebuyi.com/blog/69ebad54c394487547921ff6b3239622.html

