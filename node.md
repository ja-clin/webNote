####Node 的作用和运用

1. 脱离浏览器运行js
2. 后端编写api
3. webpack，gulp，npm等前端工程化的工具都是依赖node的
4. 中间层： 服务器中负责IO读写的中间层服务器（文件读写、数据库查询）

####Node中间层优点

1. 性能（js异步IO 适合处理高并发）
2. 处理数据（比如后端给了array的数据，前端需要的是json数据，数据多的话，直接在前端转换体验不好，数据可以在node处理后再给浏览器）
3. 安全性能（不会直接攻击后台，先面对node这一层中间层）

####Node优势

1. 方便前端学习，node语法和js相似，只是使用习惯不一样，前端注重交互、表现、用户体验。后端注重性能、安全、吞吐量。
2. 性能好，比较快。不如c语言，但比php等语言性能好（因为v8引擎做预编译等）。
3. 利于前端代码整合（比如同一套js前后端可以通用，比如同一套正则验证，前后端都可以用）。

####node模块

1. 全局模块：如process （也叫全局对象，类似js的document）不管作用域，不管层级多深都可以调用
2. 系统模块：系统内置的模块，不需要下载 （引入系统模块require ）
   1. path:用于处理文件路径和目录路径。不同电脑环境不一样相对路径可能有时有点问题，可以通过 __dirname可以拿到运行文件的目录，resolve解析拿到绝对路径。
   2. fs:用于读写操作
3. 自定义模块：自己写好暴露出去的模块

####require

1. 有路径就去路径找
2. 没有的话就去node_modules里面找
3. 什么都没有就去node的安装目录找

####require和import的区别

**import（es6语法）**

- ES6模块主要有俩个功能：export和import
- ES6 模块的`import`命令是异步加载，有一个独立的模块依赖的解析阶段。

- import是编译时调用，必须放在文件开头引入，目前部分浏览器不支持，需要用babel把es6转成es5再执行（babel-loader）
- default 是 ES6 Module 所独有的关键字，export default fs 输出默认的接口对象，import fs from 'fs' 可直接导入这个对象
- import静态编译，import的地址不能通过计算。Import url 直接报错了。

**require（CommandJS规范 ，在nodejs中使用）**

- 在nodejs环境中，我们采用的是CommandJS模块规范，使用require引入模块，使用module.exports导出接口
- CommonJS 模块的`require()`是同步加载模块

- require是运行时调用，所以可以随处引入
- require地址可以计算，例如 const url = "a" + "b"。require(url)不会报错。所以require都会用在动态加载的时候

**require/exports 和 import/export 形式不一样**

require/exports 的用法只有以下三种简单的写法：

```text
const fs = require('fs')
exports.fs = fs
module.exports = fs
```

而 import/export 的写法就多种多样：

```text
import fs from 'fs'
import {default as fs} from 'fs'
import * as fs from 'fs'
import {readFile} from 'fs'
import {readFile as read} from 'fs'
import fs, {readFile} from 'fs'

export default fs
export const fs
export function readFile
export {readFile, read}
export * from 'fs'
```

```javascript
// 有default的 调用设置后，页面的a还是没改
let a =1
export default a
export const setA = (val) => {
  a=val
}

import a, {setA} from 'xxx'
setA(3) // 输出a还是1

// 没有default的  a可以改变
export let a =1
export const setA = (val) => {
  a=val
}

import{a, setA} from 'xxx'
setA(3) // 输出a 为3

```



####module

可以直接export一个对象

```javascript
exports.a = 1
exports.b = 2

// module

// obj
module.exports = {
  a: 1,
  b: 2,
}
const mod = require('mod')
console.log(mod.a)

// function
modeule.exports = function() {
  console.log(123)
}
mod();

// class
module.exports = class {
  constructor(name) {
    this.name = name
  }
  show() {
    console.log(this.name)
  }
}
let p = new mod('test')
p.show()
```

#### http模块

服务器对象：http.creactServer() 快速搭建一个服务器。

**http头**（传递信息：url,版本等）<=32k,http body（传递数据）<2G

**get:**主要是获取数据，数据放在url里面（<=32k），容量小不能查询很大的数据。

**post:** 内容比较大，所以分段传