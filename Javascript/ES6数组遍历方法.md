---
title: JS数组中高阶遍历方法
date: '2021-04-05'
categories:
 - Javascript 
tags:
 - ES6
---

### filter() 

> filter方法检查数组，删除不匹配的元素，返回一个新数组  

* filter方法内部传入回调函数，回调函数要求必须传入数组的value

```javascript
	const arr = [1,2,3,4];
	let newArr = arr.filter(function(n){
		return n<3;
	})
	console.log(newArr)//1,2
```

--------------------------------------------------
<br>

### map()

> map方法遍历并处理数组中每个元素，并返回一个新数组

回调函数中传入的值为数组的value

```javascript
	const arr = [1,2,3,4];
	let newArr = arr.map(function(n){
		return n*2;
	})
	console.log(newArr)//2,4,6,8
```

--------------------------------------------------

### forEach()

> forEach方法为数组每个元素都执行一次回调函数，将元素传递给回调函数。

forEach和map的区别是没有返回值

```js
	const arr = [1,2,3,4];
	arr.forEach(function(item,index){
		let newArr = index+'-'+item*2;
		console.log(newArr)
	})

```


<br>

### reduce()

> reduce方法依次处理数组的每个成员，最终累计为一个值。  

**建议直接用设置初始值的方法  **

1. 因为reduce方法面对空数组时需要一个初始值，  
2. 如果数组内是对象，在没有初始值方法中，第一次遍历previousValue是一个成员需要计算



**使用方法**

* 该方法接收的第一个参数为函数，第二个为初始值
* 函数中又接收两个参数，(之前的返回值，当前值)

      reduce(function(之前的返回值,当前值){},初始值)

**没有初始值**

 * 函数内参数1为数组第一个成员，参数2为数组第二个成员
 * 参数1为第一次遍历的返回值，参数2为数组依次遍历的值，直到完成

```javascript
const arr = [1,2,3,4,5]
let newArr = arr.reduce(function (preValue,nowValue,currentIndex, array) {
  return preValue+nowValue;
})
console.log(newArr);
//1 2
//3 3
//6 4
//10 5
//最后结果15
```

**设置了初始值**

 * 第一次遍历初始值与数组第一个成员
 * 第二次遍历时参数1为返回值，参数2为数组依次遍历的值，直到完成

```javascript
const arr = [1,2,3,4,5]
let newArr = arr.reduce(function (preValue,nowValue) {
  return preValue+nowValue;
},10)
console.log(newArr);
//10 1
//11 2
//13 3
//16 4
//20 5
//最后结果25
```

--------------------------------------------------

<br>

### find()

> find方法遍历数组，返回符合条件的子元素

内如果判断为true，则返回item。
```js
	  const arr = [1,2,3,4]
    let arrItem =  arr.find(function(n){
		  return n==2;
	  });
	  console.log(arrItem);
	  //2
```

--------------------------------------------------

<br>

### every()

> every方法用于检测数组内所有元素是否**都**符合条件，返回布尔值

如果数组有一个元素不满足，则整个表达式返回 false，并剩余元素不会再检测  

every不会对空数组进行检测  
every不会改变原始数组

```js
	const  arr = [1,2,3,4]
	let isTrue = arr.every(function(n){
		return n>2;
	})
	console.log(isTrue);
	//false
```
