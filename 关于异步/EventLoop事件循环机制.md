###[rocess.nextTick](http://nodejs.org/docs/latest/api/process.html#process_process_nexttick_callback)和[setImmediate](http://nodejs.org/docs/latest/api/timers.html#timers_setimmediate_callback_arg)。EventLoop事件循环机制

参考https://www.cnblogs.com/hanzhecheng/p/9046144.html

主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）。

- **宏任务**：包括整体代码script，setTimeout，setInterval
- **微任务**：Promise.then(非new Promise)，process.nextTick(node中)

#####同步和异步

所有任务可以分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。**同步任务**指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；**异步任务**指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

执行机制如下：

事件的执行顺序，`是先执行宏任务，然后执行微任务`，这个是基础。

（1）所有同步任务都在主线程上执行，形成一个[执行栈](http://www.ruanyifeng.com/blog/2013/11/stack.html)（execution context stack）。

（2）主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。

（3）一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。

（4）主线程不断重复上面的第三步。

任务可以有同步任务和异步任务，同步的进入主线程，异步的进入Event Table并注册函数，异步事件完成后，会将回调函数放入Event Queue中(`宏任务和微任务是不同的Event Queue`)，同步任务执行完成后，会从Event Queue中读取事件放入主线程执行，回调函数中可能还会包含不同的任务，因此会循环执行上述操作。

#####任务队列和异步函数

任务队列"是一个事件的队列（也可以理解成消息的队列），IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件。

"任务队列"中的事件，除了IO设备的事件以外，还包括一些用户产生的事件（比如鼠标点击、页面滚动等等）。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程读取。

所谓"回调函数"（callback），就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

**"任务队列"是一个先进先出的数据结构，**排在前面的事件，优先被主线程读取。主线程的读取过程基本上是自动的，只要执行栈一清空，"任务队列"上第一位的事件就自动进入主线程。但是，由于存在后文提到的"定时器"功能，主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程。

#####rocess.nextTick和setImmediate

process.nextTick方法可以在当前"执行栈"的尾部----下一次Event Loop（主线程读取"任务队列"）之前----触发回调函数。也就是说，它指定的任务总是发生在所有异步任务之前。setImmediate方法则是在当前"任务队列"的尾部添加事件，也就是说，它指定的任务总是在下一次Event Loop时执行，这与setTimeout(fn, 0)很像。

多个process.nextTick语句总是在当前"执行栈"一次执行完，多个setImmediate可能则需要多次loop才能执行完。

**列子1**

```javascript
setTimeout(function() {
    console.log('setTimeout');
},1000)

new Promise(function(resolve) {
    console.log('promise');
}).then(function() {
    console.log('then');
})

console.log('console');
```

- 首先setTimeout，放入Event Table中，1秒后将回调函数放入宏任务的Event Queue中
- new Promise 同步代码，立即执行`console.log('promise')`,然后看到微任务then，因此将其放入微任务的Event Queue中
- 接下来执行同步代码`console.log('console')`
- 主线程的宏任务，已经执行完毕，接下来要执行微任务，因此会执行Promise.then，到此，第一轮事件循环执行完毕
- 第二轮事件循环开始，先执行宏任务，即setTimeout的回调函数，然后查找是否有微任务，没有，时间循环结束

> 到此做个总结，事件循环，先执行宏任务，其中同步任务立即执行，异步任务，加载到对应的的Event Queue中(setTimeout等加入宏任务的Event Queue，Promise.then加入微任务的Event Queue)，所有同步宏任务执行完毕后，如果发现微任务的Event Queue中有未执行的任务，会先执行其中的任务，这样算是完成了一次事件循环。接下来查看宏任务的Event Queue中是否有未执行的任务，有的话，就开始第二轮事件循环，依此类推。

**例子2**

```javascript
async function async1() {
	console.log('async1 start')
    await async2()
    console.log('async1 end')
}

async function async2() {
	console.log('async2')
}

console.log('script start')

setTimeout(function () {
	console.log('setTimeout')
}, 0)

async1();

new Promise (function (resolve) {
	console.log('promise1')
	resolve();
}).then(function () {
	console.log('promise2')
})
console.log('script end')


```

```javascript
script start
async1 start
async2
promise1
script end
promise2
async1 end
setTimeout
```

1、同步的宏任务console.log('script start') 输出

2、异步的宏任务setTimeout放入eventQueue

3、同步的宏任务async1的console.log('async1 start')

4、async2的console.log('async2')

5、await async2 await中断async函数，先执行async外的同步代码。

6、Promise宏任务console.log('promise1')，Promise.then加入微任务的Event Queue)，所有同步宏任务执行完毕后，如果发现微任务的Event Queue中有未执行的任务，会先执行其中的任务

7、宏任务console.log('script end')

8、微任务Promise .then() console.log('promise2')且return Promise.resolve(undefined)。

9、await后面的宏任务console.log('async1 end')

10、按顺序执行微任务

11、setTimeout

宏任务：

script start、async1 start、 async2、promise1、script end、      async1 end

​	微任务：promise2、undefined

宏任务： 

setTimeout