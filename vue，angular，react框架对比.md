首先，我们先了解什么是MVX框架模式？

MVX框架模式：MVC+MVP+MVVM
1.MVC：Model(模型)+View(视图)+controller(控制器)，主要是基于分层的目的，让彼此的职责分开。
View通过Controller来和Model联系，Controller是View和Model的协调者，View和Model不直接联系，基本联系都是单向的。
用户User通过控制器Controller来操作模板Model从而达到视图View的变化。

2.MVP：是从MVC模式演变而来的，都是通过Controller/Presenter负责逻辑的处理+Model提供数据+View负责显示。
在MVP中，Presenter完全把View和Model进行了分离，主要的程序逻辑在Presenter里实现。
并且，Presenter和View是没有直接关联的，是通过定义好的接口进行交互，从而使得在变更View的时候可以保持Presenter不变。
MVP模式的框架：Riot,js。

3.MVVM：MVVM是把MVC里的Controller和MVP里的Presenter改成了ViewModel。Model+View+ViewModel。
View的变化会自动更新到ViewModel,ViewModel的变化也会自动同步到View上显示。
这种自动同步是因为ViewModel中的属性实现了Observer，当属性变更时都能触发对应的操作。
MVVM模式的框架有：AngularJS+Vue.js和Knockout+Ember.js后两种知名度较低以及是早起的框架模式。

Vue.js是什么？
看到了上面的框架模式介绍，我们可以知道它是属于MVVM模式的框架。那它有哪些特性呢？
其实Vue.js不是一个框架，因为它只聚焦视图层，是一个构建数据驱动的Web界面的库。
Vue.js通过简单的API（应用程序编程接口）提供高效的数据绑定和灵活的组件系统。
Vue.js的特性如下：
1.轻量级的框架
2.双向数据绑定
3.指令
4.插件化

Vue.js与其他框架的区别？
1.与AngularJS的区别
相同点：
都支持指令：内置指令和自定义指令。
都支持过滤器：内置过滤器和自定义过滤器。
都支持双向数据绑定。
都不支持低端浏览器。
不同点：
1.AngularJS的学习成本高，比如增加了Dependency Injection特性，而Vue.js本身提供的API都比较简单、直观。
2.在性能上，AngularJS依赖对数据做脏检查，所以Watcher越多越慢。
Vue.js使用基于依赖追踪的观察并且使用异步队列更新。所有的数据都是独立触发的。
对于庞大的应用来说，这个优化差异还是比较明显的。
2.与React的区别
相同点：
React采用特殊的JSX语法，Vue.js在组件开发中也推崇编写.vue特殊文件格式，对文件内容都有一些约定，两者都需要编译后使用。
中心思想相同：一切都是组件，组件实例之间可以嵌套。
都提供合理的钩子函数，可以让开发者定制化地去处理需求。
都不内置列数AJAX，Route等功能到核心包，而是以插件的方式加载。
在组件开发中都支持mixins的特性。
不同点：
React依赖Virtual DOM,而Vue.js使用的是DOM模板。React采用的Virtual DOM会对渲染出来的结果做脏检查。
Vue.js在模板中提供了指令，过滤器等，可以非常方便，快捷地操作DOM。
如何使用Vue.js?
1.安装
（1）script
如果项目直接通过script加载CDN文件，代码示例如下：
<script src="http://webapp.didistatic.com/static/webapp/shield/z/vue/vue/1.0.24/vue.min.js"></script>
(2)npm
如果项目给予npm管理依赖，则可以使用npm来安装Vue,执行如下命令：
$npm i vue --save-dev
(3)bower
如果项目基于bower管理依赖，则可以使用bower来安装Vue，执行如下命令：
$bower i vue --save-dev

学习web前端开发中，会有很多的框架，那么目前流行的框架有哪些呢？
一、Vue.js：
     其实Vue.js不是一个框架，因为它只聚焦视图层，是一个构建数据驱动的Web界面的库。
     Vue.js通过简单的API（应用程序编程接口）提供高效的数据绑定和灵活的组件系统。
     Vue.js的特性如下：
　　    1.轻量级的框架
　　    2.双向数据绑定
　　    3.指令
　　    4.插件化
   优点： 1. 简单：官方文档很清晰，比 Angular 简单易学。
　　　　  2. 快速：异步批处理方式更新 DOM。
　　　　  3. 组合：用解耦的、可复用的组件组合你的应用程序。
　　　　  4. 紧凑：~18kb min+gzip，且无依赖。
　　　　  5. 强大：表达式 & 无需声明依赖的可推导属性 (computed properties)。
　　　　  6. 对模块友好：可以通过 NPM、Bower 或 Duo 安装，不强迫你所有的代码都遵循 Angular 的各种规定，使用场景更加灵活。
　 缺点： 1. 新生儿：Vue.js是一个新的项目，没有angular那么成熟。
　　　　　2. 影响度不是很大：google了一下，有关于Vue.js多样性或者说丰富性少于其他一些有名的库。
　　　　　3. 不支持IE8：
二、angularJS：
　　angularJS是一款优秀的前端JS框架，已经被用于Google的多款产品当中。
　　angularJS的特性如下：
　　　　1.良好的应用程序结构
　　　　2.双向数据绑定
　　　　3.指令
　　　　4.HTML模板
　　　　5.可嵌入、注入和测试
　　优点： 1.  模板功能强大丰富，自带了极其丰富的angular指令。
　　　　　 2. 是一个比较完善的前端框架，包含服务，模板，数据双向绑定，模块化，路由，过滤器，依赖注入等所有功能；
　　　　   3. 自定义指令，自定义指令后可以在项目中多次使用。
　　　　　 4. ng模块化比较大胆的引入了Java的一些东西（依赖注入），能够很容易的写出可复用的代码，对于敏捷开发的团队来说非常有帮助。
　　　　　 5. angularjs是互联网巨人谷歌开发，这也意味着他有一个坚实的基础和社区支持。
　　缺点： 1. angular 入门很容易 但深入后概念很多, 学习中较难理解.
　　       2. 文档例子非常少, 官方的文档基本只写了api, 一个例子都没有, 很多时候具体怎么用都是google来的, 或直接问misko,angular的作者.
　　　　　 3. 对IE6/7 兼容不算特别好, 就是可以用jQuery自己手写代码解决一些.
　　　　   4. 指令的应用的最佳实践教程少, angular其实很灵活, 如果不看一些作者的使用原则,很容易写出 四不像的代码, 例如js中还是像jQuery的思想有很多dom操作.
　　　　   5. DI 依赖注入 如果代码压缩需要显示声明.
三、React：
　 React主要用于构建UI。你可以在React里传递多种类型的参数，如声明代码，帮助你渲染出UI、也可以是静态的HTML DOM元素、也可以传递动态变量、甚至是可交互的应用组件。
　  React特性如下：　
　　　　1.声明式设计：React采用声明范式，可以轻松描述应用。
　　　　2.高效：React通过对DOM的模拟，最大限度地减少与DOM的交互。
　　　　3.灵活：React可以与已知的库或框架很好地配合。
　　优点： 1. 速度快：在UI渲染过程中，React通过在虚拟DOM中的微操作来实现对实际DOM的局部更新。
　　　　　 2. 跨浏览器兼容：虚拟DOM帮助我们解决了跨浏览器问题，它为我们提供了标准化的API，甚至在IE8中都是没问题的。
　　　　　 3. 模块化：为你程序编写独立的模块化UI组件，这样当某个或某些组件出现问题是，可以方便地进行隔离。
　　　　 　4. 单向数据流：Flux是一个用于在JavaScript应用中创建单向数据层的架构，它随着React视图库的开发而被Facebook概念化。
　　　　　 5. 同构、纯粹的javascript：因为搜索引擎的爬虫程序依赖的是服务端响应而不是JavaScript的执行，预渲染你的应用有助于搜索引擎优化。
　　　　　 6. 兼容性好：比如使用RequireJS来加载和打包，而Browserify和Webpack适用于构建大型应用。它们使得那些艰难的任务不再让人望而生畏。
　　缺点： 1. React本身只是一个V而已，并不是一个完整的框架，所以如果是大型项目想要一套完整的框架的话，基本都需要加上ReactRouter和Flux才能写大型应用。
