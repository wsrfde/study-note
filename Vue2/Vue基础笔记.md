---
title: Vue基础学习笔记
date: '2021-06-13'
categories:
 - Vue
tags:
 - Vue2
---

### 方法和函数的区别
1. 面向对象的时候称为方法，面向流程的时候叫函数。  
2. 方法是作为一个对象实例的属性的函数  


<br>

### 生命周期

生命周期：事务从诞生到消亡的整个过程  
以下是生命周期函数，可以在某个周期做某件事

	[
		'beforeCreate',
		'created',		//组件创建完成
		'beforeMount',
		'mounted',		//DOM树创建完成
		'beforeUpdate',	//数据更新前   可以更改数据
		'updated',		//DOM更新完成   最好不要在这里改数据，会发生死循环
		'beforeDestroy',	//事件的移除  清空定时器
		'destroyed',	//手动移除组件或路由切换，会发生视图改变
		'activated',	//钩子函数，需要作用keep-alive下
		'deactivated',	//钩子函数，需要作用keep-alive下
		'errorCaptured',
		'serverPrefetch'
	  ]


<br>

### 计算属性  

> computed：运行时作为一个属性来添加，所以调用里面函数时不用加括号()  


注：函数起名时不要用动词，因为computed时作为属性来调用，如getBgColor => BgColor  

**区别：**  
	conputed在多次调用时只执行一次，有缓存，只做数据处理  
	methods是调用几次执行几次，没有缓存，浪费性能  



<br>

### 函数重载

在js中相同名的函数，下面的函数会覆盖上面的  

在typescript中，函数重载指
* 两(多)个函数的函数名相同
* 传入的参数不同
* 与返回值无关


<br>

### v-on的参数问题

1. 如果该方法不需要额外参数，那么调用方法时()可以不添加。  

		@click = 'getBtn'
2. 如果方法中有参数，调用时没有添加()和参数,则默认传入event
3. 如果需要同时传入某个参数，同时需要event时，可以通过$event传入事件。  

		@click = 'getBtn("abc",$event)'  

#### v-on的多个绑定

```html
<div @click="clickFun" @mousemove="moveFun">我是内容</div>
<!-- 对象语法 -->
<div v-on="{click:clickFun,mousemove:moveFun}">我是内容</div>
```

<br>

### v-on的修饰符

* .stop - 调用 停止事件冒泡 event.stopPropagation() 。

   `<a v-on:click.stop="doThis"></a>`

* .prevent - 调用 阻止默认事件 event.preventDefault()。
		`<form v-on:submit.prevent="onSubmit"></form>`
	
* 修饰符可以串联
		
	
		`<a v-on:click.stop.prevent="doThat"></a>`
	
* .{keyCode | keyAlias} - 只当事件是从特定键触发时才触发回调。

   ```html
   <input type="text" @keyup="keyup($event.which)"> 	//监听键码
   <input type="text" @keyup.enter="keyup()">  //监听键盘字符
   <input type="text" @keyup.13='keyup()'>		// 监听简码
   ```

* .self - 只当在 event.target 是当前元素自身时触发处理函数 

   即事件不是从内部元素触发的

   `<div v-on:click.self="doThat"></div>`

* .native - 监听组件根元素的原生事件。

* .once - 只触发一次回调。


<br>

### v-if、v-else-if、v-else  

> **原理：**  

v-if后面的条件为布尔值，为flase时对应的元素以及其子元素不会渲染。

	<h2 v-if="ishow">当ishow为true时，显示我</h2>
	<h2 v-else>当ishow为false时，显示我</h2>

这三个指令和JavaScript的指令类似  

> **bug注意：**  
* v-if的渲染时为虚拟dom，为了节省内存会把判断有无原标签，有的话原标签复用。  
* 但后果是如果输入input标签进行if else切换会出现原来输入的value还存在的问题。  
* 解决方法是在标签内部添加key，key值不一样不会复用。   _ps: key='new'_


<br>

### v-show

v-show和v-if非常相似，用于决定一个Dom会不会渲染
true渲染，false隐藏

> **区别：**
* v-show是在标签内部添加行内样式display='none'
* v-for是直接创建或移除Dom
* 实际开发中我们用v-for多一点


<br>

### v-for

> v-for遍历数组

```html
<!-- 项目，下标 -->
<p v-for="(item,index) in message">{{index}}-{{item}}</p>
```

<br>

> v-for遍历对象

```html
<!-- 值，键，下标 -->
<li v-for="(value,key,index) in message">{{index+'.'+key+'-'+value}}</li>
```

<br>

### v-for 组件的key属性

* 官方推荐我们使用v-for时，给对应的元素或组件添加上一个:key属性。  
* Vue的虚拟DOM的Diff算法在更新操作节点时，给某个元素插入赋值，
	后面的元素依次更新，没有则重新创建dom再赋值，来完成节点操作，
	这样的效率很慢```ps：ABCD => ABECD中，c=>e，d=>c，''=>d```
* 添加key可以给每个元素做唯一标识，Diff算法就可以正确识别节点准确更新
* **所以，key的作用主要是为了高效的更新虚拟DOM。**

#### 使用key的示例

```html
<!--如果是静态资源，key可以使用index，但如果不是，则使用id保证当前item的key则是唯一的-->  
<p v-for="item in message" :key='item.id'>{{item}}</p>
```

#### 使用template时不能用key怎么办?

> template是空标签，key必须要绑定到真实dom上

```html
<template v-for='(item,index) in arr'>
  <p :key='`name${index}`'>{{item}}</p>
  <p :key='`index${index}`'>{{index}}</p>
</template>
```

#### vue的复用策略

当使用if进行替换操作时，vue默认会复用原来的dom，解决办法便是加上key

```html
<template v-if='isShow'>
  <div>我是A</div>
  <input type="text" key='1'>
</template>
<template v-if='!isShow'>
  <div>我是B</div>
  <input type="text" key='2'>
</template>
```



<br>

### vue数据响应式

> 响应式是指数据可以动态响应到页面上，不需要手动刷新

**如果数组里是对象，直接更改即可，如果是纯数组，则需要用到以下的方法**

> 以下方法操作数组vue都是响应式的  
* push()
* pop()
* shift()
* unshift()
* splice()
* sort()
* reverse()  


> 直接使用下标的方式来改变数组不是响应式  

	this.arr[0] = 'aaa'  

解决办法  

	(被修改的obj/数组，key/下标，要修改的值)
	Vue.set(this.arr,0,'bbb')

> 删除数据delete也不是响应式

	delete this.obj.age;

解决办法

	(被删除的obj，key)
	Vue.delete(this.obj,'age')

### vue的初始化选项

获取vue最开始定义的属性值，并不会被后来的改变所影响

```js
let vm = new Vue({
  el:'#app',
  customOption:'foo'
})
vm.customOption = 'obj';

console.log(vm.$options.customOption) //foo
```



<br>

### 过滤器(filters)

* 过滤器filters和计算属性computed都是vue的一种方法  
* 内部也是一种函数方法，返回过滤后的值
* 和computed一样，调用时无需加括号()
* 但过滤器函数内可以传值，传递的是当前被过滤的数据


```js
<h2>{{数据 | 过滤器}}</h2>
```

```html
<div id='app'>
	<h2>{{message | changeMessage}}</h2>
</div>

<script>
	const app = new Vue({
		el:'#app',
		data:{
			message:1.23
		},
		filters:{
			changeMessage(value){
				//toFixed是显示小数点后几位
				return value.toFixed();
			}
		}
	})
</script>
```

<br>

### 监听data属性(watch)

> 用法是监听data的属性，当属性发生改变调用函数。和methods同级

注意：不要使用箭头函数来定义watch函数。因为箭头函数指向父级作用域上下文，不会指向vue实例
```js
	watch:{
	  name(){
      //执行函数...
			setTimeout(this.refresh,20)
	  }
	}
```

#### watch的高级用法

```js
watch:{
  name:{
    handler(newVal){    //watch的处理函数
      console.log(newVal);
    },
      immediate:true,   //即时：第一次定义data时也被监控
      deep:true,    //深层：监控data中对象中的参数
      lazy:true     //懒惰：compouted的实现
  }
}
```

除了上述配置还可以只监听对象中某一属性

```js
data(){
  return {
    obj:{name:'vicer',age:18}
  }
},
watch:{
  // 注意，此写法只能监听对象中某一属性，不能监听数组中某一属性
  "obj.name":function(newVal,oldVal){
    console.log(newVal,oldVal)
  }
}
```



<br>

### img加载监听

原生的js监听图片: 

	img.onload = function() {}  
Vue中监听:  

	@load='方法'

### $nextTick下一帧

$nextTick()内部传递箭头函数

意为箭头函数可以在其他函数运行完的下一帧运行。 

 

