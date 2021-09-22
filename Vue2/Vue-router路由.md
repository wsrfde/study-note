---
title: Vue router路由
date: '2021-05-26'
categories:
 - Vue
tags:
 - Vue2
---

### 什么是前端渲染，什么是后端渲染

1. 后端路由  
由后端服务器html+css+java动态绑定数据并渲染好一个页面，直接发送到前端。
优点是有利于seo优化，缺点html和数据逻辑混在一起，难以维护

2. 前后端分离  
随着Ajax出现，后端只负责数据，前端去静态服务器请求html+css+js，然后调用api接口再进行数据处理进行渲染

3. 前端路由SPA阶段  
* 在前后端分离的基础上加了一层前端路由。  
* 前端去静态服务器请求html+css+js（全部资源），资源分成多个组件，点击一个url显示一个组件，页面进行不整体刷新。
* 前端路由的作用就是管理url和对应组件的映射关系

<a name='b'></a>

### 前端路由规则

#### URL的hash

URL的hash也就是锚点(#), 本质上是改变window.location的href属性.  

	location.hash = 'aaa'

#### HTML5中history模式：

**1. pushState**
把url push进去，类似栈结构，遵循先入后出

	history.pushState({},'','home')
**2. popState** 
路径的退回，类似栈结构，遵循先入后出

	history.popState()

**3. replaceState** 
替换url，不会保留历史记录（无法退回）

	history.replaceState({},'','home')

**3. go** 
history.go指跳转到指定页面

	history.go(1) 前进页面，等同于 history.forward()
	history.go(-1) 返回上一页，等同于 history.back()

**4. forward** 
前进下一页面

	history.forward()

**5. back** 
返回前一页面

	history.back()
<br>

### 安装和使用router

1. 安装vue-router，也可以通过脚手架安装  

		npm install vue-router --save
2. 配置router文件  
	1. 新建router文件夹index.js文件，导入路由和vue，并且调用 Vue.use(VueRouter)
	
	2. 创建路由，并且传入<a href='#a'>路由映射配置</a>
	
	   ```js
	   //传入路由
	   const routes = [
	   	  {
	   		//路由路径
	   		path:'/home',
	   		//映射组件
	   		component:Home
	   	  }
	   ]
	   //创建路由
	   const router = new vueRouter({
	   	  routes,
	   	  mode:'history'
	   	})
	   ```
	
	3. 默认导出，然后在main.js中导入，挂载
	
	   `export default router;`

<a name='a'></a>  

### 配置路由映射关系

1. 导入要映射的组件，也可以使用<a href="#c" >路由懒加载</a>
		`import Home from '../components/home'`
	
2. 配置路由路径和组件
		
	```js
	const routes = [
	    {
	        //路由路径
	        path:'/home',
	        //映射组件
	        component:Home
	    }
	]
	```
	
	
	
3. 使用路由  

 1. 通过&lt;router-link&gt;的属性to指定路由路径
			`<router-link to="/home">主页</router-link>`
	
 2. &lt;router-view&gt;指定展示位置
			`<router-view></router-view>`
			
### 路由的默认路径

让路径默认跳转到首页，多配置一个映射即可  

使用redirect重定向路由路径  

```js
{
	path:'/',
	redirect:'home'		//定义跳转
},
{
  path: '/home',
  name: 'Home',
  component: Home,	//定义home页面
},
```

### 路由的模式切换

之前在<a href='#b'>前端路由规则</a>中讲过有两种模式。  
* 在路由中默认使用的是URL中hash模式，但有个缺点，前面有个#锚记链接  

* 我们可以改成html5中的history模式
		
	```js
	const router = new vueRouter({
		  routes,
		  mode:'history'
	})
	```
	
	
### router-link的属性

* to：用于指定跳转的路径  
* tag：可以指定&lt;router-link&gt;渲染成什么标签，如tag='div'。默认渲染成a标签  
* replace: 当不需要用户前进或后退页面时，可用replace。该属性是替换url，不是push，所以不会留下history记录
* active-class: 当&lt;router-link&gt;对应的路由匹配成功时，会自动给当前元素设置一个router-link-active的class
  * 设置active-class可以修改默认的className。  

  * 也可以在路由实例配置中统一设置

     ```js
     new vueRouter({
       linkActiveClass:'active'
     })
     ```

###  路由代码跳转$router

除了使用&lt;router-link&gt;标签中to的跳转方式，我们还可以使用$router方法来控制url

```js
changeHome(){
  //push URL方法
  //this.$router.push('/home')
  //replace URL方法
   this.$router.replace('/home')
}
```

注意：使用函数方法push相同的路径会报错，解决思路，使用if判断，push路径等于当前路径则什么都不做，不等于才push进去路由
	
```js
if(this.$route.path != this.path){
	this.$router.push(this.path)
}
```

### $router和$route的区别

* $router是全局的路由对象，而$route是当前活动的组件对象  

* $router一般用来调用路由方法，如push。

* $route一般用来获取路由的属性，如params/query/path

  ```js
  //获取路由属性
  const getParamsData = this.$route.params;		//获取params（也就是动态路由）传递过来的属性
  const getQueryData = this.$route.query;			//获取query 方式传递过来的属性
  const getRoutePath = this.$route.path;			//获取当前路由的路径
  ```

  

<a name="c"></a>

### 路由懒加载

当我们打包js时文件太大，用户请求页面会导致加载时间过长，甚至出现页面空白。  
路由懒加载可以在打包时把对应组件打包成一个个js代码块，当路由被访问到时，才加载对应组件

懒加载方式：
1. 结合Vue的异步组件和Webpack的代码分析
		`const Home = resolve => { require.ensure(['../components/Home.vue'], () => { resolve(require('../components/Home.vue')) })};`
2.  AMD写法
		`const About = resolve => require(['../components/About.vue'], resolve);`
3.  **ES6写法**（推荐）
		`const Home = () => import('../components/Home.vue')`

### 路由嵌套

1. 创建对应的子组件, 并且在路由映射中配置对应的子路由.
		  
	```js
	{
			path:'/home',
			component:Home,
			children:[
			  {
				path:'news',
				component:News
			  }
			]
	}
	```
	
	
	
2. 在**子组件内部**使用router-link和router-view标签.
		`<router-link to="/home/news">新闻</router-link>
		<router-view></router-view>`
		

如果想在子路由中设置默认路径，直接在children中设置重定向，方法一样

### URL包括

**协议://主机:端口/路径?查询#片段**
`scheme://host:port/path?query#fragment`

### 传递及获取参数的两种方式

当传递数据较多的话，使用query类型

**1. params的类型:**  

* 配置路由映射: 

   ```js
   {
     path:'/user/:id',		//多个属性时  '/user/:id/:name'
   	component: User
   }
   ```

* 传递的方式: 在path(user)后面跟上对应的值，  
		
	1. 使用route-link标签的属性
	   
	   ```js
	   <router-link :to="'/user/'+useId">用户</router-link>
	   
	   data(){
	     return {
	       useId:'vicer'
	     }
	   }
	   ```
	   
	 2. 或者绑定点击事件调用函数
	
	    ```js
	    data(){d
	      return {
	    		useId:'vicer'
	    	}
	    },
	    methods:{
	      clickFun(){
	      	this.$router.push('/user/'+this.useId)  
	        //多个属性时  '/user/'+this.useId +'/'+ this.name
	        //多个属性时不建议用这种方式传递属性
	      }
	    }
	    ```
	
* 获取参数：

   方式 一： html内直接获取 `<p>{{$route.params.id}}</p>`

   方式二： 通过JS获取 

   ```js
   data(){
   	return {
   		id:null   //初始化一个变量
   	}
   },
   create:{
   	this.id = this.$route.params.id;		
   }
   //如果没有通过params获取到，请查看path设置是否此格式   path:'/user/:id'
   ```

   

**2. query的类型:**

* 配置路由映射: 

   ```js
   {
     path:'/user',
   	component: User
   }
   ```

* 传递的方式: path和query以对象方式传递  
		
	1. 使用route-link标签的属性
	   
	   ```html
	   <router-link :to="{path:'/profile/'+123,query:{name:'vicer',age:18}}">档案</router-link>
	   ```
	   
	2. 或者使用函数方式
	
	   **注意：query中不能使用this。可以保存进变量传入** 
	
	   ```js
	   const iid = this.id;
	   this.$router.push({
	     path:'/profile/'+123,
	     query: {name: 'vicer', age: 18,id:iid}
	   })
	   ```
	
* 获取参数：
	
		方式 一： html内直接获取`<p>{{$route.query.id}}</p>`
	
	方式二： 通过JS获取 
	
	```js
	data(){
		return {
			id:null   //初始化一个变量
		}
	},
	create:{
		this.id = this.$route.query.id;
	}
	
	```
	
	
	​	

### router源码解读

>源码大概了解即可，看不懂把上面的记住

源码解读：所有的组件都继承自vue的原型  

		Vue.prototype.test = function(){
			console.log('test')
		}
意为着我们在所有vue方法中都可以调用  

		this.test()
而Object.defineProperty(目标对象，被添加的key，被添加的value)
可以往对象中添加属性  

		Object.defineProperty(Vue.prototype,'$router',{
			get(){return this._routerRoot._route
		})
等同于  

		Vue.prototype.$route = get(){return this._routerRoot._route}
所以我们的$router和$route也是继承自vue原型.  
而且源码中还注册了两个全局组件  

		Vue.component('RouterLink',Link)  
		Vue.component('RouterVier',View)
所以我们可以大胆的使用这些组件了。

### 导航守卫

导航守卫主要用来监听路由的进入和离开，
vue-router提供了beforeEach和afterEach的钩子函数  

`from`：导航要离开的路由  
`to`：即将要进入的路由  
`next`：跳转到下一个钩子  

```js
//1. 在路由映射中添加meta元数据
{
  path:'/home',
  component:Home,
  meta:{
	title:'主页'
  }
}
//2. 在router配置文件中写导航守卫。前置钩子  
router.beforeEach((to,from,next) => {
  console.log(to);
  document.title = to.matched[0].meta.title
  //next()必须调用，否则无法跳转
  next();
})
```

注意：
* 后置钩子afterEach不需要调用next，使用方法和前置钩子一样
* 这两个导航守卫我们统一称为全局守卫
* 还有路由独享的守卫,组件内的守卫。具体查看官网[导航守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)

### keep-alive遇见vue-router

当&lt;keep-alive&gt;包含&lt;router-view&gt;时，所有路径匹配到的视图组件都会被缓存

当组件被缓存时，切换组件就不会重复进行创建(created)和销毁(destroyed)，组件会处于活跃(activate)或非活跃(dedestroyed)状态

而activated和deactivated是keep-alive的生命周期钩子函数，只有当使用keep-alive时才能使用这两个函数

**keep-alive属性**  

* include - 字符串或正则表达，只有匹配的组件会被缓存（默认包含所有组件）
* exclude - 字符串或正则表达式，任何匹配的组件都不会被缓存
	（当希望某个组件被创建/销毁时，在keep-alive添加exclude属性并赋值组件的name即可）


**需求**：当我们切换路由再返回时，希望保存原来的子路由位置，并进行返回

1. 使用keep-alive包裹router-view切换到activated生命周期函数，使组件不会被重复创建销毁

2. 利用每次切换回来调用activated的特性，把当前路径push进去
		
	```js
	activated() {
		  console.log('activated  ' + this.path);
		  this.$router.push(this.path);
	}
	```
	
3. 当离开路由时保存即将离开路由的地址。以便切换时回到原位置

    ```js
    //离开路由时，deactivated在载入下个路由才调用，保存的是下个路由。
    	//所以使用组件导航守卫，保存后再next()
    	beforeRouteLeave (to, from, next) {
    	  console.log('beforeRouteLeave  '+this.$route.path);
    	  this.path = this.$route.path;
    	  next();
    	}
    	
    ```
### 高级使用技巧  
使用计算属性判断路径是否相同

```js
computed:{
	isActive(){
		//下面的举例，只是为了抛砖引玉，indexOf !== -1则代表相同,返回true
		return this.$route.path.indexOf(this.path) !== -1;
	}
}
```
