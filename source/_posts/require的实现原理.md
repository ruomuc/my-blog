---
title: require的实现原理
categories:
  - node.js
tags:
  - node.js
date: 2018-07-03 18:31:16
updated: 2018-07-03 18:31:16
cover: https://proxy.qnoss.seeln.com/images/wp4202323-aconcagua-wallpapers.jpg
---

##### <font color = grat>几乎所有Node.js开发人员都可以告诉你这个require()函数的作用，但是我们中有多少人真的知道它是如何工作的？我们每天都使用它来加载库和模块，但它的行为却是一个谜。</br>好奇，我挖掘了Node核心，了解幕后发生了什么。但是，我没有找到单一功能，而是成为Node模块系统的核心：module.js。该文件包含一个令人惊讶的功能但相对未知的核心模块，它控制所使用的每个文件的加载，编译和缓存。require()事实证明，这只是冰山一角。</font>

## MODULE.JS
```js
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  // ...
```
###### 找到的Module类型在module.jsNode.js 中有两个主要角色。首先，它为所有Node.js模块的构建提供了基础。每个文件在加载时都会获得此基本模块的新实例，即使在文件运行后也会持续存在。这就是为什么我们可以module.exports根据需要附加属性并在以后返回它们。
###### 该模块的第二项重要工作是处理Node的模块加载机制。require我们使用的独立函数实际上是一个抽象module.require，它本身只是一个简单的包装器Module._load。这种加载方法处理每个文件的实际加载，是我们开始旅程的地方。

#### Module._load
```js
Module._load = function(request, parent, isMain) {
  // 1. Check Module._cache for the cached module.
  // 2. Create a new Module instance if cache is empty.
  // 3. Save it to the cache.
  // 4. Call module.load() with your the given filename.
  //    This will call module.compile() after reading the file contents.
  // 5. If there was an error loading/parsing the file,
  //    delete the bad module from the cache
  // 6. return module.exports
};
```

###### Module._load负责加载新模块和管理模块缓存。在加载时缓存每个模块可减少冗余文件读取的数量，并可显着加快应用程序的速度。此外，共享模块实例允许类似单一模块，可以在项目中保持状态。
###### 如果缓存中不存在模块，Module._load则将为该文件创建新的基本模块。然后它会告诉模块在发送新文件之前读入新文件的内容module._compile。
###### 如果您注意到上面的步骤＃6，您将看到module.exports返回给用户。这就是你使用exports和module.exports定义公共接口的原因，因为那正是Module._load然后require会返回的。令我感到惊讶的是，这里没有更多的魔法，但如果有什么更好的话。

#### module._compile

```js
Module.prototype._compile = function(content, filename) {
  // 1. Create the standalone require function that calls module.require.
  // 1.创建调用module.require的独立需求函数。
  // 2. Attach other helper methods to require.
  // 2.将其他辅助方法附加到require。
  // 3. Wraps the JS code in a function that provides our require,
  //    module, etc. variables locally to the module scope.
  // 3.在提供我们需求的函数中包装JS代码，模块等变量本地到模块范围。
  // 4. Run that function
};
```
###### 这是真正的魔法发生的地方。首先，require为该模块创建一个特殊的独立功能。这是我们都熟悉的必需功能。虽然函数本身只是一个包装器Module.require，但它还包含一些鲜为人知的帮助器属性和方法供我们使用：
- require()：加载外部模块
- require.resolve()：将模块名称解析为其绝对路径
- require.main：主要模块
- require.cache：所有缓存的模块
- require.extensions：每种有效文件类型的可用编译方法，基于其扩展名

###### 一旦require准备好，整个装载的源代码被包裹在一个新的功能，这需要在 require，module，exports，和所有其他暴露变量作为自变量。这为该模块创建了一个新的功能范围，因此不会污染Node.js的其余环境。
```js

(function (exports, require, module, __filename, __dirname) {
  // YOUR CODE INJECTED HERE!
});
```
###### 最后，运行包装模块的功能。整个Module._compile方法是同步执行的，因此原始调用Module._load只是在完成并返回module.exports给用户之前等待此代码运行。

####结论
###### 因此，我们已经到达了需求代码路径的末尾，并且通过创建require我们首先开始调查的函数，这样做已经完全循环。

###### 如果你已经完成了这一切，那么你已经准备好了最后的秘密：require('module')。没错，模块系统本身可以通过模块系统加载。盗梦空间。这可能听起来很奇怪，但它允许userland模块与加载系统交互而无需深入Node.js核心。像嘲弄和重新布线这样的流行模块都是基于此构建的。