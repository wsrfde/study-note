## Vue3源码学习

### 虚拟DOM

#### 虚拟DOM的优势

> 目前框架都会引入虚拟DOM来对真实的DOM进行抽象，这样做有很多的好处：

1. 因为对于直接操作DOM来说是有很多的限制的，比如diff、clone等等，但是使用JavaScript编程语言来操作这 些，就变得非常的简单；
2. 是方便实现跨平台，包括你可以将VNode节点渲染成任意你想要的节点

#### 虚拟DOM的渲染过程

![](./img/vue-source/虚拟DOM渲染过程.png)

### 三大核心系统

* Compiler模块：编译模板系统； 

* Reactivity模块：响应式系统

* Runtime模块：也可以称之为Renderer模块，真正渲染的模块； 

  

![](./img/vue-source/三大核心.png)

协同工作方式：

1. 对.vue进行Compiler模块解析，生成randerer函数，并转化成Vnode

2. 对Vnode进行diff算法对比，并绑定响应式
3. 由虚拟DOM生成真实DOM进行数据展示

### 实现Mini-Vue

这里我们实现一个简洁版的Mini-Vue框架，该Vue包括三个模块：

* 渲染系统模块； `runtime -> vnode -> 真实dom`

* 可响应式系统模块； `reactive(vue2/vue3)`

* 应用程序入口模块;   `createApp(rootComponent).mount('#app')`

#### 渲染系统实现

渲染系统，该模块主要包含三个功能： 

**功能一**：h函数，用于返回一个VNode对象； 

**功能二**：mount函数，用于将VNode挂载到DOM上； 

**功能三**：patch函数，用于对两个VNode进行对比，决定如何处理新的VNode；

```html
<div id="app"></div>
<script src="./render.js"></script>
<script>
  // 1. 通过h函数创建vnode
  const vnode = h('div', { class: 'cont',id:'aaa' }, [
    h('p', null, '当前计数：100'),
    h('button', {class:'add-btn'}, '+1'),
    h('button', null, '-1')
  ]);
  // 2. 通过mount函数，将vnode挂载到#app上
  mount(vnode, document.querySelector('#app'));

  // 3. patch函数
  const vnode1 = h('div', { class: 'new-vnode',id:'aaa' }, [
    h('p', null, '哈哈'),
  ]);
  setTimeout(() => {
    patch(vnode, vnode1);
  }, 1000);
</script>
```

```js
// 功能一：h函数
const h = (tag, porps, children) => {
  return {
    tag,
    porps,
    children,
  };
};
// 功能二：mount函数
const mount = (vnode, container) => {
  // 创建标签DOM
  const el = (vnode.el = document.createElement(vnode.tag));
  // 给props设置属性/绑定事件
  if (vnode.porps) {
    for (let key in vnode.porps) {
      const value = vnode.porps[key];
      if (key.startsWith('on')) {
        el.addEventListener(key.slice(2).toLowerCase(), value);
      } else {
        el.setAttribute(key, value);
      }
    }
  }
  // 给child设置
  if (vnode.children) {
    if (typeof vnode.children === 'string') {
      el.textContent = vnode.children; //innerText 与样式有关, 会触发回流, textContent 不会
    } else {
      vnode.children.forEach((child) => {
        mount(child, el);
      });
    }
  }
  container.appendChild(el);
};
// 功能三：对两个VNode进行对比
const patch = (n1, n2) => {
  // 标签不同直接进行替换
  if (n1.tag !== n2.tag) {
    const parent = n1.el.parentElement;
    parent.removeChild(n1.el);
    mount(n2, parent);
  } else {
    const el = (n2.el = n1.el);
    const oldProps = n1.porps || {};
    const newProps = n2.porps || {};
    // 对新属性进行增加
    for (let key in newProps) {
      const oldValue = oldProps[key];
      const newValue = newProps[key];
      if (newValue !== oldValue) {
        if (key.startsWith('on')) {
          el.addEventListener(key.slice(2).toLowerCase(), newValue);
        } else {
          el.setAttribute(key, newValue);
        }
      }
    }
    // 对旧属性进行删除
    for (let key in oldProps) {
      if (!(key in newProps)) {
        if (key.startsWith('on')) {
          el.removeEventListener(key.slice(2).toLowerCase(), oldProps[key]);
        } else {
          el.removeAttribute(key);
        }
      }
    }

    // 对child进行对比
    const oldChild = n1.children || [];
    const newChild = n2.children || [];

    if (typeof newChild === 'string') { // 情况一：newChild是一个string
      // 边界情况（edge case）
      if (typeof oldChild === 'string') {
        if (newChild !== oldChild) {
          el.textContent = newChild;
        }
      } else {
        el.innerHTML = newChild;
      }
    } else {  // 情况二：newChild是一个数组
      if (typeof oldChild === 'string') {
        el.innerHTML = '';
        newChild.forEach((item) => {
          mount(item, el);
        });
      } else {
        // old = [v1,v2,v3]
        // new = [v1,v5,v6]
        const minLength = Math.min(oldChild.length, newChild.length);
        //1. 只对前面相同的节点进行patch操作。注意是minLength，所以不会遍历多出来的vnode
        for (let i = 0; i < minLength; i++) {
          patch(oldChild[i], newChild[i]);
        }

        //2. old.length > new.length
        if (oldChild.length > newChild.length) {
          oldChild.slice(minLength).forEach((item) => {
            el.removeChild(item.el);
          });
        }

        //3. old.length < new.length
        if (oldChild.length < newChild.length) {
          newChild.slice(minLength).forEach((item) => {
            mount(item, el);
          });
        }
      }
    }
  }
};
```

#### 响应式系统实现

##### watchEffect依赖收集

```js
class Dep {
  constructor() {
    this.subscribers = new Set();
  }
  depend() {
    if (watchData !== null) {
      this.subscribers.add(watchData);
    }
  }
  notify() {
    this.subscribers.forEach((effect) => effect());
  }
}
let dep = new Dep();

let watchData = null;
function watchEffect(effect) {
  watchData = effect;
  dep.depend(); // 添加依赖
  effect(); // 模拟watchEffect首次使用时会自动调用一次
  watchData = null; // 清空准备下次使用
}

let info = {
  counter:10
};
watchEffect(() => {
  console.log(info.counter * 2);
});
watchEffect(() => {
  console.log(info.counter * info.counter);
});

info.counter++;
dep.notify(); // 发送通知
```

