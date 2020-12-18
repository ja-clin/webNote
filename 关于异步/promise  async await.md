###Promise

promise实例一经创建，立刻执行。

##### 异步回调存在的四个问题

1. 嵌套层次很深，难以维护
2. 无法正常使用return和throw （后面的回调函数是在前面操作完成后才回调的，不处于同一个栈）
3. 无法正常检索堆栈信息（因为每次回调都是在系统层面上的一个新的堆栈）
4. 多个回调直接难以建立联系 

###promise 与async/await

参考：https://blog.csdn.net/tcy83/article/details/80544048

#####async

async 函数（包含函数语句、函数表达式、Lambda表达式）会返回一个 Promise 对象，如果在函数中 `return` 一个直接量，async 会把这个直接量通过 `Promise.resolve()` 封装成 Promise 对象。

```javascript
async function testAsync() {
    return "hello async";
}

const result = testAsync();
console.log(result);

c:\var\test> node --harmony_async_await .
Promise { 'hello async' }
```

**await**

一般来说，都认为 await 是在等待一个 async 函数完成。不过按[语法说明](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/await)，await 等待的是一个表达式，这个表达式的计算结果是 Promise 对象或者其它值（换句话说，就是没有特殊限定）。

因为 async 函数返回一个 Promise 对象，所以 await 可以用于等待一个 async 函数的返回值——这也可以说是 await 在等 async 函数，但要清楚，它等的实际是一个返回值。注意到 await 不仅仅用于等 Promise 对象，它可以等任意表达式的结果，所以，await 后面实际是可以接普通函数调用或者直接量的。

如果它等到的不是一个 Promise 对象，那 await 表达式的运算结果就是它等到的东西。

如果它等到的是一个 Promise 对象，await 就忙起来了，它会阻塞后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果。

> 这就是 await 必须用在 async 函数中的原因。async 函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个 Promise 对象中异步执行。

**例子（rejected）**  没用await

```javascript
async function async1() {
	console.log( 'async1 start')
	async2()
	console.log( 'async1 end')
}

async function async2() {
	new Promise(function(resolve) {
        console.log('4'); 
        rejected();
    }).then(function() {
        console.log('5')
    })
}

async1()

console.log( 'script start')

输出>>>
async1 start
4
async1 end
script start
5
```

用awiat 后面跟的是promise

```javascript
async function async1() {
	console.log( 'async1 start')
	await async2()
	console.log( 'async1 end') 
}

async function async2() {
	new Promise(function(resolve) {
        console.log('4'); 
        resolve();
    }).then(function() {
        console.log('5')
    })
}

async1()

console.log( 'script start')

输出>>>
async1 start
4
script start
5
async1 end

```

后面跟的不是promise函数

```javascript

function testAwait(){
    setTimeout(function(){
        console.log("testAwait");
    }, 1000);
}
async function helloAsync(){
    await testAwait();
    console.log("helloAsync");
}
helloAsync();
console.log('late')

输出>>>
late
helloAsync
testAwait


function testAwait(){
    console.log('2323')
}
async function helloAsync(){
    await testAwait();
    console.log("helloAsync");
}
helloAsync();
console.log('late')

输出>>>
2323
late
helloAsync
```

**结论：**

await中断async函数，先执行async外的同步代码。

1、对于Promise对象，await会阻塞主函数的执行，等待 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果，然后继续执行主函数接下来的代码。

2、对于非Promise对象，await等待函数或者直接量的返回，而不是等待其执行结果。

**例子（rejected）**

awit遇到rejected的时候的处理方式：

```javascript

function testAwait(){
  	return Promise.reject("error");
 }
  async function helloAsync(){
  	await testAwait();
  	console.log("helloAsync");//没有打印
  }
  helloAsync().then(v=>{
      console.log(v);
  }).catch(e=>{
     console.log(e);//"error"
  });
```

从执行结果看，返回reject状态被外层的catch捕获到，然后终止了后面的执行。

但是在有些情况下，出错后是希望继续执行，而不是中断。对于这种情况可以采用tcy...catch在函数内部捕获异常

```javascript

function testAwait(){
  		return Promise.reject("error");
  	}
  async function helloAsync(){
  	try{
       await testAwait();
  	}catch(e){
  		console.log("this error:"+e)//this error:error
  	} 	
  	console.log("helloAsync");//helloAsync
  }
  helloAsync().then(v=>{
  }).catch(e=>{
     console.log(e);//没有打印
  });
```

异常被try...catch捕获后，继续执行下面的代码，没有导致中断。

#####async/await 的优势在于处理 then链

如果有一堆参数要处理 ，promise.then处理参数太麻烦，用await处理比较好（见例子2）

**例子1**

假设一个业务，分多个步骤完成，每个步骤都是异步的，而且依赖于上一个步骤的结果。我们仍然用 `setTimeout` 来模拟异步操作：

```javascript
/**
 * 传入参数 n，表示这个函数执行的时间（毫秒）
 * 执行的结果是 n + 200，这个值将用于下一步骤
 */
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}
```

现在用 Promise 方式来实现这三个步骤的处理

```javascript
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt();

// c:\var\test>node --harmony_async_await .
// step1 with 300
// step2 with 500
// step3 with 700
// result is 900
// doIt: 1507.251ms
```

用 async/await 来实现清晰很多

```javascript
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time2);
    const result = await step3(time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();
```

**例子2**

现在把业务要求改一下，仍然是三个步骤，但每一个步骤都需要之前每个步骤的结果。

```javascript
function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(m, n) {
    console.log(`step2 with ${m} and ${n}`);
    return takeLongTime(m + n);
}

function step3(k, m, n) {
    console.log(`step3 with ${k}, ${m} and ${n}`);
    return takeLongTime(k + m + n);
}
```

这回先用 async/await 来写：

```javascript
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time1, time2);
    const result = await step3(time1, time2, time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();

// c:\var\test>node --harmony_async_await .
// step1 with 300
// step2 with 800 = 300 + 500
// step3 with 1800 = 300 + 500 + 1000
// result is 2000
// doIt: 2907.387ms
```

用promise ：

```javascript
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => {
            return step2(time1, time2)
                .then(time3 => [time1, time2, time3]);
        })
        .then(times => {
            const [time1, time2, time3] = times;
            return step3(time1, time2, time3);
        })
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt();

```

