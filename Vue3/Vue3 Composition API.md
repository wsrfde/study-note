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

##### Ref和Reactive的小结

* (reactive/ref)包裹数据，在页面中都可以帮助其自动解包（除非用普通对象包裹ref变量）
* ref 在setup中需要通过`.value`改变属性的值，而reactive不需要
* ref支持所有数据类型，而reactive只支持对象/数组

**建议：当包裹原始数据类型时，使用ref；当包裹对象/数组时，使用reactive**

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

#### setup中使用ref访问元素

> 在$optionsAPI中可以通过this.$ref访问元素或组件,那么在setup中如何使用呢

使用方法：其实非常简单，我们只需要定义一个ref对象为null，然后绑定到元素或者组件的ref属性上即可；

```vue
<template>
  </div>
    <h2 ref="home">Home</h2>
    <button @click="getRefEl">获取当前ref</button>
  </div>
</template>

<script>
import {ref} from "vue";
export default {
  setup() {
    let home = ref(null)

    let getRefEl = () => {
      console.log(home.value)
    }

    return {
      home
      getRefEl,
    }
  }
}
```

#### 何时使用ref或reactive

### computed API

在Composition API中，computed和watch结合了以往的使用，增加了一些新的使用方式



**使用方式一：**

接收一个getter函数，并为 getter 函数返回的值，返回一个不变的 **ref 对象**

**使用方式二：**

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

### watch API

> 在Composition API中，我们可以使用watchEffect和watch来完成响应式数据的侦听；

* watchEffect用于自动收集响应式数据的依赖；（即自动监听`当前函数中`的**数据依赖**，并且`首次数据值也会被监听`到）
* watch需要手动指定侦听的数据源；（即手动选择监听目标）

#### watchEffect

##### 基本使用

注意：watchEffect虽然会自动收集函数中数据依赖，但不会收集setTimeout中的数据（vue版本3.0）

```js
import {ref, watchEffect} from "vue";

export default {
 setup() {
    let firstName = ref('张')
    let lastName = ref('三')

    watchEffect(()=>{
      console.log(firstName.value);	// 根据只监听当前函数中的数据依赖原则，其他数据改变时，不会触发watchEffect
    })
    let changeName = () => {
      firstName.value = '李'
      lastName.value = '四'
    }
 }
}
```



##### 停止监听

> 当我们需要停止监听函数时，可以利用watchEffect返回值函数，来调用该函数进行停止

```js
let age = ref(18)
let stopWatch = watchEffect(() => {
  console.log(age.value);
})
let changeName = () => {
  age.value++
  if (age.value > 20) {
    stopWatch()
  }
}
```

##### 清除副作用

> 当我们监听网络请求时，由于网络请求时间不确定性，数据还未到达我们便停止了侦听器。但网络请求终会到达，这时我们需要取消网络请求，就需要用到watchEffect的参数：onInvalidate

**onInvalidate介绍**：onInvalidate是watchEffect的一个参数，同时它又一个函数，当`watchEffect函数重新执行、或watchEffect被停止时，都会调用该函数`，所以onInvalidate一般用来做一些清除操作（因为onInvalidate是参数，所以可以被随意命名）



**和停止监听的区别：**stopWatch会完全停止监听，而onInvalidate更多用来清除上一次watchEffect中的操作

```js
let age = ref(18)

watchEffect((onInvalidate) => {
  let timer = setTimeout(function () {
    console.log('网络请求成功')
  }, 1000)

  onInvalidate(() => {	// 清除副作用
    clearTimeout(timer)
  })
  console.log(age.value);
})

let changeName = () => {
  age.value++
}
```

##### watchEffect的执行时机

> 当在watchEffect中，使用ref属性来获取某个元素或者组件时，watchEffect会被执行两次

```vue
<template>
	<h2 ref="home" class="aaa">Home</h2>
</template>

<script>
import {ref, watchEffect} from "vue";
  
export default {
  setup() {
    let home = ref(null)

    watchEffect(() => {
      console.log(home.value)//  第一次=> null  第二次=> <h2></h2>
    })

    return {
      home
    }
  }
}
```

因为：

* watchEffect在setup函数中会立即执行监听的数据。而此时DOM还没有挂在，所以显示null
* 而当DOM挂载时，会给title的ref对象赋值新的值，副作用函数会再次执行，打印出来对应的元素

* 调整watchEffect的执行时机

  > watchEffect除了可以传递函数，还可以传入一个配置项，来决定执行时机

  `{flush:'pre' | 'post' | 'sync'}`

  ```js
  watchEffect(() => {
    console.log(home.value)
  },{
    flush:'pre' 	// 默认是pre ，在元素挂在或更新之前执行
  //flush:'post'  // 在DOM挂载完成之后执行
  //flush:'sync'  // 通过执行，但这个性能很低，不建议使用
  })
  ```

  所以在watchEffect时，想要`监听ref实例`或者`需要等待DOM渲染完成才监听变化`时，就可以配置`{flush:'post'}`选项



#### watch

> watch的API完全等同于$options的watch选项的属性

**与watchEffect的区别**

* 懒执行副作用（第一次不会直接执行）

* 更具体的说明当哪些状态发生变化时，触发侦听器的执行

* 访问侦听状态newVal和oldVal值

##### 基本使用

* 监听getter函数中引用的可响应式对象（比如reactive或者ref）；

* 直接写入一个可响应式的对象，reactive或者ref（比较常用的是ref）

  ```js
  // 监听reactive对象
  setup(){
    let info = reactive({
      name: 'vicer',
      age: 18
    })
  	// 方法一：只监听对象属性
    watch(() => info.name, (newVal, oldVal) => {
      console.log(newVal, oldVal)
    })
    
    // 方法二：监听对象所有属性   
    watch(info, (newVal, oldVal) => {		// 缺点：不能监听到oldVal,reactive/ref都是如此
      console.log(newVal, oldVal)
    })
   	// 解决办法：
    // 响应式对象解构后变成普通对象或数组，即可正常监听到oldVal
    watch(() => ({...info}), (newVal, oldVal) => {
      console.log(newVal, oldVal)
    })
  }
  ```

  ```js
  // 监听ref对象
  setup(){
    let info = ref({
      name: 'vicer',
      age: 18
    })
  	// 方法一：只监听对象属性
    watch(() => info.value.name, (newVal, oldVal) => {
      console.log(newVal, oldVal)
    })
    // 或者直接使用ref监听单一属性
    let name = ref('vicer')
    watch(name, (newVal, oldVal) => {
      console.log(newVal, oldVal)
    })
    
  }
  ```

**总结：**

* `当只监听一个属性时，使用ref响应式`进行监听。

* 当`对象/数组中所有数据都要监听时，使用getter方法解构对象/数组，可对所有属性进行监听`。（如果`无需监听oldVal，直接传入响应式对象`即可）

##### 监听多个数据源

> 当监听参数中传入数组，即可监听多个数据源

```js
setup(){
  let name = ref('vicer')
  let age = ref(18)

  watch([name, age], (newVal, oldVal) => {
    // (newVal, oldVal)  可以解构成 [newName, newAge], [oldName, oldAge]
    console.log(newVal, oldVal)
  })
}
```

##### watch的选项

> watch的选项和之前的$options的选项相同

* `deep` 深层监听   (当直接传递reactive对象时，默认开启)
* `immediate`   立即执行

```js
setup(){
  let name = ref('vicer')

  watch(name, (newVal, oldVal) => {
    console.log(newVal, oldVal)
  },{
    deep:true,
    immediate:true
  })
}
```

### 生命周期

> 在setup中，可以使用对应的生命周期，和原来的相比，只需要在前面加上一个on即可，如原来的mounted改为onMounted

| 选项式 API      | Hook inside `setup` |
| --------------- | ------------------- |
| `beforeCreate`  | Not needed*         |
| `created`       | Not needed*         |
| `beforeMount`   | `onBeforeMount`     |
| `mounted`       | `onMounted`         |
| `beforeUpdate`  | `onBeforeUpdate`    |
| `updated`       | `onUpdated`         |
| `beforeUnmount` | `onBeforeUnmount`   |
| `unmounted`     | `onUnmounted`       |
| `activated`     | `onActivated`       |
| `deactivated`   | `onDeactivated`     |

提示：`在setup中不再需要beforeCreate和created`，因为在源码中，setup早于它们执行，所以这两个生命周期的代码直接写在setup函数中即可

使用方法：

```js
import {onMounted} from "vue";

  setup() {
    onMounted(() => {
      console.log('APP Mounted')
    })
  }
```

### Provide和Inject函数

>  Provide和Inject是向组件深度传递接收数据，Composition API也可以替代之前的 Provide 和 Inject 的选项

Provide使用：`provide('属性名称', 属性值)`

Inject使用：`inject('接收的provide属性名','默认值')`   （默认值是可选参数）

在使用Provide和Inject之前，我们先了解一下什么是**单项数据流**

#### 单向数据流

指父组件向子组件传递数据，但子组件不能逆向更改在父组件定义的数据，除非向父组件发送事件进行更改。这个便是单向数据流



为什么这样做：因为当父组件下面有多个子组件时，子组件可以随意更改父组件数据，当出bug时，难以定位出问题的组件，所以在组件数据最好在数据提供的位置来修改

#### 根据单向数据流使用Provide和Inject

```js
// 父组件
import {ref, readonly, provide} from "vue";

export default {
  name: "App",
  components: {
    Child
  },
  setup() {
    let name = ref('viceroy')
    provide('name', readonly(name))
    
    let changeName = ()=>{
      name.value = 'vicer'
    }
  }
}
```

```js
// 子组件
import {inject} from "vue";

export default {
  name: "Child",
  setup() {
    let name = inject('name')
    // 当父组件不使用readonly时，子组件是可以更改的，为了保证单项数据流，所以不推荐子组件直接更改父组件的数据
    let changeName = () => {
      name.value = 'tace'
    }
  }
}
```



### Use之逻辑抽离

> 类似react的hook，指把对应程序代码抽离成函数，当我们想使用某个程序时，只需执行并取出函数进行操作即可

#### useCounter

当我们使用setup时，所有函数和定义都放在setup中，我们可以使用useCounter方法进行代码抽离。

注意：在实际使用的时候可以不用叫useCounter，叫其他名字也可以，但是社区里面一般都称之为useCounter

```vue
<!-- App.vue -->
<template>
  <div>
    <p>计数：{{ count }}</p>
    <p>2倍计数：{{ doubleCount }}</p>
    <button @click="add">+</button>
    <button @click="less">-</button>
  </div>
</template>

<script>
import useCounter from './hook/useCounter'

export default {
  name: "App",
  setup() {
    let {count, doubleCount, add, less} = useCounter()

    return {
      count,
      doubleCount,
      add,
      less
    }
  }
}
</script>
```

```js
// useCounter.js
import {computed, ref} from "vue";

export default function () {
  let count = ref(0)
  let doubleCount = computed(() => count.value * 2)

  let add = () => count.value++;
  let less = () => count.value--;

  return {count, doubleCount, add, less}
}
```

#### useTitle

> 我们编写一个修改title的Hook

```js
// App.vue
import useTitle from "./hook/useTitle";

setup() {
  let titleRef = useTitle('哈哈')
  setTimeout(() => {
    titleRef.value = '呵呵'
  },2000)
}
```

```js
// useTitle.js
import {ref, watch} from 'vue'

export default function (title = '默认值') {
  const titleRef = ref(title)
  watch(titleRef, (newVal) => {
    document.title = newVal
  }, {
    immediate: true
  })
  return titleRef
}
```

#### useLocalStorage

> 来完成一个使用 localStorage 存储和获取数据的Hook

```js
// App.vue
import useLocalStorage from "./hook/useLocalStorage";

setup() {
	let data = useLocalStorage('info',{name:'vicer',age:18})
   data.value = 'tace'
}
```

```js
//  useLocalStorage.js
import {ref, watch} from 'vue'

export default function (key, value) {
  let data = ref(value)

  if (value) {
    window.localStorage.setItem(key, JSON.stringify(value))
  } else {
    data.value = JSON.parse(window.localStorage.getItem(key))
  }

  watch(data, (newVal) => {
    window.localStorage.setItem(key, JSON.stringify(newVal))
  })
  return data
}
```



1.20

