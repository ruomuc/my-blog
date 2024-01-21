---
title: nodejs调用c++
categories:
  - node.js
tags:
  - node.js
  - c++
date: 2024-01-21 15:30:16
updated: 2024-01-21 15:30:16
---

使用`nodejs`调用`c++`的方法很简单，网上也有很多教程，但是为什么公司一般不会大规模去使用`c++`扩展呢。

## nodejs使用c++扩展的优缺点

优点：

- `c++`作为编译型语言，是执行速度最快的编程语言之一，使用`c++`扩展可以提高代码的运行效率
- `c++`拥有丰富的开源库，如果有些功能在`nodejs`中找不到合适的库，可以使用`c++`的库

缺点：

- 开发难度高、调试麻烦、维护成本高，并且跨平台时要编译多个版本
- `Napi`将`nodejs`数据结构和`c++`数据结构相互转换是比较消耗性能的，所以使用`c++`扩展通常会比直接使用`nodejs`还要慢

当然，还是有一些程序适合使用`c++`扩展来写的，比如一些**输入输出极其简单**，但是**计算过程却很复杂**的程序。

比如nodjes的核心库 [crypto](https://nodejs.org/api/crypto.html)，通过查看源码可以发现如下代码片段：

```javascript
// node/lib/crypto.js
const {
  getFipsCrypto,
  setFipsCrypto,
  timingSafeEqual,
} = internalBinding('crypto');

// lib/internal/crypto/hash.js
const {
  Hash: _Hash,
  HashJob,
  Hmac: _Hmac,
  kCryptoJobAsync,
} = internalBinding('crypto');
```

相关文件都有这个 `internalBinding('crypto')`，这个就是说明是使用`c++`扩展实现的相关功能

实际上`nodejs`是使用了`c++`的`OpenSSL`库



## 如何实现一个简单的c++扩展

#### 环境安装

1. 首先需安装 [node-gyp](https://github.com/nodejs/node-gyp)

   `npm install -g node-gyp`，	请注意，你电脑需要有python环境，看源代码仓库就知道，它主要是python写的

2. 需要配置你机器系统对应的c++编译环境

3. 安装nodejs库 `node-addon-api` 和 `bindings`。(ps: 当然还有`nan`、原生的`Node-API`等，这里用的是最简单的`node-addon-api`)



#### 编译一个hello world

##### 1. 编写binding.gyp

具体的参数比较复杂，有专门的文档，这里知道 target_name 和 sources 就行了。

- target_name：是你编译后文件的文件，以`.node`结尾，在这里编译后的文件就为`hello.node`
- sources: 是原始c++代码文件存放的地方

```json
// binding.gyp
{
    "targets": [
        {
            "target_name": "hello",
            "cflags!": ["-fno-exceptions"],  # -fno-exceptions 忽略掉编译过程中的一些报错
            "cflags_cc!": ["-fno-exceptions"],
            "sources": ["addon/hello.cc"],
            "include_dirs": [  # 头文件搜索路径，这样通过#include <napi.h>就可以引入napi
                "<!@(node -p \"require('node-addon-api').include\")"
            ],
            "defines": ["NAPI_DISABLE_CPP_EXCEPTIONS"],
        }
    ]
}

```

##### 2.编写c++代码

```c++
// hello.cc
#include <napi.h>

namespace __node_addon_api_hello__
{

  // Hello方法的实现，该方法返回hello-world
  Napi::String Hello(const Napi::CallbackInfo &info)
  {
    Napi::Env env = info.Env();
    return Napi::String::New(env, "hello-world"); // 返回hello-world
  }

  Napi::Object Init(Napi::Env env, Napi::Object exports)
  {

    // 在exports对象挂上hello属性，其值为Hello函数，也可以用以下的Nan::Export宏实现
    exports.Set(Napi::String::New(env, "hello"),
                Napi::Function::New(env, Hello));

    return exports;
  }

  NODE_API_MODULE(addon, Init)

}
```

##### 3.编写测试文件

```js
// test-hello.js

const hello = require('bindings')('hello')
console.log(hello.hello())
```

注意这里的 `.hello()`的函数名，对应`c++`文件中 `Napi::String::New(env, "hello")`

##### 4. 执行

```shell
node-gyp rebuild
node test.js
```

执行 `node test-hello.js` 会输出 `hello-world`

至此，一个简单的`nodejs`调用`c++`的demo就完成了，代码我会放到文章的最后面

#### 使用buffer共享内存

前面说过，nodejs和c++的数据类型相互转换其实是很耗费时间的，那有没有其他方案呢？

答案确实有：

- 一是我偶然想到的，因为工作中经常使用`nodejs`操作大批量数据，经常不是容器 OOM 了，就是 `nodejs` OOM了。所以有时候会把大量数据以某种格式直接存到磁盘文件上，需要的时候按需读取。那么`nodejs`和`c++`是不是也可以这样交互数据呢，但是不确定 JSON 序列化和反序列化的开销会不会大于`c++`优化的性能。
- 还有一种更好的方案，使用 **Buffer** ! Buffer 是二进制数据，无需数据类型转换，而且 `nodejs` 创建的Buffer 受 v8 管理但是使用的是堆外内存。



那么下面也写个如和用buffer传递数据的小demo：

`c++`代码

```c++
// buffer.cc
#include <napi.h>

namespace __node_addon_api_buffer
{
  void UseBuffer(const Napi::CallbackInfo &info)
  {
    Napi::Env env = info.Env();

    // 获取传递给函数的Buffer
    Napi::Object bufferObj = info[0].As<Napi::Object>();
    int rot = info[1].As<Napi::Number>().Uint32Value();

    // 获取Buffer的信息
    void *data;
    size_t length;
    napi_status status = napi_get_buffer_info(env, bufferObj, &data, &length);

    if (status != napi_ok)
    {
      Napi::TypeError::New(env, "Failed to get buffer info").ThrowAsJavaScriptException();
      return;
    }

    // 直接在C++中操作Buffer的数据
    unsigned char *buffer = static_cast<unsigned char *>(data);
    for (size_t i = 0; i < length; i++)
    {
      buffer[i] += rot;
    }
  }

  Napi::Object Initialize(Napi::Env env, Napi::Object exports)
  {
    exports.Set(Napi::String::New(env, "useBuffer"), Napi::Function::New(env, UseBuffer));
    return exports;
  }

  NODE_API_MODULE(addon, Initialize)
}
```

(以上代码是ChatGPT生成的自己调了一下 ^.^)

配置

```json
// binding.gyp
{
    "targets": [
        {
            "target_name": "buffer-test",
            "cflags!": ["-fno-exceptions"],  # -fno-exceptions 忽略掉编译过程中的一些报错
            "cflags_cc!": ["-fno-exceptions"],
            "sources": ["addon/buffer.cc"],
            "include_dirs": [  # 头文件搜索路径，这样通过#include <napi.h>就可以引入napi
                "<!@(node -p \"require('node-addon-api').include\")"
            ],
            "defines": ["NAPI_DISABLE_CPP_EXCEPTIONS"],
        },
    ]
}
```

`nodejs`代码

```js
// test-buffer.js 

const buffer = require('bindings')('buffer-test')

// 创建Node.js Buffer
const nb = Buffer.from([1, 2, 3, 4, 5]);

// 调用C++函数并将Buffer传递给它
console.log(nb)
buffer.useBuffer(nb, 12);
console.log(nb)
```

执行

```shell
node test-buffer.js

<Buffer 01 02 03 04 05>
<Buffer 0d 0e 0f 10 11>
```

我成功修改了Buffer中的数据



demo代码地址：https://github.com/ruomuc/practice/tree/master/napi-test



参考链接：

https://zhuanlan.zhihu.com/p/584943566

https://blog.risingstack.com/using-buffers-node-js-c-plus-plus/
