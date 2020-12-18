

####for-in

**for-in可以遍历数组和对象，获得的是数组的下标。可以break。**

用于遍历对象时，以原始插入顺序访问对象的可枚举属性，**包括从原型继承而来的可枚举属性**（容易造成性能浪费或对结果造成影响，所以一般会加上hasOwnProperty）。

```javascript
let obj = {
        a: 1
        , b: 2
        , c: 3
        , d: 4
    };
for( let k in obj ){
    console.log(k, obj[k]);
}
// 输出结果
// a 1
// b 2
// c 3
// d 4

// break
for( let k in obj ){
    if (k === 'b') {
        break
    }
    console.log(k, obj[k]);
}
// a 1

let newObj = Object.create(obj);

newObj.e = 5;
newObj.f = 6;

for( let k in newObj ){
    console.log(k, newObj[k]);
}
// 输出结果  
// e 5
// f 6
// a 1
// b 2
// c 3
// d 4

for(let pro in newObj){
  if (newObj.hasOwnProperty(pro)) {
    console.log(pro+':' + newObj[pro])
  }
}
// 输出结果  
// e 5
// f 6
```

用于遍历数组时，可以将数组看作对象，数组下标看作属性名。但用for…in遍历数组时不一定会按照数组的索引顺序。

```javascript
let arr = [123,'abc']
for(let pro in arr){
    console.log(pro+':' + arr[pro])
}
//0:123
//1:abc
```



####for-of

for-of语句在可迭代对象（Array，Map，Set，String，TypedArray，arguments 对象等等）上创建一个迭代循环，为每个不同属性的值执行语句，与forEach相比可以正确响应break，continue，return语句。

**一般for-of不能遍历普通对象（特殊方法可以看下文例子）**，因为： **ES6** 中引入了 **Iterator（遍历器）**，只有提供了 **Iterator** 接口的数据类型才可以使用 **for-of** 来循环遍历，而 **Array**、**Set**、**Map**、某些类数组如 **arguments** 等数据类型都默认提供了 **Iterator** 接口，所以它们可以使用 **for-of** 来进行遍历。

**for-of循环获得的是数组的值。**

```javascript
let arr = [123,'abc']
for(let item of arr){
    console.log(item)
}
//123
//abc
```

for…of遍历Map时，可以获得整个键值对对象：

```javascript
let iterable = new Map([["a", 1], ["b", 2], ["c", 3]]);

for (let entry of iterable) {
  console.log(entry);
}
// ["a", 1]
// ["b", 2]
// ["c", 3]

// 也可以只获得键值或value
for (let [key,value] of iterable) {
  console.log(key+':'+value);
}
//a:1
//b:2
//c:3
```



**使用for-of遍历对象的方法**：为对象已经其它的一些数据类型提供 **Iterator** 接口

```javascript
newObj[Symbol.iterator] = function* (){
    let keys = Object.keys( this )
        ;

    for(let i = 0, l = keys.length; i < l; i++){
        yield {
            key: keys[i]
            , value: this[keys[i]]
        };
    }
}

```

拓展在 **class** 中使用 **Symbol.iterator**

```javascript
class User{
    constructor(name, gender, lv){
        this.name = name;
        this.gender = gender;
        this.lv = lv;
    }

    *[Symbol.iterator](){
        let keys = Object.keys( this )
            ;

        for(let i = 0, l = keys.length; i < l; i++){
            yield {
                key: keys[i]
                , value: this[keys[i]]
            };
        }
    }
}

let zhou = new User('zhou', 'male', 1);

for(let {key, value} of zhou){
    console.log(key, value);
}
// 输出结果
// name zhou
// gender male
// lv 1
```

#####forEach

forEach方法对数组/Map/Set中的每个元素执行一次提供的函数。该函数接受三个参数：

- 正在处理的当前元素，对于Map元素，代表其值；
- 正在处理的当前元素的索引，对于Map元素，代表其键，对于Set而言，索引与值一样。
- forEach()方法正在操作的数组对象。

```javascript
let arr = [1,2,3,4]
arr.forEach(function(value,index,currentArr){
    currentArr[index]=value + 1
})
console.log(arr)//[ 2, 3, 4, 5 ]

// 通过keys遍历对象
Object.keys(obj).forEach(function(key){
	console.log(key,obj[key]);
});

// return等同for的continue
let  a = ['a', 'b', 'c']
a.forEach(b=>{
    if (b === 'b') return false
    console.log(b)
})
// a c

```

**特殊方法跳出循环方法**

```javascript
// catah
try {
  [1, 2, 3, 4, 5].forEach(function(item) {
    if (item=== 2) throw item;
    console.log(item);
  });
} catch (e) {}

// 修改索引（看下文解释执行细节）
var array=[1, 2, 3, 4, 5]
array.forEach(item=>{
  if (item == 2) {
    array = array.splice(0);
  }
  console.log(item);
})
```

forEach的执行细节：

1. 遍历的范围在第一次执行callback的时候就已经确定，所以在执行过程中去push内容，并不会影响遍历的次数，这和for循环有很大区别，下面的两个案例一个会造成死循环一个不会.

   ```javascript
   var arr=[1,2,3,4,5]
   //会造成死循环的代码
   for(var i=0;i<arr.length;i++){
       arr.push('a')
   }
   //不会造成死循环
   arr.forEach(item=>arr.push('a'))
   ```

2. 如果已经存在的值被改变，则传递给 `callback` 的值是 `forEach` 遍历到他们那一刻的值。

   ```javascript
   var arr=[1,2,3,4,5];
   arr.forEach((item,index)=>{
       console.log(`time ${index}`)
       arr[index+1]=`${index}a`;
       console.log(item)
   })
   
   ```

3. .已删除的项不会被遍历到。如果已访问的元素在迭代时被删除了（例如使用 `shift()`），之后的元素将被跳过。

   ```javascript
   var arr=[1,2,3,4,5];
   arr.forEach((item,index)=>{
       console.log(item)
       if(item===2){
           arr.length=index;
       }
   })
   ```

   在满足条件的时候将后面的值截掉，下次循环的时候照不到对应的值，循环就结束了，但是这样操作会破坏原始的数据，因此我们可以使用一个小技巧，即将数组从0开始截断，然后重新赋值给数组也就是array=array.splice(0)

####总结区别

#####跳出循环

**return** 只存在于函数体中，**for，for-in，for-of都不支持return, 用break**。
**forEach，every这些函数中可以使用return**
**forEach中的return false相当于for循环中的continue，一般无法跳出循环（特殊的跳出循环的方法可以用catch）**



若你需要**提前终止循环**，你可以使用：
简单循环
[for...of](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of) 循环
[Array.prototype.every()](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
[Array.prototype.some()](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some)
[Array.prototype.find()](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find)
[Array.prototype.findIndex()](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)

every在碰到return false的时候，中止循环。some在碰到return ture的时候，中止循环。

```javascript

var a = [1, 2, 3, 4, 5]
a.every(item=>{
    console.log(item); //输出：1,2
    if (item === 2) {
        return false
    } else {
        return true
    }
})
var a = [1, 2, 3, 4, 5]
a.some(item=> {
    console.log(item); //输出：1,2
    if (item === 2) {
        return true
    } else {
        return false
    }
})
```

#####遍历的内容和获得的内容

for-in可以遍历数组和对象，获得的是数组的下标。

一般for-of不能遍历普通对象，for-of循环获得的是数组的值。

forEach不能直接遍历对象，可以通过Object.keys或者Object.values遍历

##### 

