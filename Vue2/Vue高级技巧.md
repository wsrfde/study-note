---
title: Vue高级技巧
date: '2021-06-10'
categories:
 - Vue
tags:
 - Vue2
---

### 事件总线

在非父子组件中是无法直接通信的。  
在vue中除了vuex可以统一管理状态，还有另外一种方法叫事件总线

**原理：**  
在vue原型中创建Vue实例，利用vue的事件总线进行发射接收

* 创建$bus
  	`Vue.prototype.$bus = new Vue()`
* 发射事件 
  	//参数可以省略
  	`this.$bus.$emit('事件名',参数)`
* 监听事件 
  	//监听事件总线一般会在生命周期函数mounted中监听
  	`this.$bus.$on('事件名',()=>{})`
  	

注意：  
使用$bus要在destroyed生命周期函数中使用$off销毁，要不然会叠加触发次数 
		

```js
	//全局销毁:当这个组件销毁的时候bus也跟着一起销毁
	this.$bus.$off('事件名')
	
	//局部销毁:如果你需多次调用总线，可以局部销毁
	//把箭头函数赋值给一个变量名，销毁时只需要销毁变量即可
	this.myFun = ()=>{fun()}
	this.$bus.$off('事件名称',this.myFun)
```


### 防抖debounce

在实际开发中我们可能每当用户输入一个字符就请求一次数据，这时一个很大的性能浪费，
可以当用户在300ms内持续输入时，取消数据请求，过了时间后再发送数据，缓解服务器压力。
这种操作就叫做防抖处理


```javascript
//ES6写法
debounce(fun,delay){
  let timer = null;
  return function (...args) {
    if(timer) clearTimeout(timer);
    timer = setTimeout(()=>{
      fun.apply(this,args)
    },delay)
  }
}

//ES5写法
function debounce(fun,delay){
  var time = null;
  return function (){
    var _this = this;
    var _arguments = arguments;
    if(time) clearTimeout(time);
    time = setTimeout(function(){
      fun.apply(_this,_arguments);
    },delay)
  }
}
```

**防抖函数讲解：**
**知识点：**seTimeout会返回一个ID池，通过clearTimeout内传入ID值来取消timeout

1. 第一次执行，没有timer跳过，直接执行setTimeout
2. 第二次执行，拿到了setTimeout第一次返回的timer，在delay时间内被清空。然后继续执行setTimeout
3. 第三十次，重复到第29次timer清空，执行setTimeout。这时没有第31次了，直接执行seTimeout内的函数
4. 通过apply指向当前function，是因为fun是形参，不是属于Vue实例的方法，要用Vue实例调用就得改变this指向

**使用方法**  

1. 传入函数和延时const backFun = debounce(function,delay)。  
   //这里function如果是一个方法不要加()，如果是待执行操作则嵌套再箭头函数内传递。
2. 调用返回函数backFun()
3. args是如果在调用backFun()时，里面可以传参数，(...args)时es6数组解构，可以传多个参数

**示例：**

```js
//默认已经编写好上面的防抖函数
let delay = 1000;
const myFun = function(e){
  console.log('我是防抖函数');
  console.log(e);			//tace
} 
const backFun = debounce(myFun,delay);

const btnEle = document.querySelector('#btn');
btnEle.onclick = function () {		//这里模拟多次点击调用防抖函数
	backFun('tace');		
}
```

注意事项

1. 如果我们在多个页面中使用防抖函数，可把防抖函数封装成函数导出
2. 把使用方法封装在mixin中，并把debouce返回的函数用data的属性来保存，不要用const或let
   1. 好处一、混入返回的是新的变量，不会影响原来的页面
   2. 好处二、如果在函数中调用，data保存可以防止防抖函数不断销毁重新创建的问题

### 节流throttle

节流函数是用来限制一个函数被调用的频率。使用场景如游戏：飞机大战，我们按键速度再快，飞机都一直按照固定频率发射子弹。

节流函数与防抖函数实现方式不同，我们采用时间戳的方式来完成：

1. 使用`last`变量来记录上次的执行时间
2. 每次执行前，把当前时间保存到`now`，当`now-last > interval`则运行函数
3. 函数执行时再将`now`赋值给`last`

```js
function throttle(fn,interval){
  var last = 0;
  return function (){
    var _this = this;
    var _arguments = arguments;
    var now = new Date().getTime();
    if(now - last > interval){
      fn.apply(_this,_arguments);
      last = now;
    }
  }
}
```

在节流函数中，最后一次值是不会被执行的，因为没有到达最终的时间，也就是`now - last < interval`没有被满足

我们可以在最后一次执行的设置一个setTimeout定时器，如果不满足条件`now - last < interval`则执行定时器的内容

```js
function throttle(fn,interval){
  var last = 0;
  var time = null;
  return function (){
    var _this = this;
    var _arguments = arguments;
    var now = new Date().getTime();
    if(now - last > interval){
      if(time) {
        clearTimeout(time);
        time = null;		//为了函数下次运行，设置time = null
      }
      fn.apply(_this,_arguments);
      last = now;
    }else if(time === null){		//这里还可以增加一个参数，如函数开头传入 isShowLast   ,这里用逻辑并（&&）判断是否为true，开启执行最后一次函数
      time = setTimeout(function(){
        time = null;		//执行后也设置为null，为了下次运行
        fn.apply(_this,_arguments);
      },interval)		//这里如果要严谨一点，可以改成interval - （now - last）
    }
  }
}
```







### 混入mixin

关于混入，即重复代码复用，减少代码的重复使用。
mixin中可以看作类似Vue实例，data/methods/生命周期函数等都可以在mixin中复用

**继承和混入的区别**：**继承是继承原来的变量，混入是返回一个新的变量**

使用示例

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
   };
   ```

   

2. 在需求页导入
   `import {myMixin} from "./mixin";`
   	  

   ```js
     new Vue({
   	  //mixins和data同级，注意这里有复数s
   	  mixins:[myMixin],
   	  create:{
   		  this.testFun()
   	  }
     })
   ```

### Vue封装插件

在vue中如果我们导入组件还需要再特定位置引用和设置参数等，我们可以把一个组件封装成一个插件，
直接一行代码就可以使用封装的组件

这里以toast显示插件为例。自己如果有更好的代码也可以用如下步骤进行封装

组件Toast的代码就是一个methods方法，调用显示文字。
这里就不引入了，自己按需创建。只讲解插件封装过程

```js
//3.导入Toast组件
import Toast from "./Toast";
//1.创建index文件，封装对象并导出
const obj = {};
//2.当使用Vue.use时自动执行install，并且会把Vue传递进去
obj.install = function (Vue) {
  //4.创建组件构造器
  const toastConstructor = Vue.extend(Toast);
  //5.通过new的方式，根据组件构造器可以创建出来一个组件对象
  const toast = new toastConstructor();
  //6.将组件对象，手动挂载到某一个元素上
  toast.$mount(document.createElement('div'));
  //7.这里toast.$el对应的就是div
  document.body.appendChild(toast.$el);
  //8.把toast挂载到vue原型上，通过vue的$toast方法来调用toast对象
  Vue.prototype.$toast = toast;
};

export default obj
```

9. 在main.js中导入，并使用vue进行安装
   	`import toast from './toast'`
   `Vue.use(toast)`	

10. 通过this.$toast来调用对象(组件)toast的方法把~

    `this.$toast.show();`



### Vue自定义指令

除了核心功能默认内置的指令 (`v-model` 和 `v-show`)

当你需要对普通 DOM 元素**进行底层操作**，这时候就会用到自定义指令

```js
// 注册一个全局自定义指令 `v-focus`
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})

//注册一个局部组件指令
directives:{
  focus:{
     // 指令的定义
    inserted:function(el){
      el.focus()
    }
  }
}
```

#### 钩子函数

一个定义指令对象，除了inserted，还可以使用如下几个钩子函数：

- **bind**：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- **inserted**：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- **update**：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新。
- **componentUpdated**：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
- **unbind**：只调用一次，指令与元素解绑时调用。

```js
//还可以写多个钩子函数
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  },
  update:function(el){
    console.log('我被更新了')
  }
})
```



#### 钩子函数参数

> 除了 `el` 之外，其它参数都应该是只读的

指令钩子函数会被传入以下参数：

- `el`：指令所绑定的元素，可以用来直接操作 DOM 。

- `binding`：一个对象，包含以下属性：

  - `name`：指令名，不包括 v- 前缀。

  - `value`：指令的绑定值，例如：v-my-directive="1 + 1" 中，绑定值为 2。

  - `oldValue`：指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。  

  - `expression`：字符串形式的指令表达式。例如 v-my-directive="1 + 1" 中，表达式为 "1 + 1"。

  - `arg`：传给指令的参数，可选。例如 v-my-directive:foo 中，参数为 "foo"。

  - `modifiers`：一个包含修饰符的对象。例如：v-my-directive.foo.bar 中，修饰符对象为 { foo: true, bar: true }。

- `vnode`：Vue 编译生成的虚拟节点。可以拿到当前指令所在的上下文 context，也就是可以拿到当前实例和方法

- `oldVnode`：上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。

**参数演示**

```html
<div id="app">
	<div v-demo:foo.a.b="message"></div>
</div>

<script>
// 注册一个全局自定义指令 `v-focus`
Vue.directive('demo', {
  bind: function(el, binding, vnode){
    console.log('bind...');
    var s = JSON.stringify
    el.innerHTML =
    'name: '       + s(binding.name) + '<br>' +					//name:'demo'
    'value: '      + s(binding.value) + '<br>' +				//value:'hello world'
    'expression: ' + s(binding.expression) + '<br>' +		//表达式 expression:'message'
    'argument: '   + s(binding.arg) + '<br>' +					//参数 argument:'foo'
    'modifiers: '  + s(binding.modifiers) + '<br>' +		//修饰符 modifiers:' { a: true, b: true }'
    'vnode keys: ' + Object.keys(vnode).join(', ')		  //vnode keys:'tag,data,children,text,elm等'
    }
})

var vm = new Vue({
  el:'#app',
  data: {
     message: 'hello world'
  }
});
</script>
```

**使用示例**

> 除了 `el` 之外，其它参数都是只读的，但指令的参数可以是动态的。

例如，在` v-mydirective:[argument]="value" `中，argument 参数可以根据组件实例数据进行更新！

```html
//注意，这里left和right用中括号包裹，通过data进行赋值
<div id="app">
  <div id="dynamicexample">
    <h3>在此部分内向下滚动 ↓</h3>
    <p v-pin:[left]="100">我固定在页面左侧100px处。</p>
    <p v-pin:[right]="100">我固定在页面右侧100px处。</p>
  </div>
</div>

<script>
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'		//设置定位
    var s = (binding.arg == 'left' ? 'left' : binding.arg)	//判断参数left or right
    el.style[s] = binding.value + 'px'	//赋值value。
    //这里el.style[s]用中括号不是点，是因为s是动态变量，不是固定参数
  }
})

var vm = new Vue({
  el:'#app',
  data:{
    right: 'right',
    left: 'left'
  }
});

</script>
```

**使用示例二**

> 使用自定义指令监听日历弹窗的点击消失

```html
<div id="app">
  <div v-click-out>
    <input type="text" >
    <div class="box" v-if='isShow'>
      <button>按钮</button>
    </div>
  </div>
</div>

<script>
  let vm = new Vue({
    el: '#app',
    data: {
      isShow: false,
    },
    methods:{
      focus(){
        this.isShow = true;
      },
      blur(){
        this.isShow = false;
      }
    },
    directives:{
      clickOut:{
        bind:function(el,bindings,vnode){
          el.fn = (e) =>{
            if(el.contains(e.target)){
              vnode.context['focus']()
            }else{
              vnode.context['blur']()
            }
          }
          document.addEventListener('click',el.fn);
        },
        unbind:function(el){
          document.removeEventListener('click',el.fn);
        }
      }
    }
  })
</script>
```



### 函数组件之jsx-render的使用

> 在组件的使用中，除了使用`.vue`文件，我们也可以使用render函数的形式来进行渲染`html`

#### render函数组件的用法

```vue
//app.vue
<template>
  <div class="app">
    <renderCom tagNum="1">我是h1</renderCom>
    <renderCom tagNum="2">我是h2</renderCom>
    <renderCom tagNum="3">我是h3</renderCom>
  </div>
</template>

<script>
  import renderCom from "./renderCom";

  export default {
    name: "app",
    components:{
      renderCom
    }
  }
</script>
```

```js
//renderCom.js
export default {
  props: {
    tagNum: {
      type: String
    }
  },
  //<h1>我是h1</h1>
  //<h2>我是h2</h2>
  //<h3>我是h3</h3>
  render(h) {
    let count  = 'test'+this.tagNum;
    //h('标签名',属性,内容)
    //更多render函数属性  https://cn.vuejs.org/v2/guide/render-function.html
    return h('h'+this.tagNum,{
      attrs: {
        name: count,
        id: '#' + count
      },
    },this.$slots.default)
  }
}
```

#### jsx的用法

> 当我们只想简单渲染几个标签时，使用render函数过面太过于复杂，我们可以使用`jsx`来进行渲染`html`

使用方法：

* <> 	代表html
* {}      代表js



```js
//renderCom.js
export default {
  props: {
    tagNum: {
      type: String
    }
  },
  methods:{
    getAttr(e){
      console.log(e.target.getAttribute('name'))
    }
  },
  render(h) {
    // let count  = 'test'+this.tagNum;
    // return h('h'+this.tagNum,{
    //   attrs: {
    //     name: count,
    //     id: '#' + count
    //   },
    // },this.$slots.default)

    
    let tag = 'h' + this.tagNum;
    //  <> 代表html   {} 代表js
    // name='1' 传递字符串属性  name={1}  传递Number属性
    return <tag name={this.tagNum} onclick={this.getAttr}> {this.$slots.default} </tag>
  }
}

```

[更多关于jsx的属性用法](https://github.com/vuejs/jsx#installation)



### 从0编写Element-UI中的Message组件

> 用过Element-UI的同学都知道，message消息提示的弹窗调用非常方便，那么我们如何自己封装一个自己的message组件呢

🌈思路：

查看Element UI中message的组件，当调用时发现body中多一个div，div中展示的则是message弹窗

知道了行为那么我们就可以按照它的方式来编写代码



💌提示：

本文章按照思路和步骤从0封装，直接想看最终代码的可以跳转到第5️⃣条



1️⃣、首先我们按照Element中menssage 的调用方法来编写methods

```vue
// App.vue

<template>
  <div id="app">
    <button @click="show">按钮</button>
  </div>
</template>

<script>
  import {Message} from "./components/Message.js";   //导入Message方法

  export default {
    name: 'App',
    methods: {
      show() {
    		 Message.success({
          msg: '你好',
          duration: 3000
        })
      }
    }
  }
</script>
```

2️⃣、在components中创建Message.js 和 MessageComp.vue

> 通过Vue实例$mount渲染模板的方法，来把MessageComp.vue组件挂载到内存中，并调用组件中的方法，来实现

```js
// components/Message.js
import Vue from 'vue'
import MessageComp from "./MessageComp.vue";

let Message = {
  success(options) {
    // 点击弹出层 将.vue文件挂在到内存中
    let instance = new Vue({
      render: h => h(MessageComp)
    }).$mount()
    // 将渲染好的内容添加到页面
    document.body.appendChild(instance.$el)
  }
}

export  {
  Message
}
```

```vue
// components/MessageComp.vue

<template>
  <div>
    我是弹窗
  </div>
</template>
```

现在已经可以正常弹窗了，但是这样是不是有点low啊，而且我们想要以数据驱动视图，而不是每次点击弹出的都是固定的东西

3️⃣、需要数据驱动视图，那么我们就可以需要在MessageComp.vue组件中添加数组，动态渲染弹窗，并且添加push数组的方法，当调用时添加弹窗

```vue
// components/MessageComp.vue

<template>
  <div>
    <div v-for="item in list" :key="item.id">
      {{item.msg+item.id}}
    </div>
  </div>
</template>

<script>
  export default {
    name: "MessageComp",
    data() {
      return {
        list: []
      }
    },
    mounted() {
      this.id = 0 // 需要一个id来找到当前列表，但并不需要进行对id进行数据监听，所以直接写到mounted中
    },
    methods: {
      add(option) {
        let item = { ...option, id: ++this.id }
        this.list.push(item)
        // 关闭弹窗
        item.timer = setTimeout(() => {	// 在add方法中调用setTimeout并赋值timer的好处是：
          this.closeMsg(item)						// 在closeMsg时，可以实时得到timer并进行clearTimerout
        }, item.duration)
      },
      closeMsg(item) {
        clearTimeout(item.timer)
        let itemIndex = this.list.indexOf(item)
        this.list.splice(itemIndex, 1)
      }
    }
  }
</script>

<style scoped>

</style>

```

```js
// components/Message.js
import Vue from 'vue'
import MessageComp from "./MessageComp.vue";

let Message = {
  success(options) {
    // 点击弹出层 将.vue文件挂在到内存中
    let instance = new Vue({
      render: h => h(MessageComp)
    }).$mount()
    // 将渲染好的内容添加到页面
    document.body.appendChild(instance.$el)
    instance.$children[0].add(options)		//MessageComp组件在instance中挂载，所以MessageComp是instance的子节点，通过		
  }																				//$children查找并调用其方法
}

export  {
  Message
}
```

但这样有一个坏处，就是每次调用success方法都会对重新创建实例进行渲染，并且性能也不高。

4️⃣、所以我们需要只创建一次实例即可，success方法只调用实例中的add方法

```js
// components/Message.js
import Vue from 'vue'
import MessageComp from "./MessageComp.vue";

let instance;

function setInstance() {
  instance = new Vue({
    render: h => h(MessageComp)
  }).$mount()
  document.body.appendChild(instance.$el)
}


let Message = {
  success(options) {
    !instance && setInstance()
    instance.$children[0].add(options)
  },
  warn() {
    // ...
  },
  info() {
    // ...
  }
}

export  {
  Message
}
```

 

5️⃣、这样一个自定义的组件基本上就完成了，但是我们看到Element-UI中的messsage组件是通过this.$message来进行调用的，那我们怎么做到呢

通过Vue.use()方法对message属性进行全局挂载，具体怎么做呢，show code

```js
// components/Message.js
import Vue from 'vue'
import MessageComp from "./MessageComp.vue";

let instance;

function setInstance() {
  instance = new Vue({
    render: h => h(MessageComp)
  }).$mount()
  document.body.appendChild(instance.$el)
}


let Message = {
  success(options) {
    !instance && setInstance()
    instance.$children[0].add(options)
  },
  warn() {
    // ...
  },
  info() {
    // ...
  }
}


export default {
  install(_Vue) { // 使用vue.use时默认回调用install方法，并传入Vue构造函数
    // install中一般会做三件事 1. 注册全局组件 2. 注册全局指令 3. 往原型上添加方法
    let $message = {}
    Object.keys(Message).forEach(key => { // 这里没有直接对$message=Message，防止改动$message时message也跟着改动，所以进行
      $message[key] = Message[key]      	// 深拷贝对象
    })
    _Vue.prototype.$message = $message
  }
}

```

```vue
// components/MessageComp.vue

<template>
  <div>
    <div v-for="item in list" :key="item.id">
      {{item.msg+item.id}}
    </div>
  </div>
</template>

<script>
  export default {
    name: "MessageComp",
    data() {
      return {
        list: []
      }
    },
    mounted() {
      this.id = 0 // 需要一个id来找到当前列表，但并不需要进行对id进行数据监听，所以直接写到mounted中
    },
    methods: {
      add(option) {
        let item = { ...option, id: ++this.id }
        this.list.push(item)
        // 关闭弹窗
        item.timer = setTimeout(() => {	// 在add方法中调用setTimeout并赋值timer的好处是：
          this.closeMsg(item)						// 在closeMsg时，可以实时得到timer并进行clearTimerout
        }, item.duration)
      },
      closeMsg(item) {
        clearTimeout(item.timer)
        let itemIndex = this.list.indexOf(item)
        this.list.splice(itemIndex, 1)
      }
    }
  }
</script>

<style scoped>

</style>

```

```vue
<template>
  <div id="app">
    <h3>第10课-message组件</h3>
    <button @click="show">按钮</button>

  </div>
</template>

<script>
  // import {Message} from "./components/Message.js";   //导入Message方法
  import Vue from 'vue'
  import Message from "./components/Message.js";
  Vue.use(Message)    // 使用全局方法 this.$message

  export default {
    name: 'App',
    methods: {
      show() {
        // Message.success({
        //   msg: '你好',
        //   duration: 3000
        // })
        this.$message.success({
          msg: '你好',
          duration: 3000
        })
      }
    }
  }
</script>

<style>

</style>

```



💣踩坑：注意，Message.js 首字母必须大写，否则提示message.js not found 。不知道是什么原因导致冲突，vue的bug还是开启了eslint的原因？

我的环境：win10，webstorm，eslint。有知道原因的可以告知感激不尽

🐱‍💻🐱‍🐉🐱‍👓🐱‍👓
