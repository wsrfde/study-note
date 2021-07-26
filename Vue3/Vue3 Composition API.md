---
title: Vue3 Composition API
date: '2021-06-07'
categories:
 - Vue
tags:
 - Vue3
---

## Composition API

为什么我们要使用composition API，而放弃传统的options API？

答：代码分散，实现一个功能要分散到各个属性中，当一个功能还好，如果多个功能的话将会使组件变复杂难以维护，代码难以阅读理解。

当使用Composition API时会帮我们把**同一个逻辑关注点相关的代码收集在一起**，也有人把Vue Composition API简称为**VCA**

### Composition API基础使用

#### setup函数

> **注意：setup函数中不能使用this！**

##### setup函数的参数

* `props`  通过props传递的属性，可以通过props参数获取
* `context`  上下文，也称之为是一个**SetupContext**，其中包含三个属性
  * `attrs`：所有的非prop的attribute（如id、class，没有通过props传递的属性）
  * `slots`：父组件传递过来的插槽（这个在以render函数返回时会有作用）
  * `emit`：当我们组件内部需要发出事件时会用到emit（我们不能访问this，及通过 this.$emit发出事件）

```vue
// home.vue
<script>
export default {
  name: "Home",
  props: {
    msg: {
      type: String,
      default: ''
    }
  },
  // (props, context)
  setup({msg}, {attrs, slots, emit}) {	// 对象结构
    console.log(msg)	// => hhh
    console.log(attrs)	// => {id: "home"}
  }
}
</script>
```

```vue
// A
<home id="home" msg="hhh"><p>我是插槽</p></home>
```



##### setup函数的返回值

> setup的返回值可以在模板template中被使用，也就是说我们无需再使用data属性

```vue
<template>
  <div>
    <span>home：{{message}}</span>
  </div>
</template>
<script>
  setup({msg}, {attrs, slots, emit}) {
    let message =  msg + '111'
    return {
      message
    }
  }
</script>
```

#### setup函数中定义响应式

> 如果我们想要在setup中定义响应式，可以引用以下两种方法，推荐Ref API

##### Reactive API

> 我们在编写data选项时，其内部响应式处理就是交给了Reactive函数

**Reactive API的缺点：**要求我们必须传入的是**一个对象或者数组类型**，否则会报警告并无法展示

```vue
<template>
  <div>
    <!--  reactive  -->
    <p>{{ status.count }}</p>
    <button @click="changCount">按钮</button>
  </div>
</template>

<script>
import { reactive } from "vue";

export default {
  name: "Home",
  setup(props, context) {
    let status  =  reactive({
      count:100
    })
    let changCount = ()=>{
      status.count += 1
      console.log(status.count)
    }
    return {
      status,
      changCount
    }
  }
}
</script>
```

##### Ref API

> ref 会返回一个可变的响应式对象，内部数据通过value来进行维护

**Ref API注意事项：**

* 由于ref数据通过value维护，所以在setup函数中我们以然需要使用`ref.value`的方式去使用

* 在template中引入ref的值时，Vue会自动帮助我们进行解包操作，即我们不需要在template中使用`ref.value`，直接使用`ref`即可

```vue
<template>
  <div>
    <!-- ref  -->
    <p>{{ count }}</p> 	<!-- => ref变量：自动解包 -->
    <!-- 普通obj包含ref变量  -->
    <p>{{ obj.count.value }}</p>  <!-- => 普通的obj包含ref变量：不会解包 -->
    <!-- ref obj -->
    <p>{{ newObj.count }}</p>  <!-- => ref对象包含(ref/普通)变量：自动解包 -->
    <!--  reactive  -->
    <p>{{ status.count }}</p>  <!-- => reactive对象包含(ref/普通)变量：自动解包 -->
    
    <button @click="changCount">按钮</button>
  </div>
</template>

<script>
import {reactive, ref} from "vue";

export default {
  name: "Home",
  setup(props, context) {
    let count = ref(100)	// count.value += 1
    let obj = {   // obj.count.value += 1
      count
    }
    let newObj = ref({  
      count							// newObj.value.count += 1
      //count:100       // newObj.value.count += 1  :包含非ref变量，使用方式一样
    })
    let status = reactive({ // status.count += 1
      count
    })
    let changCount = () => {
			// 对上面的变量进行+1，具体方法看对应变量后面的注释
    }
    return {
       count,
         obj,
      newObj,
      status,
      changCount
    }
  }
}
</script>
```

**总结：**

* Ref 在setup中无论以那种形式都需要通过value改变属性的值。

* (reactive/ref)对象包裹变量，都可以帮助其自动解包，但在setup中使用方法略有不同，`ref支持所有数据类型，而reactive只支持对象`

##### Readonly API

> 当我们使用一个响应式对象，希望传给其他地方（组件）使用，但是不想让其修改时，可以使用readonly

原理：`readonly会返回原生对象的只读代理。它依然是一个Proxy，只是一个proxy的set方法被劫持，不能对其更改`

使用方法：readonly方法可以传入三种**对象类型**的参数

* 普通对象

* reactive返回的对象

* ref的对象

```js
import {ref,readonly} from 'vue'

 setup(){
   // 普通对象
    const obj1 = {
      name:'vicer'
    }
    const readObj1 = readonly(obj1)
    
    // 传入reactive
    const obj2 = reactive({
      name:'vicer'
		})
    const readObj2 = readonly(obj2)
    
    // 传入ref对象
    const obj3 = ref('vicer')
    const readObj3 = readonly(obj3)
 }
```

使用场景：

我们给其他组件传递数据，但是不希望其他组件修改内容，只希望在父组件修改时，就可以使用readonly了

```vue
<template>
	<home :info='readonlyInfo' />
</template>

<script>
	import {reactive,readonly} from 'vue'
  
  import Home from './pages/Home.vue'
  
  export default{
    components:{
      Home
    },
    setup(){
      const info = reactive({
        name:'vicer',
        age:18
      })
      const readonlyInfo = readonly(info)
      return {
        readonlyInfo
      }
    }
  }
</script>
```

##### Reactive的其他API

* `isProxy`

  检查对象是否是由 reactive 或 readonly创建的 proxy。 

* `isReactive `

  检查对象是否是由 reactive创建的响应式代理： 

  如果该代理是 readonly 建的，但包裹了由 reactive 创建的另一个代理，它也会返回 true；

* `isReadonly`

  检查对象是否是由 readonly 创建的只读代理。 

* `toRaw`

  返回 reactive 或 readonly 代理的原始对象（codewhy不建议保留对原始对象的持久引用）

  `toRaw` 方法从 `reactive` 对象中获取到的是**原始数据**，因此我们就可以很方便的通过**修改原始数据的值**而**不**更新视图来做一些性能优化了。

  **注意**：当`toRaw`方法接收的参数是`ref`对象时，需要加上`.value`才能获取到原始数据对象

* `shallowReactive `

  创建一个响应式代理，它跟踪其自身 property 的响应性，但不执行嵌套对象的深层响应式转换。

  （即只对对象的第一层进行响应式，深层的对象不进行响应式变化，可以理解为浅拷贝形式的浅响应式） 

* `shallowReadonly `

  创建一个 proxy，使其自身的 property 为只读，但不执行嵌套对象的深度只读转换。

  （即只对对象第一层进行只读设置，深层的对象可以正常读取、写入）

  

##### Reactive的补充API

> 当我们使用ES6的解构语法，对reactive返回的对象进行解构时，返回的对象不再是响应式，如果想要继续使用响应式对象，可以使用toRefs或toRef属性

* `toRefs`

  解构对象中的全部属性，使用toRefs

  ```js
  const info = reactive({	// 注意：toRefs只能对reactive响应式对象使用
    name:'vicer',
    age:18
  })
  const {name,age} = toRefs(info)
  ```

* `toRef`

  当我们只希望转换reactive对象中某一个属性时，可以使用toRef

  ```js
  const age = toRef(info,'age')
  const {name} = info
  const changeAge = ()=>{
    info.age++;
    // 或者
    // age.value++;
  }
  ```

注意：toRefs和toRef，只能用在reactive对象



##### Ref的其他API

* `isRef`

  判断值`是否是一个ref对象`

* `unref`

  如果我们想要获取一个ref引用中的value，还需要编写一层value来获得值，也可以通过unref方法，直接获得value本身：

  `如果参数是一个 ref，则返回内部值，否则返回参数本身 `

  这是` val = isRef(val) ? val.value : val` 的**语法糖函数**；

* `shallowRef`

  创建一个浅层的ref对象， 即只创建对象第一层响应式

* `triggerRef`

  手动触发和 shallowRef 相关联的副作用

  ```js
  // shallowRef 的副作用是只更新第一层对象的响应式，深层更改时不触发视图更新
  // 可以使用triggerRef来手动更新视图
  const info =shallowRef({	//age属于对象ref的value.age属性，所以属于深层对象，浅响应并不会触发视图更新
    age:18
  })
  
  const changeAge = ()=>{
    info.value.age++
    // 手动触发视图更新
    triggerRef(info)
  }
  ```

##### customRef

创建一个**自定义的ref**，并对其**依赖项跟踪**和**更新触发**进行显示控制： 

* 它需要一个工厂函数，该函数接受 `track`和 `trigger`函数作为参数； 

* 并且工厂函数应该返回一个带有 `get`和 `set`的对象；

```js
// 使用customRef创建原始ref方法
import { customRef } from 'vue'

export default function (value) {
  return customRef((track, trigger) => {
    return {
      get() {
        track()		// track 跟踪依赖
        return value
      },
      set(newValue) {
        value = newValue
        trigger()		// trigger 触发更新
      }
    }
  })
}
```

```js
// 使用customRef创建防抖函数
// useDebounceRef.js
import {customRef} from 'vue'

export default function (value) {
  let timer = null
  return customRef((track, trigger) => {
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        clearTimeout(timer)
        timer = setTimeout(()=>{
          value = newValue
          trigger()
        },500)
      }
    }
  })
}


// 使用：引入需要的.vue文件
import useDebounceRef from "./useDebounceRef";
export default {
  setup() {
    let name = useDebounceRef('vicer')
  	return {name}  
  }
}
```

#### setup函数中使用computed和watch

##### computed

> 计算属性改为引入方式使用

使用方式一：

接收一个getter函数，并为 getter 函数返回的值，返回一个不变的 **ref 对象**

使用方式二：

接收一个具有 get 和 set 的对象，返回一个**可变的（可读写）ref 对象**；

```vue
<script>
import {ref, computed} from "vue";

export default {
  name: "App",
  setup() {
    let firstName = ref('张')
    let lastName = ref('三')
    
    // 使用方式一：直接返回getter
    // let fullName = computed(() => firstName.value + ' ' + lastName.value)

    // 使用方式二：传入一个对象，包含getter和setter
    let fullName = computed({
      get: () => firstName.value + ' ' + lastName.value,
      set: (newVal) => {
        let names = newVal.split(' ')
        firstName.value = names[0]
        lastName.value = names[1]
      }
    })
    
    let changeName = () => {
      fullName.value = '李 四'
    }

    return {
      fullName,
      changeName
    }
  }
}
</script>
```

##### watch

1：08
