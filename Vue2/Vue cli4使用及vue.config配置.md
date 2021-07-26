---
title: Vue cli4使用及vue.config配置
date: '2021-05-26'
categories:
 - Vue
tags:
 - Vue2
---

## Vue Cli脚手架

> vue cli4声称0配置，虽然帮我们配置了很多东西，但在实际开发中还是要做一些自己的配置，这里是自己根据官方文档及自己一些实战总结，记成博客，以备不时之需

### 安装及使用

安装：`npm i @vue/cli -g`

使用：`vue create 项目名称`

创建流程：Enter确认下一步，空格键确认选中（自己根据项目需要选择即可）

<img src=".\img\cli4\cli创建项目的过程.png" alt="image-20210606185335719" style="zoom: 80%;" />

创建完成目录文件介绍：

`.browserslistrc`：适配的目标浏览器，如果想要查看配置，可以在github中搜文件名（一般默认即可）

`.gitignore`：github提交代码时需要忽略的文件

### webpack源码

> 我们会好奇，在cli中已经帮我们配置好了那么多loader等配置，那么这些配置都在哪里呢

在`node_modules/@vue/cli-service/lib/Service.js`，在该文件中定义了一个变量`builtInPlugins`

变量中定义了`['./config/base','./config/css','./config/prod','./config/app']`

也就是说把这些文件中的基础配置marge到一起，最后形成cli中所描述的0配置。

##### 查看配置

1. 在命令行中运行`vue ui`
2. 在任务选项卡中，点击**inspect**（检查webpack 配置），并运行，即可查看当前项目的webpack的配置

### vue.config.js文件配置

#### loader介绍

将对应的代码进行转换处理，如浏览器不识别typescript，less，sass等，我们需要loader把我们的代码转换成浏览器识别的js和css。或将图片转换成base url等

#### loader安装

通过npm安装需要使用的loader，如`npm i less-loader less`     

**注意：**安装时要安装对应的loader，如`less-loader`，还要安装对应的语言插件，如`less`。

​			如果不需要的话则不用安装，如css只需要安装`css-loader`即可

#### loader配置

> 在cli4中，内置的webpack会预配置我们基本的loader，如果在使用过程中内置loader没有达到我们的要求，也可以进行手动安装相应loader 

##### 查看loader配置

1. 在命令中运行`vue ui`
2. 在任务选项卡中，点击**inspect**（检查webpack 配置），并运行，即可查看当前项目的webpack的配置



或直接运行命令`vue inspect`

##### 修改loader配置

> 这里我们以修改images为例

1. 在查看webpack配置中找到image的loader规则

   ![](./img/cli4\image的loader原配置.png)

   可以看到，image的默认使用`url-loader`,当小于`limit`使用base64编码，大于`limit`时使用`file-loader`配置，这些配置在cli4中已经默认帮我们配置好

2. 在`vue.config.js`中修改配置

   现在我们要修改`limit `的大小

   ```js
   // vue.config.js
   module.exports = {
     chainWebpack: config => {
       config.module
           .rule('images')	//上面图片中箭头指向，要修改的规则
           .use('url-loader')		//使用webpack中的哪个loader配置
           .loader('url-loader')	//配置当前loader，一般与use相同（可以注释，注释后系统自动使用cli4中的默认loader）
           .tap(options => {
             options.limit = 5240;	//要修改的配置
             // options.fallback.loader = "file-loader";
         		console.log(options);
             return options;		//修改完成后记得return才会生效
           })
     }
   }
   ```

3. 重新运行**inspect**，查看`webpack`的变化

   <img src=".\img\cli4\image的loader修改后.png" style="zoom:80%;" />

##### 添加loader

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    // GraphQL Loader
    config.module
      .rule('graphql')
      .test(/\.graphql$/)
      .use('graphql-tag/loader')
        .loader('graphql-tag/loader')
        .end()		//结束
      // 你还可以再添加一个 loader
      .use('other-loader')
        .loader('other-loader')
        .end()
  }
}
```

修改后的示例，可以看到新增的loader

![](.\img\cli4\新加的loader.png)

##### 删除loader配置

###### 清空loader配置

在某些情况下我们想要清空loader中的某一配置，可以使用clear()函数

<img src=".\img\cli4\清除loader配置.png" style="zoom:80%;" />

如css中的oneOf，我们可以在oneOf后面+s，并使用clear()，代码如下

```js
const css =  config.module.rule('css');
css.oneOfs.clear();
```

###### 删除单个配置

如我们想要删除css.oneOf下面的resourceQuery，可以这样操作

```js
const css =  config.module.rule('css');
css.oneOf('vue-modules').delete('resourceQuery')
```



##### 替换loader

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    const svgRule = config.module.rule('svg')

    // 清除已有的所有 loader。
    // 如果你不这样做，接下来的 loader 会附加在该规则现有的 loader 之后。
    svgRule.uses.clear()

    // 添加要替换的 loader
    svgRule
      .use('vue-svg-loader')
        .loader('vue-svg-loader')
  }
}
```



#### 插件配置

##### 查看插件配置

> 与查看loader配置方法相同，运行`vue ui`后点击**inspect**查看**plugins**即可。或者直接运行命令`vue inspect`

##### 添加插件配置

方法 一：

```js
// vue.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

module.exports = {
  configureWebpack: {
    plugins: [
      new BundleAnalyzerPlugin()
    ]
  }
}
```

方法二：

```js
// vue.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

module.exports = {
  chainWebpack: config => {
    config
      .plugin('BundleAnalyzerPlugin')
      .use(BundleAnalyzerPlugin)
      .end()
  }
}
```



##### 修改插件配置

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config
      .plugin('html')
      .tap(args => {
        args[0].template = '/Users/username/proj/app/templates/index.html'
        return args	//返回修改的配置
      })
  }
}
```

##### 删除插件

<img src=".\img\cli4\删除插件.png" style="zoom:80%;" />

```js
config.plugins.delete('define');
```

#### 更改webpack参数

当我们想要更改某一参数时，可以使用set方法

```js
//可链式调用
config.output.set('path','testFile');
```

#### 更多chainWebpack调用方法

> 如果以上方法无法满足你的使用需求，可以查看中文文档

[中文文档](https://github.com/Yatoo2018/webpack-chain/tree/zh-cmn-Hans)

#### configureWebpack的用法

> 像webpack中一样配置即可，如module，plugins等

如果这个值是一个对象，则会通过webpack-merge合并到最终的配置中去

```js
configureWebpack: {
    output: {
      //...
    },
    module: {
      rules: [
        //...
      ]
    },
    plugins: [
			//...
    ]
  }
```

如果这个值是一个函数，则会接收被解析的配置作为参数。

```js
// vue.config.js
module.exports = {
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      // 为生产环境修改配置...
    } else {
      // 为开发环境修改配置...
    }
  }
}
```



#### webpack源文件地址

有些外部工具可能需要通过一个文件访问解析好的 webpack 配置，比如那些需要提供 webpack 配置路径的 IDE 或 CLI。在这种情况下你可以使用如下路径：

```
<projectRoot>/node_modules/@vue/cli-service/webpack.config.js
```

### 环境变量

> 如果我们需要在生产和开发环境中需要定义不同的全局变量，则需要用到`.env`文件

我们都知道Vue cli项目中有三个模式

- `development` 模式用于 `vue-cli-service serve`	（运行网页）
- `production` 模式用于 `vue-cli-service build` 和 `vue-cli-service test:e2e` （打包和测试）
- `test` 模式用于 `vue-cli-service test:unit`  （测试）

每个模式下都会将 `NODE_ENV` 的值设置为模式的名称（如在 development 模式下 `NODE_ENV` 的值会被设置为 `"development"`）

```js
//打印run servicer的模式
console.log(process.env.NODE_ENV); 		//development
```

可以通过为 `.env` 文件增加后缀来设置某个模式下特有的环境变量。比如，如果你在项目根目录创建一个名为 `.env.development` 的文件，那么在这个文件里声明过的变量就只会在 development 模式下被载入。



**更多使用方法看[官方文档](https://cli.vuejs.org/zh/guide/mode-and-env.html)**



### 手动更改开发/生产模式

在之前vue cli2中需要配置环境

```js
const webpack = require('webpack')

plugins: [
    new webpack.DefinePlugin({
        "process.env.NODE_ENV": JSON.stringify('production/development')
    })
]
```

才能在全局中console出`process.env.NODE_ENV`的开发环境。

**现在cli4中无需配置以上代码，即可进行打印**

如果我们想切换当前环境，可以在`package.json`中，更改当前的运行命令

如

```js
{
  "scripts": {
    "serve": "vue-cli-service serve  --mode development",	//默认是serve的mode就是development环境
    "build": "vue-cli-service build"
  }
}
```

#### 其他配置

##### vue-cli-service serve

```
用法：vue-cli-service serve [options] [entry]
示例：vue-cli-service serve  --mode development

选项：

  --open    在服务器启动时打开浏览器
  --copy    在服务器启动时将 URL 复制到剪切版
  --mode    指定环境模式 (默认值：development)
  --host    指定 host (默认值：0.0.0.0)
  --port    指定 port (默认值：8080)
  --https   使用 https (默认值：false)
```

##### vue-cli-service build

```
用法：vue-cli-service build [options] [entry|pattern]

选项：

  --mode        指定环境模式 (默认值：production)
  --dest        指定输出目录 (默认值：dist)
  --modern      面向现代浏览器带自动回退地构建应用
  --target      app | lib | wc | wc-async (默认值：app)
  --name        库或 Web Components 模式下的名字 (默认值：package.json 中的 "name" 字段或入口文件名)
  --no-clean    在构建项目之前不清除目标目录
  --report      生成 report.html 以帮助分析包内容
  --report-json 生成 report.json 以帮助分析包内容
  --watch       监听文件变化
```



### babel中polyfill

#### polyfill的配置

默认在配置babel时，polyfill默认包含以下配置

```js
  presets: [
    '@vue/cli-plugin-babel/preset'   
    //=>  ['es.array.iterator', 'es.promise', 'es.object.assign', 'es.promise.finally']
  ],
```

我们也可以进行自定义其配置

```js
// babel.config.js
module.exports = {
  presets: [
    ['@vue/app', {
      polyfills: [
        'es.promise',
        'es.symbol'
      ]
    }]
  ]
}
```

#### polyfill的useBuiltIns选项

> 在cli4中，useBuiltIns的默认配置就是usage

* `useBuiltIns：‘usage’`     自动检测源码中需要polyfill的地方进行解析

* `useBuiltIns：‘entry’`     自行在需要导入的地方导入
* `useBuiltIns：false`         不使用polyfill

### webpack模块可视化

> 很新颖的一个东西，可以概览项目中用的模块占用内存大小，先上图片

<img src=".\img\cli4\模块视图分析.png" style="zoom: 50%;" />

那么就让我们来看下模块可视化工具（装B神器）的用法

1. 安装webpack-bundle-analyzer

   `npm i webpack-bundle-analyzer -D`

2. 在vue.config.js中配置webpack-bundle-analyzer

   ```js
   const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
   
   //如果你看过我上面的文章，应该知道怎么在vue.config.js中怎么添加插件，好吧直接给出代码
   configureWebpack: {
     plugins: [
       new BundleAnalyzerPlugin()
     ]
   }
   ```

3. 配置package.json

   `"build:analyze": "vue-cli-service build --report",`

4. 运行

   `npm run build:analyze`



### 项目部署

#### 打包文件本地预览

`dist` 目录需要启动一个 HTTP 服务器来访问 (除非你已经将 `publicPath` 配置为了一个相对的值)，所以以 `file://` 协议直接打开 `dist/index.html` 是不会工作的。

在本地预览生产环境构建最简单的方式就是使用一个 Node.js 静态文件服务器，例如 [serve](https://github.com/zeit/serve)：

```bash
npm install -g serve
# -s 参数的意思是将其架设在 Single-Page Application 模式下
# 这个模式会处理即将提到的路由问题
serve -s dist
```

注：如果你非要本地打开，也可以进行如下设置

1. 设置  publicPath:'./'   

2. 路由不能为history 模式



#### 项目打包踩坑

**问题1：打包后放服务器路由无法正常跳转**

解答：

方法1：需要把dist文件夹中的内容放在服务器域名（或本地服务器）的**根目录**下面，才会正常跳转

方法2：在`vue.config.js`中更改`publicPath`的路径

```js
//如果你的应用被部署在 https://www.xxx.com/my-app/，则设置 publicPath 为 /my-app/
module.exports = {
  //publicPath: './',     默认为相对路径，按照方法1即可解决跳转问题
  publicPath: '/my-app/',  
}
```

方法3：如果你想部署在文件夹下又不想每次该publicPath，可以使用三元判断，在开发和生产环境同样有效

```js
  publicPath: process.env.NODE_ENV === 'production'
    ? '/my-app/'
    : './'
```



**问题2：打包后页面刷新404**

解答：这种原因多数情况是因为vue中的router模式设置为history模式，需要进行后端配置，查看[官网配置](https://router.vuejs.org/zh/guide/essentials/history-mode.html)，或者查看我的博客[宝塔服务器配置nginx刷新404的问题汇总](https://www.cnblogs.com/lovecode3000/p/12393168.html)

