### 前端路由规则

#### URL的hash

URL的hash也就是锚点(#), 本质上是改变window.location的href属性.  

	location.hash = 'aaa'

##### hash原理：

```html
<div id="app">
  <a href="#/home">home</a>
  <a href="#/about">about</a>

  <div class="content">Default</div>
</div>

<script>
  const contentEl = document.querySelector('.content');
  // 通过监听hash改变，进行页面改变
  window.addEventListener("hashchange", () => {
    switch(location.hash) {
      case "#/home":
        contentEl.innerHTML = "Home";
        break;
      case "#/about":
        contentEl.innerHTML = "About";
        break;
      default:
        contentEl.innerHTML = "Default";
    }
  })
</script>
```



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
history.go指**跳转**到指定页面   （和push/pop的区别是跳转）

	history.go(1) 前进页面，等同于 history.forward()  可以理解为go的快捷方式
	history.go(-1) 返回上一页，等同于 history.back()

**4. forward** 
前进下一页面

	history.forward()

**5. back** 
返回前一页面

	history.back()

##### history原理

```html
<div id="app">
  <a href="/home">home</a>
  <a href="/about">about</a>

  <div class="content">Default</div>
</div>

<script>
  const contentEl = document.querySelector('.content');

  const changeContent = () => {
    switch(location.pathname) {
      case "/home":
        contentEl.innerHTML = "Home";
        break;
      case "/about":
        contentEl.innerHTML = "About";
        break;
      default: 
        contentEl.innerHTML = "Default";
    }
  }

  const aEls = document.getElementsByTagName("a");
  for (let aEl of aEls) {
    aEl.addEventListener("click", e => {
      e.preventDefault();

      const href = aEl.getAttribute("href");
      // 使用history压栈式导航方式好处就是更改url，而页面自定义显示
      history.pushState({}, "", href);
      // history.replaceState({}, "", href);

      changeContent();
    })
  }
  // 监听页面出栈
  window.addEventListener("popstate", changeContent)
</script>
```

### 安装和使用Vue Router

安装：

`npm install vue-router@4`		(默认安装3的版本，所以这里需要指定4)

**@4指版本4的最新版本**

38分
