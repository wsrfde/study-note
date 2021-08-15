## render函数

> 当我们希望使用完全Javascript编程时（即不使用template模板编译html），可以使用render函数来进行渲染界面

### 页面的生成

* template方法：Vue在生成真实的DOM之前，会将我们的节点转换成VNode，而VNode组合在一起形成一颗树结构，就是虚 拟DOM（VDOM）

  `template`模板 => `VNode`  =>  `VDOM`虚拟DOM

* h方法：创建一个`vnode`函数

  

### h函数介绍

> h函数是render函数的一个特性，可以用来创建vnode函数来生成页面

**使用方法：**h()函数接收三个参数

1. `tag {String | Object | Function}` （必传）

   HTML标签名、组件、异步组件、函数式组件

2. `props {Object}`  (可选)

   标签的属性，如attribute、prop和事件对应的对象

3. `children {String | Array | object}`  (可选)

   如果为String，则为当前tag的内容。

   如果为传入Array，并用h()函数渲染，则为子VNodes

   如果传入Object，则为插槽对象

**注意事项：**

* 如果`没有props`，那么通常可以将`children作为第二个参数`传入； 
* 如果会产生歧义，可以将`null作为第二个参数传入`，将`children作为第三个参数`传入；

### h函数基本使用

使用方法一：单独定义render

```vue
<script>
import {h} from 'vue'

export default {
  name: "App",
  // data(){
  //   return{
  //     name:'viceroy'
  //   }
  // },
  setup() {
    let name = 'viceroy'
    return {
      name
    }
  },
  render() {
    return h('h2', {class: 'app'}, this.name)		// 当使用data或者setup取出里面的值都需要this获取。
    																						// setup也是函数返回对象，并且render函数绑定了this，所以可以取到
  }
}
</script>
```

使用方法二：在setup中使用render

```vue
<script>
  setup() {
    let name = 'viceroy'
    return ()=> h('h2', {class: 'app'}, name)
  }
</script>
```

### h函数计数器案例

```js
setup() {
  let count = ref(0)
  return () => h('div', null, [
    h('h2', {class: 'app'}, count.value),
    h('button', {onClick: () => count.value++}, '+'),	// 监听事件前面需要加on。后面的click可驼峰，可全小写
    h('button', {onClick: () => count.value--}, '-'),
  ])
}
```

### h函数之组件和插槽的使用

```js
// App.vue
import Child from "./Child";

export default {
  name: "App",
  render() {
    return h(Child, null, {
      default: props => h('span', null, `我是普通插槽`),
      other: props => h('p', null, `我是其他插槽，名称：${props.name}`),
    })
  }
}
```

```js
// Child.vue
export default {
  name: "Child",
  render() {
    return h('div', null, [
      h('h2', null, 'hello world'),
      this.$slots.default ? this.$slots.default() : h('span', null, '我是默认default插槽'),
      this.$slots.other ? this.$slots.other({name: 'viceroy'}) : h('p', null, '我是默认other插槽')
    ])
  }
}
```

## JSX

> 我们像上面一样写js代码太麻烦了，所以我们可以如果需要使用完全JS编程时，可以使用JSX的方式编写代码，babel会自动帮我们转成h函数的方式

### jsx的babel配置

`当前cli无需配置，如果需要配置的话再回来，按照如下方式配置`

安装babel插件：`npm install @vue/babel-plugin-jsx -D` 

```js
// babel.config.js
module.exports = {
  plugins:[
    "@vue/babel-plugin-jsx"
  ]
}
```

### jsx的使用

```vue
// App.vue
<script>
import Child from "./Child";

export default {
  name: "App",
  data() {
    return {
      count: 0
    }
  },
  render() {
    let add = () => this.count++;
    let less = () => this.count--;
    return (
      // jsx中使用变量或绑定事件，使用 {} 即可
        <div>
          <h2>计数：{this.count}</h2>	 
          <button onclick={add}>+</button>
          <button onclick={less}>-</button>
          <Child>
            {{default: props => <div>哈哈哈</div>}}
          </Child>
        </div>
    )
  }
}
</script>
```

```vue
// Child.vue
<script>
export default {
  name: "Child",
  render() {
    return (
        <div>
          <p>Child</p>
          {this.$slots.default ? this.$slots.default() : <span>默认插槽</span>}
        </div>
    )
  }
}
</script>
```

