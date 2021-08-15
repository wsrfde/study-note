---
title: Vue3插槽
date: '2021-06-07'
categories:
 - Vue
tags:
 - Vue3
---

### slot插槽的基本使用

> 类似电脑的usb，使用接口给电脑增加不同的功能，键盘/音响，而不是直接在组件里添加  

**介绍：**  
默认自定义标签内不允许添加内容，而插槽可以让我们在内部添加内容

**封装方法：**  
抽取共性，保留不同。

**使用方法：**

直接在需要使用的位置中插入slot标签，即可在使用自定义组件中插入内容，内容会自动替换空slot		

```vue
// App.vue
<template>
  <slot-comp>
  	<h2>我是插入的内容</h2>
  </slot-comp>	
</template>
```

```vue
// slotComp.vue
<template>
  <div>
    <slot></slot> 	<!--  <h2>我是插入的内容</h2> -->
  </div>
</template>
```

**注意：组件标签中插入的无论插入多少值，都会一起作为替换元素替换slot**

**插槽默认值：**

> 当组件中没有插入任何元素，那么slot便会默认为其展示默认值（已设置默认值的前提下）

```html
<slot><b>默认显示我</b></slot>
```



<br>

### 具名插槽slot

> 当有多个插槽时，我们只想替换其中一个，就需要用到具名插槽了 

**使用方法：**

```vue
// slotComp.vue
<template>
  <div>
    <slot name="left"></slot>
    <slot name="right"></slot>
  </div>
</template>
```

```html
// App.vue
<template>
  <div>
    <slot-comp>
      <template v-slot:left>
        <h2>替换左边👈</h2>
      </template>
      
      <template v-slot:right>
        <h2>替换右边👉</h2>
      </template>
    </slot-comp>
  </div>
</template>
```

知识点：当`slot`不使用name属性时，`默认name为default`

#### 插槽语法糖

> 跟 v-on 和 v-bind 一样，v-slot 也有缩写

把`v-slot:`替换为`#`即可

```html
<slot-comp>
  <template #left>   <!-- 语法糖 -->
    <h2>替换左边👈</h2>
  </template>
</slot-comp>
```

#### 动态插槽名

> 当我们封装高级组件，需要根据配置来生成组件时，则需要用到动态配置插槽名

```vue
// App.vue
<template>
  <div>
    <slot-comp :name="name">	<!-- 别忘记传递属性 -->
      <template v-slot:[name]> <!-- 语法糖写法：#[name] -->
        <h2>动态名称:{{name}}</h2>
      </template>
    </slot-comp>
  </div>
</template>

<script>
import SlotComp from "./slotComp";

export default {
  name: "App",
  data(){
    return{
      name:'vicer'
    }
  },
  components: {SlotComp}
}
</script>
```

```vue
// slotComp.vue
<template>
  <div>
    <slot :name="name"></slot> <!-- => 动态名称:vicer -->
  </div>
</template>

<script>
export default {
  name: "slotComp",
  props:['name']
}
</script>
```

### 编译作用域

  父组件模板的所有东西都会在**父级作用域内编译**；  
  子组件模板的所有东西都会在**子级作用域内编译**。  

```vue
// App.vue
<template>
	<slot-comp>
  	<h2>{{name}}</h2>		<!-- 我给slot-comp组件中传递了插槽，那么能获取slot-comp组件的name嘛? -->
  </slot-comp>	
</template>

<script>
export default {
  name: "App",
  data(){
    return{
      name:'vicer'
    }
  }
}
</script>
```

```vue
// slotComp.vue
<template>
  <div>							<!-- 答案是不能的，因为编译作用域不同 -->
    <slot></slot> 	<!-- => <h2>vicer</h2> -->
  </div>
</template>

<script>
export default {
  name: "slotComp",
  data(){
    return{
      name:'tace'
    }
  }
}
</script>
```



### 作用域插槽

> 当我们希望子组件插槽得数据可以传递到父组件中时，可以使用作用域插槽

使用场景：在子组件中传入数组，并希望自定义数组中得每个item得数据展现形式

#### 获取子组件插槽数据

```vue
// App.vue
<template>
	<slot-comp :names="names">  <!-- 向子组件传递数据 -->
    <!-- slot默认name为default，可以不写default，如（v-slot="slotProps"）,这里写入只为了演示如何使用具名插槽作用域 -->
    <template v-slot:default="slotProps">
      <!-- 注意：这里传递过来的是一个对象 -->
      <p>自定义组件{{slotProps.index}}：{{slotProps.item}}</p>
		</template>
  </slot-comp>
</template>

<script>
export default {
  name: "App",
  data(){
    return{
      name:'vicer'
    }
  }
}
</script>
```

```vue
// slotComp.vue
<template>
  <div>				
    <template v-for="(item,index) in names" :key="item">
      <slot :index="index" :item="item"></slot>
    </template>
  </div>
</template>

<script>
export default {
  name: "slotComp",
  props:{
    names:{
      type:Array,
      default:()=>[]
    }
  },
}
</script>
```

### 独占默认插槽的缩写

* 如果我们的插槽是默认插槽default，那么在使用的时候` v-slot:default="slotProps"`可以简写为`v-slot="slotProps"`

  ```html
  <slot-comp :names="names">  
    <template v-slot="slotProps">
      <p>自定义组件{{slotProps.index}}：{{slotProps.item}}</p>
    </template>
  </slot-comp>
  ```

* 且如果我们的插槽**只有一个默认插槽时**，我们就可以将 v-slot 直接用在组件上

  ```vue
  <slot-comp :names="names" v-slot="slotProps">  
    <p>自定义组件{{slotProps.index}}：{{slotProps.item}}</p>
  </slot-comp>
  ```

  

