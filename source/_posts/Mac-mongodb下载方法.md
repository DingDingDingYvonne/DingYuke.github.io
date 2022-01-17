---
title: Mac mongodb下载方法
date: 2021-12-05 16:33:02
tags: ["数据库"]
---

在FDU-VIS的编程训练营中彭蕾学姐带来了精彩绝伦的“服务端数据库的简易搭建”课程，但在下载MongoDB的时候，mac的同学们全部阵亡，跟着官方文档有奇怪的问题。这里记录一下最终成功的方法

![彭蕾学姐的精彩图示](2.png)

##### 下载并安装MongoDB和Node.js

在官网<https://www.mongodb.com/try/download/community>下载mac版本的tgz格式文件

解压tgz文件。在终端进入到tgz文件所在的文件夹下，输入```tar zxvf 文件名.tgz -C ./``` (例如tar zxvf mongodb-macos-x86_64-5.0.4.tgz -C ./)

建一个文件夹把解压后的文件夹放进去，并且创建一个新的文件夹，例如叫做‘db’

终端进入解压后文件夹中的bin文件夹，再在终端输入```./mongod --dbpath ${刚刚新创建的db文件夹的path}```这里的path可以直接把db文件夹拖入终端，就会填上path啦。回车就开启了本地的服务器

下次开启时再重复```./mongod --dbpath ${刚刚新创建的db文件夹的path}```的操作就好。用ctrl c关闭服务器

安装node.js<https://nodejs.org/>
和MongoDB Compass<https://www.mongodb.com/products/compass>

打开Compass软件后输入connection string：
```mongodb://localhost:27017```

##### 连接到MongoDB并插入数据

再在任意的地方新建项目文件夹dbtry
将终端进入到这个空文件夹
npm init
npm install mongoose –-save
将下面的代码创建js文档放在dbtry的目录下，运行

```javascript
/*
  mongoose.js ：连接并操作数据库
 */

// ***** 建立数据库连接 ***** 
let mongoose = require('mongoose') // 引入mongoose
let url = "mongodb://localhost:27017/JuanWang"; // 数据库地址
mongoose.connect(url)

let db = mongoose.connection; // 建立连接
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function () {
    console.log("Successful connection to " + url)
});


// ***** 定义数据模型 ***** 
let juanWang = { //定义一个名为卷王的schema，有name和nickname两个属性，都是字符串
    name: String,
    nickname: String
}
let schema = mongoose.Schema(juanWang)

let JuanWang = mongoose.model('JuanWang', schema); //将schema编译为model构造函数


// ***** 插入数据 ***** 
let u1 = new JuanWang({ name: "张宇", nickname: "宇神" })
u1.save()

let u2 = new JuanWang({ name: "王呈舜", nickname: "王博" })
u2.save()

let u3 = new JuanWang({ name: "董佳奇", nickname: "董老师" })
u3.save()


module.exports = { JuanWang } // 将JuanWang模型导出，供其他模块使用
```

若成功连接至MongoDB，console中将显示提示信息MongoDB Compass中，打开对应数据库，可以看到刚插入的数据。

##### 搭建express服务框架
终端在dbtry文件夹下输入```npm install express --save```
再将以下代码创建js文档保存在dbtry目录下，运行
```javascript
/*
 express.js: 引入 express 模块，设置路由
*/

let { JuanWang } = require("./mongoose") // 导入刚才定义的JuanWang模型用来查询数据

let express = require('express')() // 引入express框架

express.listen(3000) //设置一个想要监听的端口


// ***** 设置路由 ***** 

// 当监听到客户端向'http://localhost:3000/juanwnag'发来GET请求时，执行以下操作:
express.get("/juanwang", function (request, response) {

    response.header('Access-Control-Allow-Origin', '*') // 设置允许跨域，重要！

    let n = request.query.queryName  // 客户端发来的查询字段

    JuanWang.findOne({ "name": n }, (err, data) => { // 去数据库调取相应数据并传回
        if (err) { return console.log(err) }
        else {
            response.send(data)
        }
    })
})
```

##### 在前端查看
在dbtry目录下创建index.html，用webserver打开，即可调取数据啦
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>编程训练营3</title>
</head>

<body>
    <h1>FDU-VIS年度卷王候选人：</h1>
    <button id='bt'>查询</button>
    <p id='name'></p>
    <p id='nickname'></p>

    <script src="https://d3js.org/d3.v7.min.js"></script>

    <!-- 导入Axios库 -->
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>

    <script>
        d3.select('#bt').on('click', () => {
            // 使用axios向目标服务器发起请求：
            axios
                .get('http://localhost:3000/juanwang', { // 由于都是在本地，所以写localhost。实际应用中，应按照需求改为正确的url
                    params: {
                        queryName: '王呈舜' // 设置查询字段
                    }
                })
                .then(response => { // 收到服务器返回的数据后执行操作：
                    d3.select('#name').html('姓名：' + response.data.name)
                    d3.select('#nickname').html('称号：' + response.data.nickname)
                })
                .catch(function (error) {
                    console.log(error);
                });
        })
    </script>
</body>

</html>
```