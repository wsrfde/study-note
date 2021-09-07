---
title: ES6笔记
date: '2021-05-26'
categories:
 - Javascript
tags:
 - ES6
---

## ES6笔记整理

### let与const

> let是变量，有块级作用域。const是常量。

const注意事项：
1. 使用const不能为空，必须进行赋值
2. 一旦const修饰的标识符被赋值后，就不能在更改
3. 常量的定义是对象（也就是内存地址）不能修改，但对象内部的属性可以更改
4. 如果内部的属性值也不希望改变，可以使用

	``` JS
	const person={
		name:'vicer',
		age:18
	}
	
	const vicer = Obj.freeze(person) 
	vicer.age=20    //修改失败
	```

建议:   
在ES6开发中,优先使用const, 只有需要改变某一个标识(zhi)符的时候才使用let. 

### var与let/count的区别

> var拥有自己的function scope

在for循环中使用var定义的i。使用setTimeout时，打印i的值为循环最大值。  
```JS
for(var i=0;i<10;i++){
	setTimeout(function(){
		console.log(i)	//打印10遍10
	},1000)
}
```
因为setTimeout在异步队列中属于宏任务，同步任务执行完成后进入主线执行任务
使用let来声明for循环时，i属于块级作用域，所以i会正常打印

### 字符串模板 ，反引号

Tab上面的点 ` ，来定义字符串，和普通的引号区别是可以换行

字符串模板中间还可以添加变量
```JS
const name = 'vicer'

const helloName = `my name is ${name}`
```

### JSON的序列化与反序列化

json标准写法  '{"a":0,"b":"name"}'

* JSON序列化  

  JSON.stringify()	
  在需求中一般需要发送数据，我们需要把数据序列化，也就是转成标准JSON字符串

  ```JS
  const json = {a:0,b:'vicer'}
  console.log(JSON.stringify(json))	//{"a":0,"b":"vicer"}
  ```

* JSON反序列化  

	JSON.parse()
	把字符串转化成json
	```JS
	const json = '{"a":0,"b":"vicer"}';
	console.log(JSON.parse(json))	//{a: 0, b: "vicer"}
	```


### 对象字面量增强写法

```JS
const name = 'vicer'
//我想把name赋值到下面的对象中去

//es5写法
const obj = {
	name:name
}
//es6写法
const obj = {
	name
}
```

### 对象的解构

当使用一个对象时，我们可以通过 如obj.name获取其中的属性  

但我们还有一种方法，就是直接传入一个对象进去，  
这样我们就可以直接使用对象  
```JS
let json = {a:'1',b:'2',c:'3'}
let json2 = {
...json,
d:'4'
}
```
### 数组的解构

当我们需要使用数组时，除了把数组for循环遍历出来，还有一种写法  
```JS
//数组展开1
let arr = [1,2,3,4]
console.log(...arr)	//1 2 3 4

//展开方式2
let [a1,a2,...a3] = [1,2,3,4]
console.log(a2)		//2

//数组合并
let arr1 = [1,2,3]
let arr2 = [4,5,6]
let allArr = [...arr1,...arr2]
```

```JS
//数组收集
function test(a,b,...c){
console.log(a,b,c)
}
test(1,2,3,4,5,6,7)	//1,2,Array
```
### 解构赋值

1. 两边结构必须一样
2. 右边必须是一个东西
3. 赋值和解构必须一样

```js
//对象另一侧也是对象
let str = {a:1,b:2,c:3}
let {a,b} = str
//console.log({a,b})

//数组另一侧也是数组
let arr = [1,2,3]
let [x,y] = arr;
//console.log([x,y])
```

### 函数默认值

在我们定义函数传递参数的时候，可以给一个形参传递一个默认值

```js
function test(name,age=18){
  //之前的方式
  //age = age || 18
  console.log(name,age)
}
test('vicer')
//vicer 18
```

这里默认值可以时任意类型，也可以是一个表达式，

```js
function test(name,age = anotherFun()){
  console.log(name,age)
}

function anotherFun(){
  return 18;
}
test('vicer');
//vicer 18
```



### 箭头函数

函数的三种方式：  
1. 声明定义  
		`const a = function(){}`
	
2. 对象字面量定义函数  
		
	```js
	const b ={
			b2(){}
	}
	```
	
3. 箭头函数  
		`const c = () => {}  `

	**当需要函数作为参数传入到另一个函数时，使用箭头函数最多**  
		

### 箭头函数传参与返回值

1. 当传的参数只有一个时可省略括号()
		
	```js
	const d = n => {
			return n+n
	}
	```
	
2. 当箭头函数只有一行时必须省略花括号
		`const e = n => n+n`
	注：省略花括号{}时无需再写return，系统会自动返回值

### 箭头函数的this使用

正常情况下this是谁调用我我指向谁。

问题：箭头函数中this是如何查找的？  

答案1：在外层作用域中，逐层查找，直到有this定义。也就是指向作用域中最近this
答案2：自己理解：当前代码块中，外层代码块被谁调用就指向谁，没有找到，就继续向外查找

```javascript
const obj = {
  a(){
    console.log(this);//obj  谁调用我指向谁
    setTimeout(function(){
	  console.log(this)// window
	  //因为这里call函数，传入一个window。所以指向this
    },100);
    setTimeout(() =>{
	  console.log(this)//obj
	  //在外层作用域中，逐层查找，直到有this定义。也就是指向作用域中最近this
    },100);
  }
}
obj.a();
```

<br>

### promise异步封装

> 一般情况下有异步操作时，使用Promise对这个异步操作进行封装。  

* Promise内部执行回调函数，在执行传入的回调函数时还会传入两个参数，resolve，reject。而这两参数本身又是函数

* 在请求成功时调用resolve，参数默认传入到then中处理。  
* 在请求失败时调用reject，参数默认传入到catch中处理。

简而言之：Promise是一个优雅的函数，在内部判断成功或失败，成功调用then处理，失败调用catch处理。

#### 两种写法

1. resolve调用then，reject调用catch  

   ```js
   new Promise((resolve, reject)=>{
     setTimeout(()=>{
       // resolve('hello word');
       reject('error message')
     },1000)
   }).then((data)=>{
     console.log(data);
   }).catch(err =>{
     console.log(err);
   })
   ```

   

2. 都写在then中

   ```js
   new Promise((resolve, reject)=>{
     setTimeout(()=>{
       // resolve('hello word')
       reject('error message')
     },1000)
   }).then((data)=>{
     console.log(data);
   },(err)=>{
     console.log(err);
   })
   ```

   

#### promise的简写

当promise只有第一步进行异步操作，接下来操作不是异步时，可以进行代码简写

* 一般简写方式

  ```js
  new Promise((resolve, reject)=>{
    setTimeout(()=>{
      resolve('hello word')
    },1000)
  }).then((data)=>{
    console.log(data);
    //成功回调简写
    // return Promise.resolve(data+111)  
    //失败回调简写
    return  Promise.reject('error message')
  }).then((data)=>{
    console.log(data)
  }).catch(err =>{
    console.log(err);
  })
  ```

  

* 超级简写方式

  ```js
  new Promise((resolve, reject)=>{
    setTimeout(()=>{
      resolve('hello word')
    },1000)
  }).then((data)=>{
    console.log(data);
    //成功回调简写
    // return data+111
    //失败回调简写
    throw  'error message'
  }).then((data)=>{
    console.log(data)
  }).catch(err =>{
    console.log(err);
  })
  ```

  

#### promise.all的使用

当我们需要两个异步操作同时完成才能返回结果时，一般需要用到all方法。

all中放的是数组，如promise.all([])

```javascript
  Promise.all([
    new Promise((resolve, reject) => {
      setTimeout(()=>{
        resolve({name:'vicer',age:18})
      },1000)
    }),
    new Promise((resolve, reject) => {
      setTimeout(()=>{
        resolve({name:'lily',age:19})
      },2000)
    })
  ]).then(results =>{
    console.log(results);	//返回的是数组
  },reject =>{
	  console.log(reject)
  })
```



### promise结合jquery.ajax

#### promise与$.ajax的结合使用

我们首先看下promise与$.ajax的结合使用
```js
let p = new Promise((resolve, reject) => {
	$.ajax({
		url: './text/1.txt',
		dataType: 'json',
		success(res) {
			resolve(res)
		},
		error(err) {
			reject(err)
		}
	})
})

p.then(res =>{
	console.log(res)
},err =>{
	console.log(err)
})
```

我们都知道promise非常出名，那么ajax作者知道嘛?

答案是肯定的，在$.ajax中也可以使用then来接收成功回调
```js
$.ajax({
	url: './text/1.txt',
	dataType: 'json'
}).then(res => {
	console.log(res)
}, err => {
	console.log(err)
})
```

有人可能会好奇，ajax既然存在then，为什么还要使用promise。因为请求参数时不可能只请求一个参数，可能会请求多个，所以需要用到promise.all

#### promise.all与$.ajax的结合使用

这里直接上源码对比原来的promise.all和现在的使用区别
```js
let p = Promise.all([
	//原来的promise.all的使用方法
	new Promise((resolve, reject) => {
		$.ajax({
			url: './text/1.txt',
			dataType: 'json',
			success(res) {
				resolve(res)
			},
			error(err) {
				reject(err)
			}
		})
	}),
	//在得知$.ajax的秘密之后
	$.ajax({url: './text/2.txt',dataType: 'json'}),
	$.ajax({url: './text/3.txt',dataType: 'json'})
])

p.then(([data1,data2,data3]) => {
	console.log(data2)	
}, err => {
	console.log(err)
})
```

### promise.race的使用

race竞速的意思，数据请求中谁先完成我使用谁。  
只返回一个数据  
用途：适合不同接口但数据相同  

```js
Promise.race([
	$.ajax({
		url: './text/1.txt',
		dataType: 'json'
	}),
	$.ajax({
		url: './text/2.txt',
		dataType: 'json'
	})
]).then(res =>{
	console.log(res)
})
```

### promise.finally介绍

所有参数都请求完成后执行的操作（不论成功失败）

### async/await异步操作

* 普通函数  

	一直执行，直到结束

* async函数  

	函数能够“暂停”

> 简单介绍异步函数:	

<font color="Coral">同步的写法，异步的操作</font>

`同步的写法，异步的操作`

```
async  function show(){
	let a = 1;
	let b = 2;
	let c = await $.ajax({url: './text/1.txt',dataType: 'json'})//await后面也可以跟一个promise，等promise完成后再执行
	alert(a+b+c[0])
}
show()
```

> 异步函数有个缺点，加入操作出错了，我想捕获异步函数，该怎么做

<font color="coral">使用try和catch</font>

```js
async  function show(){
	let a = 1;
	let b = 2;
	try{
		let c = await $.ajax({url: './text/1.txt',dataType: 'json'})
		alert(a+b+c[0])
	}catch(e){
		alert(读取失败)
	}
}
show()
```



### ES5面向对象

首先说一下es5的面向对象写法，方法多，而且类与构造函数分不清
```js
function Person(name,age){
	this.name=name;
	this.age=age;
}
Person.prototype.showName = function(){
	alert(this.name)
}
Person.prototype.showAge = function(){
	alert(this.age)
}

const a = new Person('vicer',18);
a.showAge();
a.showName();
```

### ES6面向对象

#### 面向对象

class		定义类  
constructor	定义构造函数  
方法直接定义在类里面，方便管理。

```js
class Person {
	constructor(name, age) {
		this.name = name;
		this.age = age;
	}
	showName = function () {
		alert(this.name)
	}
	showAge = function () {
		alert(this.age)
	}
}

const a = new Person('vicer', 18);
a.showAge();
a.showName();
```

#### get和set的使用

在class中我们可以使用get和set获取和设置属性

```JS
class Chef{
	constructor (foods){
		this.foods = foods;
		this.dish = []
	}
	get menu(){
		return this.dish
	}
	set menu(dish){
		this.dish.push(dish)
	}
}

let a = new Chef()

console.log(a.menu = '鸡蛋')	//设置属性
console.log(a.menu = '青椒')
console.log(a.menu)			//获取属性
```

#### staitc静态方法

static静态方法就是不需要实例化就可以使用的方法

```JS
class Chef{
	constructor (foods){
		this.foods = foods;
	}
	static cook(food){
		console.log(food)
	}
}
console.log(Chef.cook('西红柿'))
```



#### 类的继承

extends 继承  
super	父类/超类

继承的好处：  
1. 省事，直接把父类的属性和方法拿过来用
2. 在写extends时父类的方法自动就添加了过来，通过super拿到父类的属性
3. 扩展，给父类添加某种方法的时候子类都会有

```js
class Person {
	constructor(name, age) {
		this.name = name;
		this.age = age;
	}
	showName = function () {
		alert(this.name)
	}
	showAge = function () {
		alert(this.age)
	}
}

class Worker extends Person {	//继承person
	constructor(name, age, work) {
		super(name, age)		//获取父类
		this.work = work
	}
	showWork(){
		alert(this.work)
	}
}

const w = new Worker('jeck',20,'工人')
w.showName();
w.showAge();
w.showWork();
```

### Set的使用

#### Set

> 它类似于数组，但是成员的值都是唯一的，没有重复的值。

* `add`		      添加数组
* `size`	    	获取length
* `has`		      检查是否拥有某个值
* `forEach`	遍历数组
* `delete`	   删除其中某个值
* `clear`	 	清空数组

```js
let arr = new Set('123'); // === new Set([1,2,3])
console.log(arr)	//{"1", "2", "3"}
console.log(arr.add('4'))	//{"1", "2", "3", "4"}
console.log(arr.has('3'))	//true
arr.forEach(res =>{
	console.log(res)		//1 2 3 4
})
console.log([...arr])		// [1,2,3,4]
arr.delete('2')
console.log(arr)	//{"1", "3", "4"}
arr.clear()
console.log(arr)	// {}
```

##### Set结构转数组

* 数组解构  `[...new Set(arr)]`
* Array.from   `Array.from(new Set(arr))`

##### Set方法取并集、交集、差集

```js
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter((x) => b.has(x)));
// set {2, 3}

// （a 相对于 b 的）差集
let difference = new Set([...a].filter((x) => !b.has(x)));
// Set {1}
```



#### WeakSet

##### 与Set的区别

> WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。

**区别一：**`WeakSet 的成员只能是对象`，而不能是其他类型的值。

```js
const ws = new WeakSet();
ws.add(1);
// TypeError: Invalid value used in weak set
ws.add({name:'vicer'});		// [{name:'vicer'}]
ws.add([1,2,3]);					// [{name:'vicer'},[1,2,3]]
```

**区别二：**`WeakSet 中的对象都是弱引用`。即如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。

##### WeakSet的使用

> 由于其特殊性，无法遍历及使用其他方法，能使用的api如下

- `add`              向 WeakSet 实例添加一个新成员
- `delete`       清除 WeakSet 实例的指定成员
- `has`              返回一个布尔值，表示某个值是否在 WeakSet 实例之中

```js
const ws = new WeakSet();
const obj = {};
const foo = {};

ws.add(obj);
ws.has(foo); // false
ws.delete(obj);
```



### Map的使用

#### Map

> Map使用键值对储存数据，并且键的范围不限于字符串，各种类型的值（包括对象）都可以当作键。普通JS对象只能用字符串作为键。如果你需要“值—值”的对应，那么使用Map比Object更合适

* `set(key,value) | ([item,item])`      新增Map结构或者更新已存在键值，并返回当前的`Map`对象，

  ```js
  const key = { foo: 1 };
  let map = new Map();
  map.set(key,'bar')
  map.get(key);	// "bar"
  
  // 除了传入键值对，还可以直接传入可迭代，且每个成员都是一个双元素的数组的数据结构，也就是说，Set和Map都可以用来生成新的 Map
  let m0 = new Map([
    ["a", "张三"],
    ["b", "李四"],
  ]);
  m0.get("a"); // "张三"
  
  const set = new Set([
    ["foo", 1],
    ["bar", 2],
  ]);
  const m1 = new Map(set);
  m1.get("foo"); // 1
  
  const m2 = new Map([["baz", 3]]);
  const m3 = new Map(m2);
  m3.get("baz"); // 3
  
  // 因为返回返回当前的`Map`对象，因此可以采用链式写法。
  let map = new Map();
  map.set("foo", true).set("bar", false);
  ```

* `size`	    	获取length

* `get`              获取对应key的值

* `has`		      检查是否拥有某个值

* `delete`	   删除其中某个值

* `clear`	 	清空数组

* `forEach`	遍历数组

* `keys`            返回键名的遍历器

* `values`       返回键值的遍历器

```js
let food = new Map();
let fruit = '水果',cook = function(){},dessert = {name:'甜点'};
food.set(fruit,'柠檬')	
food.set(cook,'eating')
food.set(dessert,'甜甜圈')	
console.log(food)			//{"水果" => "柠檬", ƒ => "eating", {...} => "甜甜圈"}
console.log(food.get(fruit))	// 柠檬
console.log(food.size)//3
food.delete(fruit)
console.log(food.has(fruit))//false
food.clear()
console.log(food)	//{}
```

##### Map结构转为数组结构

```javascript
const map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']
```

##### Map结构转为对象

```js
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k, v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

const myMap = new Map().set("yes", true).set("no", false);
strMapToObj(myMap);
// { yes: true, no: false }
```

##### 对象转为 Map

对象转为 Map 可以通过`Object.entries()`。

```javascript
let obj = { a: 1, b: 2 };
let map = new Map(Object.entries(obj));
```

此外，也可以自己实现一个转换函数。

```javascript
function objToStrMap(obj) {
  let strMap = new Map();
  for (let k of Object.keys(obj)) {
    strMap.set(k, obj[k]);
  }
  return strMap;
}

objToStrMap({ yes: true, no: false });
// Map {"yes" => true, "no" => false}
```

#### WeakMap

##### 与Map的区别

> `WeakMap`结构与`Map`结构类似，也是用于生成键值对的集合。

**区别一：**`WeakMap`只接受对象（或数组）作为键名，不接受其他类型的值作为键名

```js
// WeakMap 可以使用 set 方法添加成员
const wm1 = new WeakMap();
const key1 = { foo: 1 };
wm1.set(key1, 2);
wm1.get(key1); // 2

// WeakMap 也可以接受一个数组，
// 作为构造函数的参数
const key2 = [1, 2, 3];
const wm2 = new WeakMap([
  [key2, "foo"]
]);
wm2.get(key2); // "bar"
```

**区别二：**`它的键名所引用的对象都是弱引用`，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的`其他引用都被清除`，垃圾回收机制就会释放该对象所占用的内存。

```js
const wm = new WeakMap();

const element = document.getElementById("example");

wm.set(element, "some information");
wm.get(element); // "some information"
// WeakMap 里面对element的引用就是弱引用，不会被计入垃圾回收机制。
// 一旦消除对该节点的引用，它占用的内存就会被垃圾回收机制释放。Weakmap 保存的这个键值对，也会自动消失。
element = null;
```

**注意，WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。**

##### WeakMap的使用

> 由于其特殊性，无法遍历及使用其他方法，能使用的api如下

* `get`        获取对应key的值
* `set`        新增Map结构或者更新已存在键值，并返回当前的`WeakMap`对象
* `has`         检查是否拥有某个值
* `delete`      删除其中某个值

### ES7

#### 幂运算

```JS
let a = 3;
alert(a**2);	//3的2次方
```

#### Array.includes

查找数组是否存在

```JS
let arr = [1,2,3,4];

//indexOf是用来查找下标
//console.log(arr.indexOf(2) != -1 )	

//includes是用来查找是否存在
console.log(arr.includes(2))
```

### 迭代器

迭就是轮流的意思，代就是交换。  
迭代就是轮流交换的意思。  

每次执行的时候有两个对象  
```JS
//value 返回的值。  dong 还有没有可迭代的东西。有返回false
{value:xx,done:true/false}
```
里面还有一个next函数，用来迭代下次的value

**手写迭代器**
```js
function chef(foods){
	let i = 0;
	return {
		nest(){
			let done = (i>=foods.length);
			let value = !done ? foods[i++] : undefined;
			return{
				value:value,
				done:done
			}
		}
	}
}

let iterators = chef(['西红柿','鸡蛋'])
console.log(iterators.nest())	//{value: "西红柿", done: false}
console.log(iterators.nest())	//{value: "鸡蛋", done: false}
console.log(iterators.nest())	//{value: undefined, done: true}
```

手写迭代器有点麻烦，但要理解其中原理。
我们可以使用generators生成迭代器

1. 在函数后面加*号
2. 使用yield添加要迭代的item
3. 传入参数并调用next

```js
function* chef(foods){
	for(let item of foods){
		yield item;
	}
}

let iterators = chef(['西红柿','鸡蛋'])
console.log(iterators.next())	
console.log(iterators.next())	
console.log(iterators.next())	
```

