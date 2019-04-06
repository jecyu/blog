---
author: jecyu
layout: post
title: "企业级前端框架 dojo 学习之旅"
date: 2018-06-14
comments: true
categories: js相关
tags: [dojo]
toc: true
---

刚接触 dojo，首先是去 google 下，这个框架的出现是为了解决哪些问题的？ 有目标，带着问题学习往往更高效。好在有大神指明方向，初步了解什么是 dojo，掌握 dojo 的模块化（AMD），理解 dojo 的面向对象写法。接下就是阅读 → 编码 → 阅读的过程了。官网指南板块

- MODULES
- DOM BASIC
- FUNDAMENTALS
- WIGETS
- DATA
- MOBIE

大概扫描下，包罗万象的感觉。。。

<!-- more -->

## 什么是 dojo？

dojo 是一个基于 JavaScript 和 HTML 的框架，提供了更好的 OOP 模式，组件化思想，表格、图表、离线缓存等，容易扩展自定义的 wiget，适用于比较复杂的 web 应用程序，不过学习曲线比 jQuery 徒。由于之前对 AMD 模块仅限了解，没有用过，而 dojo 是基于 AMD 模块来组织代码的，之后便是要理解它的面向对象用法（感觉跟 java 的面向对象规则比较类似），学会如何自定义组件。

### dojo 的架构

> Dojo 是一个强大的面向对象 JavaScript 框架。主要由三大模块组成：Core、Dijit、DojoX。其中 Core
> 提供 Ajax、events、packaging、CSS-based querying、animations、JSON 等相关操作 API。Dijit 是一个
> 可更换皮肤，基于模板的 WEB UI 控件库。DojoX 包括一些创新/新颖的代码和控件：DateGrid、charts、
> 离线应用、跨浏览器矢量绘图等。

- Dojo
  提供 Ajax、events、packaging、CSS-based querying、animations、JSON 等相关操作 API。Dojo 是 Dojo 的基础，所有其他的功能都是建立在其上，它包含大约 50 个 Javascript 脚本和几个其他资源。这些资源用于处理浏览器差异的统一、Javascript 模块化、Javascript 核心库扩展、W3C DOM API、远程脚本程序、数据存储 API、本地化和国际化，以及一些其他的附加功能；
- Dijit
  一个在 Dojo 之上的小部件系统和组件库，它允许用户重新使用和使用重新编码的 wigets。
- Dojox
  包含一些尚未准备好包含在 Dojo 库中的小部件、实用程序和类。
- Util
  包含用于打包，测试和记录项目的脚本和资源。
- Custom code
  这里放一些自己定义的类、组件。

### dojo 的优点

> Dojo 的特点可从下面几部分说起
> 1、Dojo 是一个纯 Javascript 库，后台只要提供相应的接口就能够将数据以 Json 的格式输出到前台。
> 2、Dojo 自身定义了完整的函数库，屏蔽了浏览器的差异。
> 3、Dojo 自身定义了界面组件库，其组件代码采用了面向对象的思想，便于继承及扩展。
> 4、当对前端界面联动需求较为复杂的时候，基于 dojo 的页面组件将是首选，因为其可以将界面中某一个具有共性的区域抽象出来，封装这一区域的界面行为以及数据，可以用搭积木的方式完成复杂页面的开发。

### dojo 的缺点

- 需要更多的带宽
  使用 webpack 打包处理

## dojo 的模块化

dojo 是基于 (Asynchronous Module Definition)[AMD](https://github.com/amdjs/amdjs-api/wiki/AMD) 模块来组织代码的，它指定了定义模块的机制，实现异步加载模块和依赖。模块就是一个保存文件系统中的 （class.js）文件，通常引入一个模块，需要加上文件名和路径，如 my/module/class。这里的 my 可以跟 java 的包作用差不多，区分不同功能模块的代码文件

> The Asynchronous Module Definition (AMD) API specifies a mechanism for defining modules such that the module and its dependencies can be asynchronously loaded. This is particularly well suited for the browser environment where synchronous loading of modules incurs performance, usability, debugging, and cross-domain access problems.

AMD 是通过 loader（加载器） 来实现模块的创建和加载， loader 是一段 JavaScript 代码，用于处理定义和加载模块的背后逻辑。通常，我们通过引入 dojo.js 或 require.js，就得到一个 AMD 的加载器，它定义了 define 和 require 函数来供我们创建和加载模块。

> define(id?, dependencies?, callback);

    define([
      'dojo/dom'
    ], function (dom) {
      var oldText = {};
      // This returned object becomes the defined value of this module
      return {
        setText: function (id, text) {
          var node = dom.byId(id);
          oldText[id] = node.innerHTML;
          node.innerHTML = text;
        },
        restoreText: function (id) {
          var node = dom.byId(id);
          node.innerHTML = oldText[id];
          delete oldText[id];
        }
      };
    });

第一个参数是模块的标识，通常会忽略这个参数（避免污染全局变量，使用文件名和路径来代替使用），第二个参数是模块依赖，第三个参数是回调函数。loader 在确认依赖加载完成后，会将模块作为参数列表传入回调函数并执行它。注意的是，引入模块的顺序与传入回调函数的参数顺序要对应一致。

> require(dependies, callback);

    require([
          'demo/myModule',
          'dojo/fx',
          'dojo/domReady!'  // 这个模块指定代码等待 dom 加载完成，再执行
        ], function (myModule, fx) {
          myModule.setText('greeting', 'Hello Dojo!');

          setTimeout(function () {
            myModule.restoreText('greeting');

            // with an animation
            fx.slideTo({
              node: greeting,
              top: 100,
              left: 200
            });
          }, 3000);
        });

需要注意的是，这里既使用到了 CDN 引入的 dojo.js，又使用到本地自定义的模块。因此需要在引入 dojo.js 的 script
标签前，写入 dojoConfig 对象 loader 对代码文件依赖的模块作指定的处理，这里可以设置引用不同 dojo
版本、包的引入路径等。

     var dojoConfig = {
          async: true, // 异步引入依赖
          // This code registers the correct location of the "demo"
          // package so we can load Dojo from the CDN while still
          // being able to load local modules
          packages: [{
            name: "demo",
            location: location.pathname.replace(/\/[^/]*$/, '')
          }]
        };

## dojo 是如何处理 dom 的？

### DOM 节点

从处理 DOM 的增删改除入手，dojo 通过引入 dojo/dom 模块，提供 `dom.byId()` 获取单一节点，返回值为当前节点。通过 `dojo/query` 模块里的 `query` 方法类似 CSS3 选择器获取多个节点，返回值为一个匹配的数组节点。

- Retrieval（获取）
  dojo/dom 模块的 byId(idName)方法，dojo/query 模块的 query(id/class/...)
- Creation（创建）
  dojo/dom-construct 模块的 create(node, {propeties} ,parent or sibling node, position) 方法，第一个参数为需要创建节点的字符串名字，第二个为参考的节点（用于摆位置，可省），第三个参数为该节点的属性哈希集合。第四个参数为相对于父亲或兄弟节点的位置（可省，默认为 “last”）。
- Placement（摆位）
  dojo/dom-construct 模块的 place(currentNode, referenceNode, position) 方法（上面提到 dom 的 create 背后使用了这个方法来摆位）
- Destruction（删除）
- dojo/dom-construct 模块里的 domConstruct.destroy() 方法删除单个节点，domConstruct.empty() 删除所有节点

### DOM 事件

说完 dom 节点的处理后，之后便是与用户交互关键的事件，dojo 提供了 dojo/on 模块，通过 on(element,event name,handler) 方法来实现事件的注册，返回值一个包含了 remove() 方法的 handler 对象，用于移除挂载在该元素上的注册事件。通常该事件处理程序在在第一个传递的参数的上下文运行的环境，事件委托除外(on(parent element, "selector:event name",handler)上下文参考第二个参数） 无论是普通事件还是事件委托都可以通过 `dojo/_base/lang` 模块的 hitch() 方法指定事件处理程序运行上下文环境。

当我们需要处理当前还不存在的节点时（如 todo 应用），可以使用 dojo 的事件发布/订阅机制，通过引入模块 dojo/topic，使用 topic.subcribe(topicName,callback(args)) 订阅一个主题，通过 topic.publich(topicName, args) 在需要的地方发布调用。

> dojo.publish and dojo.subscribe implement the idea of a central hub, which can **receive messages** on given topics (sent by calls to dojo.publish)

- widget 之间可以通过 topic 进行通信，实现 widget 的关注点分离，需要注意的重要一点是，任何发布者或订阅者都不一定知道有谁在应用程序中发布或订阅同一主题，因此范例中没有紧密耦合或上下文敏感性。
- 挂载 subscribe 的最佳位置通常是页面的顶级，在父 Widget 进行订阅处理，在子 widget 进行触发发布。特殊情况下，顺序会反过来，取决于你需要在哪里处理数据，就在哪里 subscribe。(如果是同一个 widget，除了函数传参外，也可以把数据存储到 widget 对象中集中处理, this.data)
- 如果是 widget 内部函数共享数据时，可以存储为 this.xxx 共享

![image_1cm7k5nb3aoj1hiuglb17gd12fd9.png-46.7kB](http://static.zybuluo.com/linjiyu/3vaqen42a6bcs82j5e2us1ij/image_1cm7k5nb3aoj1hiuglb17gd12fd9.png)

> **注意：如果是需要手动清除事件，并且使用了 this.own 的时候，它返回的是一个数组，第一个元素为当前绑定的元素。**

![image_1cnbqp50ne8p15m7gpm58s4nkc.png-51.9kB](http://static.zybuluo.com/linjiyu/g6b9yfdm5g7deanadqp155al/image_1cnbqp50ne8p15m7gpm58s4nkc.png)

## dojo 是怎样实现面向对象的？

面向对象是相对于面向过程的，面向对象则是给你一个任务，你要先做一个什么东西来做这件事，再做一个什么东西来做那件事，它们之间是怎样相互配合的。对象是类的实例，而类之间可以相互继承。dojo 通过
`dojo/_base/declare` 模块来实现这个过程， 提供方法 declare(className, superClass, properties)，为了避免污染全局命名空间和减少命名冲突，第一个参数通常省略，使用 AMD 模块的文件名来代替。

第二个参数则是继承的父类，这里可以是继承单个类，也可以继承多个类，继承多个类的时候，继承多个类的时候。只有通过多重继承的第一个类才是真正的超类。剩下的就是 mixin，并且混合到子类中以产生我们需要的继承链，子类可以通过继承链遍历要使用的方法。对于父类之间含有同名的方法和属性的情况，子类会继承传入参数中最后一个类的。

第三个参数则是定义当前类的属性和方法，以及构造器 construtor，在构造函数可以通过传入参数覆盖默认设置的属性和方法。

    define(['dojo/_base/declare', 'dojo/_base/lang'], function (declare, lang) {
      return declare(null, {
        name: 'Anonymous',
        age: null,
        residence: 'Universe A',

        constructor: function (/*Object*/ kwArgs) {
          lang.mixin(this, kwArgs);
        },

        moveTo: function (/*String*/ residence) {
          this.residence = residence;
        }
      });
    });

**值得注意的是，如果在类的属性对象中定义了引用类型如 array、object 的话，所有的类实例对象都可以更改类的属性**，故此最好先定义默认的空对象或空数组，再在构造函数中根据实例对象自定义。

## 如何编写自定义 widget

自定义 widget 则是 Dojo 自定义对象的扩展，反映了 Dojo 是基于 HTMl 的 JavaScript 框架。Dojo 工具包附带了 Dijit 框架，它是一组 widgets，通过这些 widgets 可以方便地构建用户图形界面。最重要的是，它还提供 **`dijit/_WidgetBase`** 和 **`dijit/_TemplatedMixin`** 这两个模块，可以很方便地编写自己的 widget。其中 \_WidgetBase 模块是 Dijit 的基础模块，\_TemplatedMixin 对 \_WidgetBase 的扩展。

### dijit/\_WidgetBase

Dijit 的基础，以及创建自己的 widget，主要依赖于 dijit / \_WidgetBase 模块中定义的一个基类，而理解 dijit 系统最重要的是一个 widget 的生命周期。 在一个基于 \_WidgetBase 声明的 widget 实例化过程包括：

- 根据默认值和运行值初始化 widget 的数据
- 生成 Widget 的 DOM 结构
- 把 Widget 的 DOM 结构添加到文档中
- 处理依赖于文档中存在的 DOM 结构的背后逻辑

**widget 的生命周期**

    // 被调用时机：在实例化 widget 传入参数之前调用
    // 用处：初始化数组、对象，覆盖 widget 的默认值
    **constructor**: function (/*Object*/) {
      console.log('1-constructor');
    },

    // 被调用时机：在 DOM 渲染之前，constructor 实例化参数后
    // 用处：用于 widget 实例被创建之前，更改实例的属性
    postMixInProperties: function () {
      console.log('2-postMixInProperties');
    },

    // 被调用时机：在 postMixInProperties 之后
    // 用处：创建节点，挂载事件，最终结果赋值给 this.domNode
    buildRendering: function () {
      // 创建 widget template 片段
      console.log('3-buildRendering');
    },

    // 被调用时机：在已经初始化所有属性后 和 template 添加到 DOM 前，这时实例的 widget已经创建，但是它的 containNode 里的 子 widget 还没有(可以在订阅事件，然后在子 widget 发布使用)
    // 用处：自定义 domNode 的样式和行为

    postCreate: function () {
      var domNode = this.domNode;
      // 如果 widget 被 destroy 销毁后，取消相关事件的绑定
      this.own(/*这里是注册的事件*/);
      // 添加 enter 监听跳页
      this.own(
        on(
          this.paginationInput,
          "keydown",
          lang.hitch(this, function(e) {   // hitch(scope,method)
            var event = window.event || e;
            if (event.keyCode === 13) {
              this.jumpToPage();
            }
          })
        )
      );
      console.log('4-postCreate');
    },


    // 被调用时机：dom 片段已经被添加文档中，所有的子 widget 已经被解析和创建完成，注意：如果使用命令式写法，则需要手动调用（在建立了一个父组件后，还需要添加若干子组件（这里的子组件不是表示继承关系，而是表示包含关系），并希望全部加载后一起展现时，可以不手动调用 startup）一个关于该函数的最佳实践就是即使没有子组件，也对一个组件新建实例调用该函数）



    // 用处：widget 布局
    startup: function () {
      this.inherited(arguments); // 调用父类的方法
      console.log('5-startup');
    },

    // 被调用时机：对 widget  使用了destroy() 方法
    // 用处：销毁 widget
    destroy: function () {
      console.log('6-destroy');
    }

    // 手动销毁 this.destroyRecursive() 常用于关闭窗口
    // 这时就不要显式声明 destroy 钩子，否则手动销毁函数不执行
    _closeDocWindow: function() {
      this.destroyRecursive();
    }

![image_1cll06hac12jne301pe51rkl1ht9p.png-21.7kB](http://static.zybuluo.com/linjiyu/sz3xxxrjir46qicup7h4f8up/image_1cll06hac12jne301pe51rkl1ht9p.png)

### dijit/\_TemplatedMixin(使用绑定事件 dojo-attach-point 时，必须提前在 widget 声明)

\_TemplatedMixin 重写了上面提到的 buildRendering 和 destroyRendering 方法，提供了 templateString 属性，让我们可以直接使用一段 HTML 片段作为 widget 的模板。注意的是，该 HTML 片段只能有一个根节点。\_TemplatedMixin 是一个强大的模版系统，使得我们可以方便地在模版中挂载事件和属性。

**1. variable substitution 替换变量，把 widget 定义的属性在实例时传递到 HTML 片段中**

        ${property}，${propertyObject.property}，${!property}

这里的变量名称对应着在 widget 中声明的属性，其中 `!property`主要是避免当插入的 字符串包含反斜杠时被解析时被进行转义。注意的是，这种 ${..} 方法适合于这个变量值在整个 widget 生命周期均不会改变的情况。如果会被改变，则使用自定义的方法 `_setFooAttr()` 来映射 DOM 的属性与 widget 的属性。

**2. attach-point 绑定节点**

通过 HTML5 的 data 属性语法来实现，直接在 DOM 节点中，添加 `data-dojo-attach-point= "nodeName"`，这个连接点告诉将这个属性的值设置为 widget 的属性，这样就可以在 widget 的声明中根据名称引用指定 DOM 的节点。通常，模版的根节点被自动设置为 widget 的 domNode 属性，不需要显式指定。另外，在模版中 `data-dojo-attach-point="containerNode"` 的节点，作为 widget 实例的根节点的内容的容器。

**3. event attachments**

除了绑定节点外，通过 `data-dojo-attach-event="{event1: handler1, event2: handler2,... }"` 直接把 widget 声明的方法挂载到改节点上。

### 编写自定义 widget

在 dijit/\_WidgetBase 的生命周期方法 和 dijit/\_TemplatedMixin 的模版解析帮助下，可以很方便地实现自己的 widget 了。通常实现一个 widget 有如下步骤：

1. 构建 widget 的文件结构

- dijit
- dojo
- dojox
- util
- myApp(or organzationName/ personName)
  - data
  - widget
    - css
    - images
    - templates
    - xxx.js
    - xxxWidget.js

2. 基于你的 widget 创建 HTMl 片段
3. 把 HTML 片段转换为基于 dijit 的模版(注意：该片段 dojo-attach-event 里的事件，需要提前声明)
   这里可以使用 data-dojo-attach 来挂载事件和属性，以及使用 ${...} 语法来传递 widget 属性值。把该模版单独抽离出来后，需要使用 `dojo/text!./templates/xxxWidget.html` 引进来
4. 使用 declare 创建我们 widget 类
   这里就是妥妥的面向对象写法了，定义 widget 的属性以及方法，习惯上使用下划线 `_` 作为私有变量或私有方法的命名前缀。
5. 最后，使用 CSS 修饰美化你的 widget 界面。

### 声明式写法 vs 命令式写法

使用 dojo 有两种编码风格，一种是声明式写法，另外一种是命令式写法。命令式写法是只使用 javaScript 来实例化对象， 所有的 widget 行为都用 javaScript 来表示。声明式是指使用 `dojo/parser` 来读取 DOM，并且解析出已使用特殊属性`data-dojo-\*` 声明的节点以及某些 `<script>` 标签来扩展 widget 的行为。

**1. 引入类**

无论是声明式还是命令式，首先需要引入对应的类

    dojo.require('dijit.form.FilteringSelect')

**2. 声明式（declarative）**

使用声明式写法创建 widget，意味着使用具有特殊属性的 HTML 标记来创建 widget，首先需要指导 Dojo 去解析页面，有两种方式。一种是配置 dojoConfig 的变量 parserOnLoad 的值为 true，这样 parser 在所有资源加载后，会解析整个页面。

     var dojoConfig = {
      async: true,
      parseOnLoad: true,
    };

另外一种是，调用 parser.parse() 可以传入指定节点开始解析。（这里把 parseOnLoad 设置为 false）

      // 加载 parser
      require(['dojo/parser']);

      // 解析页面
      ready(function () {
        parser.parse();
      });

      // 实例 widget
      <span data-dojo-type="FancyCounter">press me</span>

      声明式写法基于 HTML， 直接在 HTML 属性上写入 `data-dojo-type="xxx/xxx/xxxwidget"`，这种写法需要在 require 中引入 `dojo/parser`，使用 `parser.parse()` 进行 DOM 解析。任何“类”（或对象，如由dojo.declare创建的对象）都可以通过在DOM中的某个节点上使用data-dojo-type属性来实例化，并从中创建一个 widget，并且使用 `data-dojo-props` 把属性对象传递给 widget 的构造函数。实例化的 widget，可以通过 `dijit/registry` 模块的方法通过 widgetId 或 挂载的 DOM 节点来引用该 widget。

    require(["dijit/registry"], function(registry){
        var widget = registry.byId("myWidgetId");
    });

更多细节： 官网链接：[Using Declarative Syntax](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/index.html)

**3. 命令式（programmatic）**

使用命令式写法，可以首先创建充当将来 widget 占位符的 DOM 节点，与模版文件 domNode 进行连接，使用这个 id 连接，只是为了让模版文件应用到你指定的 dom 节点。

    widget 声明：
    constructor: function (propertiesObject, elementId) {
    	lang.mixin(this, propertiesObject, elementId);
    },
    ...

    widget 引用：
    <button id="makeMeWidget">Click Me</button>

    ...
    ready(function() {
      // 让元素转为 widgets
      var widget = new dijit.form.Button({}, 'makeMeWidget'); // options, elementID
      widget.startup(); // 必须手动调用这个函数，才能实例化成功。声明式语法 parse 帮你调用了。
    });

也可以不需要指定节点，直接使用模版文件来生成 widget。

**4. 权衡利弊**

- 命令式写法性能更好，不需要像声明式写法检索需要成为 widget 的 DOM 节点。
- 声明式写法可以允许你更快地编码，但是当你的程序代码越来越多的时候，很难维护。

综上，更建议使用命令式写法。

### 一些注意点

1. require([依赖包 1，依赖包], function(依赖包 1，依赖包 2,...) ) ，这里的顺序要对应一致，否则找不到相关的包，引用方法会导致出错。
2. 注意引入相关资源的路径问题。
3. 有关 `ready` 的模块

- dojo/domReady!
  延迟 require() 的执行，直到 DOM 加载完成。
- dojo/ready
  对于 widget 来说，仅仅使用 dojo/domReady! 是不够的，因为对大多数 widget 来说，在这些模块（如：dojo/uacss，dijit/hccss，dojo/parser）执行前不应该初始化和访问 widget。
  所以需要 dojo/ready 来确认 dom 加载完成，以及所有的依赖模块已经被解析。
  require(["xxx", ..., "dojo/ready"], function(Button, ready){
  ready(function(){
  // deal with widgets here
  parser.parse();
  });
  });

- data-dojo-config="parseOnLoad: true / false"
  设置为 true，用于告诉 dojo 应用 dojo/parser 在页面加载后解析整个页面 body 中从头开始扫描带有 `data-dojo-type`的节点，通常这样比较浪费资源。设置为 false，然后传入指定的开始节点给 `parser.parse()` 方法，缩小解析范围。

## dojo 请求后端数据

**dojo/request** 是一个以跨平台形式提供异步请求的包，内容如下：

- dojo/request
  - dojo/request/xhr
  * 提供跨浏览器兼容的 XmlHttpRequest，通过要求 dojo / request 模块，它将返回当前平台的默认提供程序（也就是 xhr 模块）
  - dojo/request/script
    - 通过 jsonp 来实现数据的请求
    - 用于跨域资源请求的场景
  - dojo/request/node
  - dojo/request/iframe

注意：以上模块的方法请求默认返回 promise 对象

### dojo promise

首先要了解 promise 的概念， promise 目的是为了把未来的值与计算它的方式（处理函数）解耦（不用无限嵌套回调函数）。

promise 是异步操作结果的占位符，处理函数可以返回一个 promise，而不用订阅一个事件或向函数传递回调参数。通过 promise 的生命周期可以进行异步读取操作未来值。

所有的 promise 都包含 then() 方法并接受两个参数。第一个参数是 promise 为 fulfilled 状态下调用的函数，任何于异步操作有关的额外数据都会传给该它。第二个参数是 promise 为 rejected 状态下调用的函数，它会被传入任何与操作未完成有关的数据。

> dojo/promise 是管理 promises 的包，换句话说，是异步线程之间的通信。

dojo/promise 组成部分如下：

- dojo/promise/Promise
- dojo/promise/all
- dojo/promise/first

**dojo/Deferred**

dojo / Deferred 是一个类，用作在 Dojo 中管理异步线程的基础
像 dojo / request 这样的包使用此类来返回在异步线程完成时得到解析的 promise。

    var deferred = new Deferred()
    script.get(requestUrl, { jsonp: "callback" }).then(
      function(response) {
        if (response.status == "success") {
          deferred.resolve(response.data);
        } else {
          var msg = ""未成功从服务--" + serviceName + "--获取数据";
          deferred.reject(msg);
        }
      },
      function(error) {
        var msg = "";
        deferred.reject(msg);
      }
    );
    return deferred.promise; // 返回一个

### 非跨域（Ajax with dojo/request）

    require(["dojo/request"], function(request){
      request("request.html").then(function(data){
        // do something with handled data
      }, function(err){
        // handle an error condition
      }, function(evt){
        // handle a progress event
      });
    });

### 跨域（JSONP with dojo/request）

dojo/request/script

> JSONP works by dynamically inserting a `<script>` element onto the page with its source set to the cross-domain URL we want to query.

请求：endpoint?q=dojo&callback=callme
响应：callme({id: "dojo", some: "parameter"})
通过预先定义的回调函数 callme 来处理
优点：将跨域资源与 JSONP 一起使用还可以减少与应用程序的 Web 服务器的连接争用。

### 如何在线渲染图片、office 文档、纯文本

**图片**

关键方法：
URL.createObjectURL() 创建 URL f 对象
URL.revokeObjectURL() 销毁 URL 对象

     _loadImg: function() {
      var that = this;
      window.URL = window.URL || window.webkitURL;
      var img = document.createElement("img");

      img.onload = function() {
        that.imgNativeWidth = this.width;
        that.imgNativeHeight = this.height;
        // 图片不超过Viewer的话居中
        // 判断图片高宽是否超过this.imgViewer，不超过图片居中
        if (
          that.imgNativeWidth < that.imgViewer.offsetWidth &&
          that.imgNativeHeight < that.imgViewer.offsetHeight
        ) {
          domClass.add(this, "middle");
        }
        window.URL.revokeObjectURL(img.src); //
      };

      img.className = "imgNode";
      img.src = window.URL.createObjectURL(this._blob); //
      this.imgViewer.appendChild(img);
      this._img = img;
    },

**纯文本**

> The FileReader object lets web applications asynchronously read the contents of files (or raw data buffers) stored on the user's computer, using File or Blob objects to specify the file or data to read.

## 小结

这几天主要学习 dojo 的代码模块化、DOM 的增删改除和事件、面向对象的实现，之前只是了解 AMD，没有真正用过，现在算是真正理解了它的模块组织方式，加深了面向对象编程的理解。接下来，打算学习如何自定 widget，做个 todomvc 来巩固下。

---

2018-06-08 更新：
今周主要学习如何自定义实现 widget，通过官网的例子搞清楚 'dijit/\_WidgetBase', 'dijit/\_TemplatedMixin' 的用法，然后实践写一些简单的例子，扩展相关的知识点如 dijit/parser 等，最后实现一个 弹框的 widget， 加深了对模块化开发的理解。

[dialog widget 代码](https://github.com/Jecyu/framework-library-learning/blob/master/Dojo/assets/linjy/widget/DialogWidget.js)

2018-08-10 更新

实战总结：

- 利用 Dojo 包系统通过构建脚本来促进模块加载和优化。
- 保持模块化 - 避免将应用程序的特定知识编码到参与其中的组件中。
- 保留关注点 - UI 应该不知道数据的来源，反之亦然

> 期待你的建议，一起交流探讨，共同学习

（完）

## 参考资料

- [dojo 官网](http://dojotoolkit.org/documentation/#tutorials)
- [Debugging Dojo: Common Error Messages](https://www.sitepen.com/blog/2012/10/31/debugging-dojo-common-error-messages/)
- http://dojotoolkit.org/documentation/tutorials/1.6/recipes/app_controller/
- [Futures and Promises Wikipedia Article](https://en.wikipedia.org/wiki/Futures_and_promises)
