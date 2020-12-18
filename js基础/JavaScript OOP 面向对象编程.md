###面向对象编程

**面向对象编程**是一种编程范例或者说编程风格。

四个核心概念：封装、抽象、继承、多态

封装：将内部的某一属性封装起来，外界不能直接操纵这个属性，只能通过开放的方法进行赋值或者取值。

抽象：抽象可以被看作封装的自然扩展。隐藏属性和方法的实现细节，仅对外公开接口，隔离代码影响

继承：让一个对象可以访问到另一个对象中的属性和方法，可以消除代码冗余

多态：同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果。（可以让你摆脱冗长的if/else 或者 switch/case）多态提供了一种完全像使用其父类一样，使用别的类的方式，这样就不会将杂糅在一起的各个类搞混。但是每个子类仍然保留着自己的私有方法。



#####创建对象的方法(包含new的原理)

```js
// 创建对象语法
const circle = {
  radius: 1,
  location: {
    x: 1,
    y: 1
  },
  draw: function() {
    console.log('draw');
  }
}

```

使用创建对象语法来创建多个对象是有问题的，不方便，应该使用**工厂函数**或**构造函数**。

```js
// 工厂函数
function createCircle(radius) {
  return {
    radius,
    draw: function() {
      console.log('draw')
    }
  }
}

const circle = createCircle(1)
```

```js
// 构造函数
function Circle(radius) {
  
  // 为了保证构造函数必须与new命令一起使用，一个解决办法是，构造函数内部使用严格模式，即第一行加上use strict。这样的话，一旦忘了使用new命令，直接调用构造函数就会报错。
 	'use strict';
  
  // 另一个解决办法，构造函数内部判断是否使用new命令，如果发现没有使用，则直接返回一个实例对象。
  
  if (!(this instanceof Circle)) {
    return new Circle(raduis);
  }
  
  //console.log(this)
  this.radius = radius
  this.draw = function () {
    console.log('draw')
  }
}

// new会创建一个空对象{}，然后会设置this指向这个对象（即指向circel，this默认指向全局对象，如果注释掉new，this会指向window），最后它从函数返回这个对象。(见下方解释)
const another = new Circle(1)

//输入another.constructor会打印出上面的Circle函数。输入circle.constructor，会打印出一个Object函数。

// another.constructor
Circle(raduis) {
  this.radius = radius
  this.draw = function () {
    console.log('draw')
  }
}

// circle.constructor
Object() {[native code]} 

// Object是JavaScript里的内建函数。当我们使用创建对象语法来创建对象的时候，js内部使用了这个构造函数。

let x = {}
//js引擎会转化为类似 let x = new Object()
new String() // '' "" ``

```

> 使用new命令时，它后面的函数依次执行下面的步骤。
>
> 1. 创建一个空对象，作为将要返回的对象实例。
> 2. 将这个空对象的原型，指向构造函数的`prototype`属性。
> 3. 将这个空对象赋值给函数内部的`this`关键字。
> 4. 开始执行构造函数内部的代码。

默认情况下，this是指向全局对象。如果是在浏览器运行，this指向window。如果是在node.js环境运行，全局对象就是global。

**每个javascript对象中都有一个叫构造函数的属性constructor**。

```js
// 构造函数作为模板，可以生成实例对象。但是，有时拿不到构造函数，只能拿到一个现有的对象。我们希望以这个现有的对象作为模板，生成新的实例对象，这时就可以使用Object.create()方法。

var person1 = {
  name: '张三',
  age: 38,
  greeting: function() {
    console.log('Hi! I\'m ' + this.name + '.');
  }
};

var person2 = Object.create(person1);

person2.name // 张三
person2.greeting() // Hi! I'm 张三.
```

```js
// 用class语法构造（es6）
class Test {
	constructor(x,y) {
		this.x = x;
		this.y = y;
	}
	add() {
		return this.x+this.y
	}
}
let math = new Test(2,3)

math.add() // add() {return this.x+this.y}
math.add // 5
math.x // 2

```



还有一种用**prototype**

```javascript
// prototype 
function Person() {
 }
 
 Person.prototype.name = "lisi";
 Person.prototype.age = 21;
 Person.prototype.family = ["lida","lier","wangwu"];
 Person.prototype.say = function(){
     alert(this.name);
 };
 console.log(Person.prototype);   //Object{name: 'lisi', age: 21, family: Array[3]}
 
 var person1 = new Person();        //创建一个实例person1
 console.log(person1.name);        //lisi
 
 var person2 = new Person();        //创建实例person2
 person2.name = "wangwu";
 person2.family = ["lida","lier","lisi"];
 console.log(person2);            //Person {name: "wangwu", family: Array[3]}
 // console.log(person2.prototype.name);         //报错
 console.log(person2.age);              //21
```



如果构造函数内部有`return`语句，而且`return`后面跟着一个**对象**，`new`命令会返回`return`语句指定的对象；否则，就会不管`return`语句，返回`this`对象。

```js
var Vehicle = function () {
  this.price = 1000;
  return 1000;
};

(new Vehicle()) === 1000
// false
```

上面代码中，构造函数`Vehicle`的`return`语句返回一个数值。这时，`new`命令就会忽略这个`return`语句，返回“构造”后的`this`对象。

但是，如果`return`语句返回的是一个跟`this`无关的新对象，`new`命令会返回这个新对象，而不是`this`对象。这一点需要特别引起注意。

```js
var Vehicle = function (){
  this.price = 1000;
  return { price: 2000 };
};

(new Vehicle()).price
// 2000
```

另一方面，如果对普通函数（内部没有`this`关键字的函数）使用`new`命令，则会返回一个空对象。

```js
function getMessage() {
  return 'this is a message';
}

var msg = new getMessage();

msg // {}
typeof msg // "object"

```





##### 数据类型

1. 值类型（元值）： 字符串、数字(没有两种数字类型)、布尔值、undefined和null（特殊的object）、Symbol
2. 引用类型：object、array（特殊的object）、function （特殊的object）

值类型复制值，对象的引用类型复制它们的引用

```javascript
let x = 10
let y = x
x = 20
// 值类型 x=20  y=10 它们相互独立

let x = {value: 10}
let y = x
x.value = 20
// 引用类型 x = {value: 20}  y = {value: 20}
// 对象不存在变量里，对象在内存中的地址存在变量里，所以x和y指向同个地址，值相同

let number = 10
function increase(number) {
  number++
}
increase(number)
console.log(number) // number = 10
// 这个值被复制到函数的本地参数的number里面，作用域不同，因为是复制的是值，所以与外面无关

let obj = {value: 10}
function increase(obj) {
  obj.value++
}
increase(obj)
console.log(obj) // obj = {value: 11}
// 当我们引用函数并传入obj，这个对象是以引用的方式传入的，所以变量通过引用地址指向外面定义的obj
```

#####获取对象的属性

遍历对象可以用for-in

```js
function Circle(raduis) {
  this.radius = radius
  this.draw = function () {
    console.log('draw')
  }
}

const circel = new Circel(1)

for (let key in circel) {
  // 如果只想得到属性，不想得到方法，可以用typeof检测类型
  if (typeof clrcel[key] !== 'function') {
    // ...
  }
  onsole.log(key) // radius,draw
  console.log(circel[key]) // 10, f () {...}
}

// Object.keys 方法可以获取对象所有的key，以数组格式返回
const keys = Object.keys(circel) 
onsole.log(keys) // ["radius", "draw"]

// 检测对象有没有包含某一属性
if ('radius' in circel) {
  console.log('yes')
}
```

##### 表达式还是语句？

```javascript
{ foo: 123 }
```

js引擎在遇到无法确定是对象还是代码块的代码，一律解释为代码块。

```javascript
{ console.log(123) } // 123，这个语句只有解释为代码块才能执行
```

**如果要解释为对象，最好在大括号前加上圆括号。**因为圆括号的里面，只能是表达式，所以确保大括号只能解释为对象。

```javascript
({ foo: 123 }) // 正确
({ console.log(123) }) // 报错
```

这种差异在`eval`语句（作用是对字符串求值）中反映得最明显。

```javascript
eval('{foo: 123}') // 123
eval('({foo: 123})') // {foo: 123}
```

上面代码中，如果没有圆括号，`eval`将其理解为一个代码块；加上圆括号以后，就理解成一个对象。

##### with语句

`with`语句的格式如下：

```javascript
with (对象) {
  语句;
}
```

它的作用是操作同一个对象的多个属性时，提供一些书写的方便。

```javascript
// 例一
var obj = {
  p1: 1,
  p2: 2,
};
with (obj) {
  p1 = 4;
  p2 = 5;
}
// 等同于
obj.p1 = 4;
obj.p2 = 5;

// 例二
with (document.links[0]){
  console.log(href);
  console.log(title);
  console.log(style);
}
// 等同于
console.log(document.links[0].href);
console.log(document.links[0].title);
console.log(document.links[0].style);
```

注意，如果`with`区块内部有变量的赋值操作，必须是当前对象已经存在的属性，否则会创造一个当前作用域的全局变量。

```javascript
var obj = {};
with (obj) {
  p1 = 4;
  p2 = 5;
}

obj.p1 // undefined
p1 // 4
```

上面代码中，对象`obj`并没有`p1`属性，对`p1`赋值等于创造了一个全局变量`p1`。正确的写法应该是，先定义对象`obj`的属性`p1`，然后在`with`区块内操作它。

这是因为`with`区块没有改变作用域，它的内部依然是当前作用域。这造成了`with`语句的一个很大的弊病，就是绑定对象不明确。

```javascript
with (obj) {
  console.log(x);
}
```

单纯从上面的代码块，根本无法判断`x`到底是全局变量，还是对象`obj`的一个属性。这非常不利于代码的除错和模块化，编译器也无法对这段代码进行优化，只能留到运行时判断，这就拖慢了运行速度。因此，建议不要使用`with`语句，可以考虑用一个临时变量代替`with`。

```javascript
with(obj1.obj2.obj3) {
  console.log(p1 + p2);
}

// 可以写成
var temp = obj1.obj2.obj3;
console.log(temp.p1 + temp.p2);
```

#####抽象

```js
function Circel(radius) {
  this.radius = radius;
  let defaultLocation = {x:0, y:0};
  let computeOptimumLocation = function(factor) {
    // ...
  }
  this.draw = function() {
    let x,y; // 只在这个作用域有效
    computeOptimumLocation(0.1); // 存在内存中，因为它也在闭包中，闭包是常驻的。
    console.log('draw');
  }
  // 如果要访问defaultLocation有两种做法
  // 第一种方法
  this.getDefaultLocation = function () {
    return defaultLocation;
  }
  // 第二种方法
  Object.defineProperty(this, 'defaultLocation', {
    get: function () {
      return defaultLocation;
    },
    set: function (value) {
      // 赋值之前还可以做些判断
      if (!value.x || !value.y) {
          throw new Error('Invalid location.')
      }
      defaultLocation = value;
    }
  })
}

const circle = new Circel(10);
circle.getDefaultLocation(); // 方法一
circle.defaultLocation; // 方法二

circle.draw();

```

作用域是临时的，每次调用draw方法，变量x,y都会重新创建和声明，函数执行完毕就销毁。而闭包是常驻的，变量defaultLocation和computeOptimumLocation仍然驻留在内存中，因为它们是draw函数的闭包。

#####继承

当两个类都要用到同一个方法，为了避免重复写这个方法，可以定义一个基类（父类），把方法放到基类里面，然后继承基类的方法。

> es6之前js没有类的概念，都是用直接使用原型对象模仿面向对象中的类和类继承。但是JS 中并没有一个真正的 `class` 原始类型， `class` 仅仅只是对原型对象运用语法糖。所以，只有理解如何使用原型对象实现类和类继承，才能真正地用好 `class`。

有3种继承：

- 经典继承
- 原型继承
- 类继承

######原型继承

js中的每一个成员（除了一个特殊的），它们都从原型继承所有成员。

