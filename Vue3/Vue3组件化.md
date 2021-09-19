---
title: Vue3组件化
date: '2021-06-07'
categories:
 - Vue
tags:
 - Vue3
---

> 全局组件和局部组件与Vue2相同，这里只记录Vue3特有的方法

### 组件命名

在组件命名的时候可以用大驼峰`MyComp`，也可以用分隔符的方式`my-comp`去注册组件。

但是在使用的时候推荐使用**分隔符的方式去使用**

```Vue
<template id="my-app">
<!-- 使用分隔符方式 -->
	<my-comp></my-comp>
</template>
<template id="my-comp">
  我是全局组件
</template>

<script>
const MyComp = {
  template: '#my-comp'
}
const app = Vue.createApp(App)
app.component('MyComp', MyComp)
app.mount('#app');
</script>
```

###  组件通信

#### 父传子Props

> 除了之前使用的数据传递方法，我们还可以直接使用`v-bind`把对象传递过去

```vue
// App.vue 
<template>
  <child-comp :name="info.name"></child-comp>		 <!-- => vicer -->
  <child-comp v-bind="info"></child-comp>				 <!-- => vicer -->
</template>

<script>
export default {
  name: 'App',
  components: {ChildComp},
  data(){
    return{
      info:{
        name:'vicer',
        age:18
      }
    }
  }
}
</script>
```

```vue
// ChildComp.vue
<template>
  <div>{{name}}</div>
</template>

<script>
export default {
  name: "ChildComp",
  props:['name']
}
</script>
```

##### Props的使用方法

* 使用数组定义传值

  ```js
  props:['name','age']
  ```

* 使用对象定义传值

  ```js
  props:{
  	//基础的类型检查（`null`和`undefined`匹配任何类型）
  	propA:Number,
  	//多个可能的类型
  	propB:[String,Number],
  	//必填的字符串
  	propC:{
  		type:String,
  		required:true
  	},
  	//带有默认值的数字
  	propD:{
  		type:Number,
  		default:100
  	},
  	//带有默认值的对象
  	propE:{
  		type:Object,
  		//对象或数组默认值必须从一个工厂函数获取
  		default(){
  			return {message:'hello'}
  		}
  	},
  	//自定义验证函数
  	propF:{
  		validator:function(value){
  			//这个值必须匹配下列字符串中的一个
  			return ['success','warning','danger'].indexOf(value) !== -1;	
  		}
  	}
    //具有默认值的函数
    propG:{
      type:Function,
      //与对象或数组默认值不同，这不是一个工厂函数。这是一个用作默认值的函数
      default(){
        return 'Default function'
      }
    }
  }
  ```

#### 非Prop的Attribute

**定义：当我们给某个组件传递属性，该属性并没有定义对应的props或emits时，就被称为非Prop的Attribute**

例如：`class、style、id`等属性

##### Attribute继承

当子组件`有单个根节点`时，`非Prop的Attribute将自动添加到根节点的Attribute中`

```vue
// App.vue 
<template>
  <child-comp :name="name" data-num="123" id="childComp"></child-comp>		
</template>
```

```vue
// ChildComp.vue
<template>
  <div>				<!-- 非Prop属性：data-num="123" id="childComp"  -->
  	<h2>{{name}}</h2>
  </div>		
</template>

<script>
export default {
  name: "ChildComp",
  props:['name']
}
</script>
```

![image-20210609091102461](.\img\vue3-comp\not-props.png)

##### 禁用Attribute继承

> 当我们不希望组件的根元素继承attribute，可以在组件中设置 `inheritAttrs: false`

禁用继承的场景：需要将attribute应用于根元素之外的其他元素；

```vue
// ChildComp.vue
<template>
  <div>		<!-- 非Prop属性：data-num="123" id="childComp"  -->
    <h2 :id="$attrs.id">{{name}}</h2>  <!-- => <h2 id="childComp"></h2> -->
    <!-- 或者 -->
    <h2 v-bind="$attrs">{{name}}</h2>  <!-- => <h2 data-num="123" id="childComp"></h2> -->
  </div>		
</template>

<script>
export default {
  name: "ChildComp",
  inheritAttrs: false,		// 别忘记禁用
  props:['name']
}
</script>
```

##### 多个根节点继承

> 当我们子组件有多个根节点时，并不会进行默认绑定，需要`我们手动绑定某一个节点`，否则会报一个警告

手动绑定多根组件时，`无需设置inheritAttrs`属性

```vue
// ChildComp.vue
<template>
		<!-- 非Prop属性：data-num="123" id="childComp"  -->
    <h2 :id="$attrs.id">{{name}}</h2>  <!-- => <h2 id="childComp"></h2> -->
    <h2>{{name}}</h2>  
    <h2>{{name}}</h2>  
</template>
```

#### 子传父emit

在传递之前进行emits得定义

* 使用数组进行定义

  ```js
  emits:['add','sub']
  ```

* 使用对象进行定义

  ```js
  emits:{
    add: null,
    sub(num) {
      return num > 5 ? true : false  // 当不符合条件时，会报一个警告，数据仍然正常返回
    }
  }
  ```

问题：在vue2中无需进行定义即可传递，vue3中定义会提高性能？？



### 非父子组件通信

#### provide和inject深度传递

> `provide` 和 `inject` 主要在开发高阶插件/组件库时使用。并不推荐用于普通应用程序代码中。

这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在其上下游关系成立的时间里始终生效。

**provide：**用来传递数据

**inject：**用来接收数据，也可以设置来源和默认值



##### **基本使用方法**

```vue
// App.vue
<template>
  <child-comp></child-comp>
</template>

<script>
import ChildComp from "./ChildComp";

export default {
  name: 'App',
  components: { ChildComp },
  //provide: {
  //  name: 'vicer',
  //  age: 18,
  //}
  /**
  *  @desc:如果我们想要使用data中得数据，需要把provide改为函数，改变this得指向
  */
  provide(){
    return {
      name: 'vicer',
      age: 18,
      length: this.friends.length
    }
  },
  data() {
    return {
      friends: ['aaa', 'bbb', 'ccc']
    }
  },
}
</script>
```

```vue
// ChildComp.vue
<template>
  <div>{{name}}-{{age}}-{{length}}</div>		<!-- => vicer-18-3 -->
</template>

<script>
export default {
  name: "SunChildComp",
  inject:['name','age','length'],
}
</script>
```

##### 动态改变provide数据

```js
// App.vue
import ChildComp from "./ChildComp";
import {computed} from "vue";		// 这个computed是vue3的新特性

export default {
  name: 'App',
  components: {ChildComp},
  provide(){
    return {
      name: 'vicer',
      age: 18,
      length: computed(()=> this.friends.length)		// 改为计算属性，通过es6得箭头函数语法糖return值
    }
  },
  created() {
    setTimeout(()=>{
      this.friends.push('jack')
    },2000)
  },
  data() {
    return {
      friends: ['aaa', 'bbb', 'ccc']
    }
  },
}
```

```vue
// ChildComp.vue
<template>
	<!-- length通过computed返回了ref对象，所以需要取出里面得value -->
  <div>{{name}}-{{age}}-{{length.length}}</div>		<!-- =vicer-18-4 -->
</template>

<script>
export default {
  name: "SunChildComp",
  inject:['name','age','length'],
}
</script>
```

#### 全局事件总线mitt库

> Vue3从实例中移除了 $on、$off 和 $once 方法，所以我们如果希望继续使用全局事件总线，要通过第三方的库

Vue3官方有推荐一些库，例如 mitt

安装：`npm i mitt`

封装工具：方便我们在每个组件中引用

```js
// eventbus.js
import mitt from 'mitt';

const emitter = mitt();
// export const emitter1 = mitt();   //为了防止多个事件总线冲突，我们可以单独命名自己的事件总线

export default emitter;
```

使用：

```js
// 发射事件总线
import emitter from './utils/eventbus';

btnClick() {
   emitter.emit("why", {name: "why", age: 18});
}
```

```js
// 接收事件总线
import emitter from './utils/eventbus';
created() {
  // 接收单一事件总线
  emitter.on("why", (info) => {
    console.log("why:", info);
  });
	// 接收全部事件总线
  emitter.on("*", (type, info) => {
    console.log("* listener:", type, info);
  })
}
```

### 动态组件

> 当我们实现点击tab切换页面，有三种切换方式

切换方式：

1. 使用`v-if`显示不同的组件

2. 是使用 `component `组件，通过一个特殊的attribute `is` 来实现
3. 通过路由跳转



如果我们组件内容相似度高，但又需要不同的组件来进行展示时，这是我们可以使用第二种方法，**动态组件**

```vue
// App.vue
<template>
  <div>
    <template v-for="tab in tabs" :key="tab">
      <button @click="currentTab = tab">{{tab}}</button>
    </template>
    <div>
      当前页面：
      <!-- 通过is，动态赋值组件名。和其他组件一样，可以传递属性 -->
      <component :is="currentTab" :currentTab="currentTab"></component>	
    </div>
  </div>
</template>

<script>
import Mall from "./mall";
import Shop from "./shop";
import Cart from "./cart";
export default {
  name: "App",
  components: {Cart, Shop, Mall},
  data(){
    return {
      tabs:['mall','shop','cart'],
      currentTab:'mall'
    }
  },
}
</script>
```

注意：

1. 无论是使用切换方式1或2，切换时都会触发组件的生命周期

2. 使用`component`时，必须是已经注册的组件

### keep-alive 缓存组件

> 当我们切换组建后，希望某个组件不被销毁，继续保存原来的属性，就可以用到keep-alive

**基本使用：**

```vue
<keep-alive>  
  <component :is="currentTab"></component>	
</keep-alive>
```

**keep-alive的属性**

* `include ` - string | RegExp | Array。只有名称匹配的组件会被缓 存； 

* `exclude ` - string | RegExp | Array。任何名称匹配的组件都不 会被缓存； 

* `max ` - number | string。最多可以缓存多少组件实例，一旦达 到这个数字，那么缓存组件中最近没有被访问的实例会被销毁；

```vue
<!-- string ,多个组件可以用(英文逗号)分隔 -->
<keep-alive include='mall,shop'>  
  <component :is="currentTab"></component>	
</keep-alive>

<!-- RegExp  ,记得使用v-bind -->
<keep-alive :include='/mall|shop/'>  
  <component :is="currentTab"></component>	
</keep-alive>

<!-- Array ,记得使用v-bind -->
<keep-alive :include="['mall','shop']">  
  <component :is="currentTab"></component>	
</keep-alive>
```

**注意：keep-alive属性匹配的是组件的name属性，而非vue文件名称**

#### keep-alive的生命周期

keep-alive不会执行created或者mounted等生命周期函数

如果想要监听组件的当前状态可以使用`activated `和 `deactivated`的生命周期钩子函数



### 异步组件

> 在了解异步组件之前，我们先看一下webpack的代码分包

#### webpack的代码分包

> 因为组件和组件之间是通过模块化直接依赖的，那么webpack在打包时就会将组 件模块打包到一起（比如一个app.js文件中）,这个时候随着项目的不断庞大，app.js文件的内容过大，会造成首屏的渲染速度变慢

对于一些不需要立即使用的组件，我们可以单独对它们进行拆分，拆分成一些小的代码块chunk.js

```js
// import {sum} from "./math";   // 正常使用，会和app.js打包到一起

import('./math').then(res =>{		// 分包加载
  console.log(res.sum(2,3))
})
```

再次打包时，我们会发现dist文件夹中多了一个chunk.js文件，里面打包的便是sum函数

#### Vue异步组件

> 对于组件我们也希望通过异步的方式来进行加载（目的是可以对其进行分包处理），那么Vue中给我们提供了一个函数：defineAsyncComponent。

`defineAsyncComponent`的使用方法

* 使用工厂函数

  ```vue
  <script>
  // import AsyncComp from './AsyncComp.vue';		
    
  import { defineAsyncComponent } from 'vue';
  // defineAsyncComponent自动帮我们把组件在then中回调并return出去
  const AsyncComp = defineAsyncComponent(() => import("./AsyncComp.vue"))	
  
  export default {
    components: {
      AsyncComp,
    }
  }
  </script>
  ```

* 对象配置

  ```vue
  <script>
  import { defineAsyncComponent } from 'vue';
  
  // import AsyncComp from './AsyncComp.vue';
  const AsyncComp = defineAsyncComponent({
    loader: () => import("./AsyncComp.vue"),
    loadingComponent: Loading,	// 等待过程的loading
  	//errorComponent,	// 加载失败时要使用的组件，使用的很少
    delay: 2000,			// 在显示loadingComponent组件之前, 等待多长时间
    /**
     * err: 错误信息,
     * retry: 函数, 调用retry尝试重新加载
     * fail ：函数，调用时当前加载程序结束退出
     * attempts: 记录尝试的次数
     */
    onError: function(err, retry,fail, attempts) {}
  })
  
  export default {
    components: {
      AsyncComp,
    }
  }
  </script>
  ```

  更多对象配置请查看[官网](https://v3.cn.vuejs.org/api/global-api.html#definecomponent)，一般开发中更多使用工厂函数的方式

#### Suspense

> Suspense悬疑的意思，目前（2021-6-13）是一个实验特性，可能随时会更改

Suspense是一个内置的全局组件，该组件有两个插槽： 

* default：如果default可以显示，那么显示default的内容； 

* fallback：如果default无法显示，那么会显示fallback插槽的内容；

```vue
<template>
  <div>
    <suspense>
      <template #default>	<!-- 如果default中的组件不能展示，则展示fallback的组件，但是如果可以展示了，则展示default的组件 -->
        <async-category></async-category>
      </template>
      <template #fallback> <!-- 也就是说fallback相当于一个占位组件 -->
        <loading></loading>
      </template>
    </suspense>
  </div>
</template>
```

### 组件访问方式

#### $refs的使用

> 直接获取到元素对象或者子组件实例

```vue
<childComp ref="childComp"></childComp>

<script>
created(){
 console.log(this.$refs.childComp)  // 得到当前组件的所有data，methods等所有属性
}
</script>
```

注意：  

* ref如果绑定在组件中，那么获取到的是组件对象
* ref如果绑定到普通元素中，那么获取到是元素对象

##### $el

> 获取组件中的html元素

```js
console.log(this.$refs.childComp.$el)  // 得到当前组件的元素
```

#### $root

直接访问根组件，也就是vue实例

#### $parent

* 类似js操作Dom中的parent，实际开发用的很少
* 子组件应该尽量避免直接访问父组件的数据，因为这样耦合度太高了。

**注意： Vue3已移除$children属性**



### 组件的生命周期

> Vue3的组件销毁更名为卸载，钩子名称改为beforeUnmount和unmounted

生命周期：事务从诞生到消亡的整个过程  
以下是生命周期函数，可以在某个周期做某件事

```js
[
	'beforeCreate',
	'created',		//组件创建完成
	'beforeMount',
	'mounted',		//DOM树创建完成
	'beforeUpdate',	//数据更新前   可以更改数据
	'updated',		//DOM更新完成   最好不要在这里改数据，会发生死循环
	'beforeUnmount',	//事件的移除  清空定时器
	'unmounted',	//手动移除组件或路由切换，会发生视图改变
	'activated',	//钩子函数，需要作用keep-alive下。当前组件活跃状态
	'deactivated',	//钩子函数，需要作用keep-alive下。当前组件活跃状态t
  ]
```

### 组件的v-model

> 我们都知道v-model的作用，value和data的双向绑定，那么如果我们现在把input封装成组件如何使用v-model呢

基本使用：

```vue
// App.vue
<template>
  <div>
    <p>msg:{{ msg }} </p>
    <child-comp v-model="msg"></child-comp>	<!-- v-model默认传递的是modelValue属性 -->
    <!-- 上方的v-model等同于👇的写法 -->
    <!-- <child-comp :model-value="msg" @update:modelValue="msg = $event"></child-comp> -->
  </div>
</template>
```

```vue
// ChildComp
<template>
  <div>
    <input type="text" v-model="inputValue">
  </div>
</template>

<script>
export default {
  name: "childComp",
  props:{
    modelValue:{  // v-model默认传递的是modelValue属性 
      type:String
    },
  },
  emits:['update:modelValue'],
  computed:{
    inputValue:{
      set(val){
        this.$emit('update:modelValue',val)
      },
      get(){
        return this.modelValue
      }
    }
  },
}
</script>
```

#### v-model绑定多个属性

> 当我们希望给v-model绑定多个属性时，可以给v-model传递一个参数，这个参数名就是我们绑定的属性名称（v-model默认传递的是modelValue属性）

绑定属性名name：`v-model:name="name"`

```vue
// App.vue
<template>
  <div>
    <p>msg:{{ msg }} </p>
    <p>name:{{ name }}</p>
    <child-comp v-model="msg" v-model:name="name"></child-comp>
  </div>
</template>
```

```vue
// ChildComp
<template>
	<div>
    <button @click="changeName">按钮</button>
  </div>
</template>

<script>
export default {
  name: "childComp",
  props:{
    modelValue:{
      type:String
    },
    name:{
      type:String
    }
  },
  emits:['update:modelValue','update:name'],
  methods:{
    changeName(){
      this.$emit('update:name','Viceroy')
    }
  }
}
</script>
```

