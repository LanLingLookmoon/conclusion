# webpack

:link:https://zhuanlan.zhihu.com/p/363928061

核心功能：将图片、css、js等进行转译、组合、拼接、生成js格式的bundler文件。

其他功能：

1.模块打包：将不同的文件整合到一起，保证他们的有序引用。

2.编译兼容：将ES6+的新语法转为ES5，使低版本浏览器可以兼容。

3.能力扩展：通过plugin机制，可以实现按需加载，代码压缩等功能。

##### 名词解释

<ul>
    <li>entry：编译入口，webpack编译的起点</li>
    <li>Compiler：编译管理器，webpack启动后创建</li>
    <li>compilation：单次编译过程的管理器，运行过程中只有一个，每次文件变更重新编译时创建新的对象</li>
    <li>dependence：依赖对象，记录模块的依赖关系</li>
    <li>module：所有资源存在的形式，操作资源的单位</li>
    <li>chunk：编译完成准备输出时，按特定的规则组织成的chunk，和输出一一对应</li>
    <li>loader：资源内容转换器</li>
    <li>plugin：构建过程中 ，在特定的时机广播对应的事件，插件监听 这些事件，在特定的事件接入编译过程</li>
</ul>

##### 内容转换+资源合并阶段

###### 1.初始化（dependence）

a.初始化参数

b.创建编译器对象：创建<b>compiler对象</b>（编译管理器，一直存活直到退出，只有一个）

c.初始化编译环境

d.开始编译：执行compiler对象的run方法

e.确定入口：根据配置中的entry找出所有的入口文件，调用compilition.addEntry将入口文件转为<b>dependence对象</b>（依赖关系）

文件转dependce

###### 2.构建阶段（module）

1.编译模块：根据entey对应的dependence创建module对象，调用loader将module转译为标准js内容，用JS解释器将内容转为AST对象。递归本步骤直至所有文件都经过本步骤处理。

2.完成模块编译：获取上一步每个模块被翻译的内容以及依赖关系图。

dependce转module

###### 3.生成阶段（chunks）

1.输出资源：根据依赖关系，将module组装成chunk。可修改输出内容。

2.写入文件系统：不可修改

module转chunks

##### plugin

###### what

带有apply的类。

apply运行时产生参数compiler(监听，文件修改时重新编译)，compiler类调用hook对象，hook调用钩子函数，例如make，最后选择调用方式。

````js
//完整例子
compiler.hook.make.tapAsync
````

<h5>
    compile钩子函数
</h5>
<ul>
    <li>environment</li>
    <li>afterEnvironment</li>
    <li>entryOption</li>
    <li>afterPlugins</li>
    <li>afterResolvers</li>
    <li>...</li>
    <a>https://webpack.docschina.org/api/compiler-hooks/#watching</a>
</ul>



生成配置参数 -----> 生成compiler对象 ---plugin生成配置环境---> compiler.run ---entry找到入口文件 --compilition.addentry--> dependence --------->  module ---loder---> js ------> ast对象 -- 递归所有的ast对象 -----> 依赖关系图，根据 import/require----> chunk --配置的路径和文件名---> 文件列表 ----> 文件  



##### 热更新(局部自动刷新)

###### 原理

基于WDS的模块热替换

###### 配置

```js
plugins: [
new webpack.HotModuleReplacementPlugin()
]
```

```js
devServer: {
    hot: true
}
```

```js
// 入口文件配置
if(module.hot) {
    module.hot.accept()
}
```

###### 更新流程

1.浏览器和webpack通过websocket实现通信

2.修改文件时产生两个文件，一个json文件，一个js文件。json文件记录hash值和修改的模块，js记录修改的代码。

3.浏览器更新时通过hash值获取最新代码，再次运行，覆盖旧代码。

###### 名词解释

webpack-complier： webpack编译器

HMR serve：将热更新的文件输出给HMR Runtime

Bunble Server：提供文件在浏览器的访问，提供localhost访问服务

HMR Runtime：打包阶段注入到浏览器的bundle.js，可以使用websocket与服务器建立连接，当收到指令后，就对文件进行更新

bundle.js：构建输出的文件

###### 执行阶段

启动阶段

webpack-compiler ---------------> bundle server（浏览器可访问）

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a4f10eb6de943b88f7a3313d8aa5795~tplv-k3u1fbpfcp-watermark.awebp)

热更新阶段

webpack-compiler ------> HMR serve ---文件变化--> HMR Runtime(websocket)更新代码

###### 总结

1.HRM Runtime 通过插件注入到 chunk 中

2.开启了bundle server 和 HMR server (与HMR Runtime通信)

3.编译完成时通过compiler.hooks.done通知客户端更新

4.客户端调用module.hot.check发起http请求获取新资源并刷新页面（获取json和js文件）

##### sourceMap

打包后的代码映射回源码的技术，需要浏览器支持。并非webpack独有，jquery也支持。

映射文件： .map

##### Loader编写思路

loader作用：将非js文件转为js格式

配置：

1.支持以数组形式配置多个，用按顺序进行链式调用，前一个loader返回作为下一个loader入参。

2.this由webpack提供，指向loaderContext和loader-runner。

规范：

1.返回值必须是标准js字符串以保证loader可链式调用。

2.遵循单一职责，只关心loader输出及对应输出。

```js
module.exports = {
    module: {
        rules: [
            {
                // 匹配规则
                test: /^your-regExp$/,
                // 预处理器
                use:[
                    {
                        loader: 'loader-name-A'
                    },{
                        loader: 'loader-name-B'
                    }
                ]
            }
        ]
    }
}
```

##### Plugin编写思路

作用：功能扩展。监听运行过程中的事件，在特定阶段执行插件任务，从而实现自己的功能。

compiler暴露webpack整个生命周期的钩子，compilation暴露与模块和依赖有关的事件钩子。

规范：

1.插件必须是一个函数或者是一个包含apply的对象。

2.传给compiler和compilation的对象是同一个引用，若某个插件对他们进行了修改会影响后续插件。

3.异步事件需要在插件处理完后调用回调函数通知webpack进入下个阶段，不然会卡住。



##### Tapable

webpack内部的流程管理工具，主要负责串联插件，完善时间执行流程。·

###### 同步钩子

###### 异步钩子

###### 具体实现

1.实例化

2.用tap等方法回调函数

3.用call调用

4.编译



##### 缺陷

1.构建花时间太长

2.打包体积过大

##### 优化方案

###### 减少不必要的loader使用

1.用include或exclude避免不必要的转译

```js
exclude: /(node_modules|bower_components)/
```

2.开启缓存将转译结果缓存至文件系统

```js
loader: 'babel-loader?cacheDirectory=true'
```

3.用dllplugin处理第三方库

dllplugin处理的第三方库会被单独打包到一个文件中，不会随着业务代码一起被重新打包，只有当依赖关系发生变化时才会重新打包。

4.happypack将loader转为多线程

###### 压缩打包体积

1.利用tree-shaking去掉冗余代码

2.按需加载

核心方法：require.ensure()

3.gzip压缩



# babel

核心功能：将ES6+的新语法转为ES5，使低版本浏览器可以执行。

存在方式：

1.单行命令

2.命令行

3.构建工具插件

##### 处理阶段

###### 解析

可以使用语法插件（能够解析更多语法），内部解析库为babylon

###### 转换

tip：babel不具备转换功能，转换功能再plugin中，当我们不配置插件时，输出内容和输入相同。

添加转译插件，对源码进行转译并输出。

###### 生成

