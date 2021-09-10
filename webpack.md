# webpack

:link:https://zhuanlan.zhihu.com/p/363928061

核心功能：将图片、css、js等进行转译、组合、拼接、生成js格式的bundler文件。

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

###### 2.构建阶段（module）

1.编译模块：根据entey对应的dependence创建module对象，调用loader将module转译为标准js内容，用JS解释器将内容转为AST对象。递归本步骤直至所有文件都经过本步骤处理。

2.完成模块编译：获取上一步每个模块被翻译的内容以及依赖关系图。

###### 3.生成阶段（chunks）

1.输出资源：根据依赖关系，组装成一个个包含多个模块的chunk。可修改输出内容。

2.写入文件系统：不可修改

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



生成配置参数 -----> 生成compiler对象 ---plugin生成配置环境---> compiler.run ---entry找到入口文件--> dependence --------->   依赖关系图 ---------> module ---loder---> js ------> ast对象 -- 递归所有的ast对象，根据 import/require----> chunk --配置的路径和文件名---> 文件列表 ----> 文件  



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

