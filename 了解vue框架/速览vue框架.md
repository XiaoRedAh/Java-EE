# 速览vue框架

官网：https://cn.vuejs.org/guide/introduction.html

## 创建vue项目

### Vue CLI快速创建前端项目

Vue CLI是Vue官方提供的构建工具，通常称为脚手架。

用于快速搭建一个带有热重载（在代码修改后不必刷新页面即可呈现修改后的效果）及构建生产版本等功能的单页面应用。

Vue CLI基于 webpack 构建，也可以通过项目内的配置文件进行配置

**以上总结为一句：通过脚手架可以帮我们创建前端项目最基本的框架，不用自己从0开始搭建**

**使用**

① 代码所在目录下载cli脚手架

```
npm install -g @vue/cli
```

② 创建vue项目

```
npm create 项目名
``` 

创建时几点注意：

* eslint不要选（语法风格相关，不熟练的话会有各种各样的问题）
* Linter取消掉（就是eslint）
* 选择将依赖存入package.json文件（类似于maven的pom.xml）

## vue项目结构

![](img/vue%20cli%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.png)


* **build**：最终发布的代码的存放位置。
* **config**：配置路径、端口号等一些信息
* **node_modules**：项目所需要的各种依赖模块。
* **src**：开发的主要目录（源码），里面包含了几个目录及文件：
    * **assets**：放置一些图片(会根据图片大小分类进行base64命名还是其他方式命名)，如logo等
    * **components**：目录里放的是一个个的组件文件
    * **router/index.js**：配置路由的地方
    * **App.vue**：项目入口组件（根组件）。也可以将组件写这里，而不使用components目录。主要作用就是将我们自己定义的组件通过它与页面建立联系进行渲染，这里面的<router-view/>必不可少。
    * **main.js** ：整个项目的入口js，引入依赖包、默认页面样式等（项目运行后会在index.html中形成一个app.js文件）。
* **static**：静态资源目录(会原分不动的对文件进行处理)
* **test**：初始测试目录，可删除
* **.XXXX文件**：配置文件。
* **index.html**：html单页面的入口页面，可以添加一些meta信息或者同统计代码啥的或页面的重置样式等。
* **package.json**：项目配置信息文件/所依赖的开发包的版本信息及所依赖的插件信息。（大概版本）
* **package-lock.json**：项目配置信息文件/所依赖的开发包的版本信息及所依赖的插件信息。（具体版本）
* **README.md**：项目的说明文件。
* **webpack.config.js**：webpack的配置文件，例：把.vue的文件打包成浏览器能读懂的文件。
* **.babelrc**:是检测es6语法的配置文件，例：适配哪些浏览器的限制
* **.gitignore**:上传到服务器忽略哪些文件的配置（比如模拟本地数据mock在get提交/打包上线的时候忽略不使用，可在这里配置）


    

安装项目所需要的依赖包/插件（在package.json可查看）：执行 cnpm install   (npm可能会有警告，这里可以用cnpm代替npm了，运行别人的代码需要先安装依赖)如果创建项目的时候没有报错，这一步可以省略。如果报错了  cd到项目里面运行  cnpm install   /  npm install

若拿到别人的项目或从gethub上下载的项目第一步就是要在项目中 cnpm install ;下载项目所依赖的插件，然后 npm run dev 运行项目

## 内置指令

v-if,v-else,v-show,v-for,@click，v-text，v-html...具体用法直接去官网看就行，这里就写一些需要主要的地方就行

官网：https://cn.vuejs.org/guide/introduction.html

### v-if和v-show区别

官网有详解，直接去看就行

使用v-if，组件是真实地再被创建和销毁

而v-show为false时，只是在css层面将组件隐藏

### v-text和v-html区别

v-text：把数据当作纯文本显示

v-html：遇到html标签,会正常解析

## 组件化开发

components.Hello.Vue

```html
<template>
    <h1>hello</h1>
</template>

<script>

</script>

<style>

</style>
```

App.vue

```html
<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <!--使用组件-->
  <Hello></Hello>
  <Hello></Hello>
</template>

<script>
//导入组件
import Hello from './components/Hello.vue'

export default {
  name: 'App',
  //注册组件
  components: {
    Hello
  }
}
</script>

<style>
...
</style>

```



