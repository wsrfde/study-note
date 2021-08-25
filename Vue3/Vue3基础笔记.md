---
title: Vue3基础笔记
date: '2021-06-07'
categories:
 - Vue
tags:
 - Vue3
---

> 本笔记是建立在已知Vue2知识点的一个新技术点补充，并非从0开始学习vue，如有vue2的技术点，请查看vue2文档

### template模板的两种写法

一、使用script标签，并且标记它的类型为 x-template

```html
<div id="app"></div>
<script type="x-template" id="why">
    <div>{{count}}</div>
    <button @click="addCount">+</button>
    <button @click="lessCount">-</button>
</script>

<script>
  Vue.createApp({
    template:`#why`,
  }).mount('#app')
</script>
```

二、使用任意标签（**通常使用template标签**，因为不会被浏览器渲染），设置id； 

template元素：是一种用于保存客户端内容的机制，该内容再加载页面时不会被呈现，但随后可以在运行时使 用JavaScript实例化；

```html
<div id="app"></div>
<template id="why">
    <div>{{count}}</div>
    <button @click="addCount">+</button>
    <button @click="lessCount">-</button>
</template>

<script>
  Vue.createApp({
    template:`#why`,
  }).mount('#app')
</script>
```

**注意：** 在Vue2中template模板中只允许有一个根元素，现在Vue3中不再有这个限制，可以有多个跟元素

### 源码调试

一、在GitHub上搜索` vue-next` 

二、下载下来在`package.json`中，在`dev`命令中增加`--sourcemap` 

三、引入Vue，然后进行`debugger`

### methods中为什么不能使用箭头函数

```js
Vue.createApp({
  template:`#why`,
  data(){
    return {
      count:0
    }
  },
  methods:{
    addCount(){
      this.count++
    },
    lessCount:()=>{
      console.log(this) //window    
    }
  }
}).mount('#app')
```

因为箭头函数会向上查找，methods:{}不是函数，只是对象，所以不是作用域。createApp({})也是一个对象，所以查找到最上层就是window

### 基本指令

#### 和vue2相同的指令

* `v-once`   用于指定元素或者组件只渲染一次，可以**用于性能优化**
* `v-text `   等价于{{}}语法
* `v-html`   将html解析在元素中
* `v-pre`      不解析{{}}语法
* `v-bind`    动态绑定属性（笔记中做了补充：动态绑定属性名称、绑定对象的所有属性到元素上）
* `v-on`         给元素绑定事件



#### `v-on`的修饰符

* ` .stop`                   调用 event.stopPropagation()。  停止事件冒泡

* `.prevent`           调用 event.preventDefault()。     阻止默认事件

* `.capture`           添加事件侦听器时使用 capture 模式。   捕获的意思，和冒泡相反，具体看[这篇文章](https://blog.csdn.net/catascdd/article/details/108273931)

* ` .self`                   只当事件是从侦听器绑定的元素本身触发时才触发回调。 即事件不是从内部元素触发的<<

* ` .{keyAlias}`    仅当事件是从特定键触发时才触发回调。

  ```html
  <input @keyup.enter="keyup()">   <!-- 只有enter弹起时调用事件 -->
  ```

* ` .once`                   只触发一次回调。 

* `.left`                  只当点击鼠标左键时触发。 

* ` .right`                 只当点击鼠标右键时触发。 

* ` .middle`              只当点击鼠标中键时触发。 

* ` .passive`            { passive: true } 模式添加侦听器

  **介绍：passive主要用在移动端的scroll事件，来提高浏览器响应速度，提升用户体验。**

  因为passive=true等于提前告诉了浏览器，touchstart和touchmove不会阻止默认事件，手刚开始触摸，浏览器就可以立刻给与响应；否则，手触摸屏幕了，但要等待touchstart和touchmove的结果，多了这一步，响应时间就长了，用户体验也就差了。

#### `v-bind`的修饰符

- `.prop` ： 被用于强制绑定 DOM 属性 (property)，设置的自定义属性不会在渲染后的 HTML 标签里显示。

  * 通过自定义属性存储变量，避免暴露数据

  - 防止污染 HTML 结构

  ```html
  <input id="input" :data.prop="inputData"></input>
  // 渲染后HTML标签结构
  <input id="input"></input>
  ```

- `.attr` ： 被用于强制绑定 DOM 属性 (attribute)，和prop刚好相反

  ```html
  <a :title.prop="firstTabTooltip" :aria-selected.attr="isFirstTabSelected">First tab</a>
  
  <!-- 两个简写方法 -->
  <a .title="firstTabTooltip" ^aria-selected="isFirstTabSelected">First tab</a>
  ```



### 侦听器watch

除了vue2中的watch用法，还增加了一个`$watch` 的API，具体用法如下

用法示例  `this.$watch('侦听的数据',()=>{回调函数},watch的配置选项)`

**`$watch`还会有一个返回函数，当调用此函数时停止监听**

```js
created(){
  const unwatch = this.$watch("info", function(newInfo, oldInfo) {
    console.log(newInfo, oldInfo);
  }, {
    deep: true,
    immediate: true
  })
  // unwatch()		// 调用停止监听
}
```

注意：当变更(不是替换)对象或数组并使用deep选项时，旧值将与新值相同，因为它们的引用指向同一个对象/数组。vue不会保留变更之前值的副本。

```js
data(){
  return {
    info:{name:'vicer',age:18}
  }
}
editInfo(){		// 变更
  this.info.name = 'tace'		// 此时watch得深度监听，oldVal和newVal相同
},
changeInfo(){ //替换
  this.info = {name:'tace'}		//此时watch得深度监听，oldVal和newVal可以正常监听
}
```



### Vue3没有filters属性

官方推荐用法

一、使用conputed

```js
data(){
  return {
    books:[
      {name:'编程书籍',price:100},
      {name:'算法书籍',price:90},
      {name:'数据书籍',price:80},
    ]
  }
},
computed:{
  filterBooks(){	//这里并不是让你拿到当前得参数进行传值，而是让你对原来得数据进行一层转换
    return this.books.map(item =>{
      //这里进行深拷贝，因为如果有两个计算属性，当他们都引用得是一个对象时，会互相影响
      const newItem = Obj.assign({},item)	
      newItem.price = "￥"+item.price
      return newItem
    })
	}
}
```

二、使用全局方法

```js
data(){
  return {
    books:[
      {name:'编程书籍',price:100},
      {name:'算法书籍',price:90},
      {name:'数据书籍',price:80},
    ]
  }
},
methods:{
  filterPrice(price){	
    return "￥"+price
	}
}
```

### 混入mixin

关于混入，即重复代码复用，减少代码的重复使用。
mixin中可以看作类似Vue实例，data/methods/生命周期函数等都可以在mixin中复用

#### 基本使用

1. 导出mixin

   ```js
   //myMixin.js
   
   export let myMixin = {
     data(){
   	return{
   		message:'hello mixin'
       }
     },
     methods:{
   	testFun(){
   		console.log(this.message)
   	}
     }
   }
   ```

2. 在需求页导入
   `import {myMixin} from "./mixin"`

   ```js
   new Vue({
       //mixins和data同级，注意这里有复数s
       mixins:[myMixin],
       create:{
           this.testFun()
   	}
   })
   ```

#### 合并规则

> 当遇到对象中相同的名称或选项时，vue会如何操作

1. 与data函数内对象冲突：以组件自身数据为准

   ```js
   data(){return{
       // 相同对象名，以组件自身为准
   }}
   ```

2. 生命周期冲突：两个钩子函数会被合并数组，两个函数都会被调用

3. 值为对象的选项，如 methods、components 和 directives，将被合并为同一个对象

   注意：如果对象中key相同，则取对象的键值对（如methods中有相同名称方法，则以组件自身的methods的方法为主）

#### 全局混入

> 如果想要为每一个组件都混入方法，可以使用全局混入

```js
// main.js
const app = createApp(App);

app.mixin({
  created() {
    console.log("全局的created生命周期");
  }
});

app.mount("#app");
```

### extends

> 允许声明扩展另外一个组件，类似于Mixins，但是不能复用template中的内容

```vue
// basePage.vue
<script>
export default {
  name: "basePage",
  data(){
    return {
      name:'vicer'
    }
  },
  methods:{
    changeName(){
      this.name = 'aaa'
    }
  }
}
</script>
```

```vue
// App.vue
<script>
import basePage from "./basePage";

export default {
  extends: basePage,
  name: "About"
}
</script>
```

类似mixin，在开发中**extends用的非常少**，在Vue2中比较**推荐大家使用Mixin**，而在Vue3中**推荐使用Composition API**。

