---
title: ES6数组小技巧
date: '2021-05-26'
categories:
 - Javascript
tags:
 - ES6
---

### 数组解构赋值应用

```js
// 交换变量
[a, b] = [b, a]
[o.a, o.b] = [o.b, o.a]
// 生成剩余数组
const [a, ...rest] = [...'asdf'] // a：'a'，rest: ["s", "d", "f"]
```

### 数组浅拷贝

```js
const arr = [1, 2, 3]
const arrClone = [...arr]
// 对象也可以这样浅拷贝
const obj = { a: 1 }
const objClone = { ...obj }
```

**浅拷贝**：拷贝的值改变时原来的值跟着一起改变。

**深拷贝**：拷贝的值可以无限层拷贝，拷贝值与原始值不发生任何影响

浅拷贝方法有很多如`arr.slice(0, arr.length)/Arror.from(arr)`等，但是用了`...`操作符之后就不会再想用其他的了~

### 数组合并

```js
const arr1 = [1, 2, 3]
const arr2 = [4, 5, 6]
const arr3 = [7, 8, 9]
const arr = [...arr1, ...arr2, ...arr3]
```

`arr1.concat(arr2, arr3)`同样可以实现合并，但是用了`...`操作符之后就不会再想用其他的了~

### 数组去重

```js
const arr = [1, 1, 2, 2, 3, 4, 5, 5]
const newArr = [...new Set(arr)]
```

`new Set(arr)`接受一个数组参数并生成一个set结构的数据类型。set数据类型的元素不会重复且是`Array Iterator`，所以可以利用这个特性来去重。

### 数组取交集

```js
const a = [0, 1, 2, 3, 4, 5]
const b = [3, 4, 5, 6, 7, 8]
const duplicatedValues = [...new Set(a)].filter(item => b.includes(item))	//includes判断数组里有无此value，返回布尔值
duplicatedValues // [3, 4, 5]
```

### 数组取差集

```js
const a = [0, 1, 2, 3, 4, 5]
const b = [3, 4, 5, 6, 7, 8]
const diffValues = [...new Set([...a, ...b])].filter(item => !b.includes(item) || !a.includes(item)) // [0, 1, 2, 6, 7, 8]
```

### 数组转对象

```js
const arr = [1, 2, 3, 4]
const newObj = {...arr} // {0: 1, 1: 2, 2: 3, 3: 4}
```

### 对象转数组

```js
const obj = {a: '群主', b: '男群友', c: '女裙友', d: '未知性别'}

const flatArr = Object.values(obj);		//["群主", "男群友", "女裙友", "未知性别"]
```

### 数组摊平

```js
const obj = {a: '群主', b: '男群友', c: '女裙友', d: '未知性别'}
const getName = function (item) { return item.includes('群')}
// 方法1
const flatArr = Object.values(obj).flat().filter(item => getName(item))
// 经大佬指点，更加简化（发现自己的抽象能力真的差~）
const flatArr = Object.values(obj).flat().filter(getName)

```

二维数组用`array.flat()`，三维及以上用`array.flatMap()`。

### 找到第一个符合条件的元素/下标

```
const arr = [1, 2, 3, 4, 5]
const findItem = arr.find(item => item === 3) // 返回子项
const findIndex = arr.findIndex(item => item === 3) // 返回子项的下标

// 我以后再也不想看见下面这样的代码了😂
let findIndex
arr.find((item, index) => {
    if (item === 3) {
        findIndex = index
    }
})
```

### array.includes() 和 array.indexOf()

`array.includes()` 返回是否包含元素的布尔值，`array.indexOf()` 返回数组子项的索引。



### array.find() 、 array.findIndex() 和 array.some()

`array.find()`返回值是第一个符合条件的数组子项，`array.findIndex()` 返回第一个符合条件的数组子项的下标，`array.some()` 返回有无复合条件的子项，如有返回`true`，若无返回`false`。

（注意这三个都是短路操作，即找到符合条件的之后就不在继续遍历。）



在需要数组的子项的时候使用`array.find()` ；需要子项的索引值的时候使用 `array.findIndex()` ；而若只需要知道有无符合条件的子项，则用 `array.some()`。

（建议在只需要布尔值的时候和数组子项是字符串或数字的时候使用 `array.some()`）



### array.find() 和 array.filter()

只需要知道 `array.filter()` 返回的是所有符合条件的子项组成的数组，会遍历所有数组；

而 `array.find()` 只返回第一个符合条件的子项，是短路操作。



### Array.from()方法

在常用代码中，我们经常会遍历所有li对象，通过以下方法来获取，并且试着去遍历每个item

```js
document.getElementsByTagName('li').forEach(() => {
 // Do something here..
})
// 会报出以下错误，xxx.forEach is not a function
```

为什么会这样？这是因为通过getElementsByTagName获取的是 `HTMLCollection` 对象，并不是数组，而是 `类数组` 对象，所以不能使用 `forEach` 来遍历它。

**什么是类数组对象：**类似数组的对象，比如

```js
let arrayLike = {
  '0': 'a', 
  '1': 'b', 
  '2': 'c', 
  length: 3
};
```

这里就需要用到 `Array.from()` 方法了，`Array.from()` 能将类数组对象转换为数组，进而能够在它上面执行所有数组操作。

```js
const collection = Array.from(document.getElementsByTagName('li'));
console.log(Array.isArray(collection))  //t 
//这里的 collection 是一个数组
```

