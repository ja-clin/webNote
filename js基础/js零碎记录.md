构造函数不能使用箭头函数。

异步回调的使用当中，没有办法使用正常的try catch机制

localStorage.setItem('comments', JSON.stringify(comments))





##### 数组合并  push、concat、 拓展运算符对比 

```javascript
let arr1 = [1, 2];
let arr2 = [3, 4];
// concat
arr1 = arr1.concat(arr2);
// 扩展运算符
arr1 = [...arr1, ...arr2];
// 或者
arr1.push(...arr2);
```

```javascript
arr1 = [...arr1, ...arr2];
  ↓ 相当于
function _toConsumableArray(arr) {
 if (Array.isArray(arr)) { 
   for (var i = 0, arr2 = Array(arr.length); i < arr.length; i++) { arr2[i] = arr[i]; } return arr2; 
   } else { return Array.from(arr); }
}
arr1 = [].concat(_toConsumableArray(arr1), arr2);

arr1.push(...arr2);
  ↓ 相当于
arr1.push.apply(arr1, arr2);
```

`concat`性能最优，兼容性好。`...`是`es6`新出的语法，`简化`了写法，代码看上去更简洁直观，但实际只是做了`封装`，底层还是用的原来的方法。在数据过大时第三种方法会因为`apply`的特点由于数据量过大导致堆栈溢出。

##### 不改变原始对象，创建新对象

拓展运算符，Object.assign(第一个参数为空())，concat，JSON.parse(JSON.stringify(xxx))

