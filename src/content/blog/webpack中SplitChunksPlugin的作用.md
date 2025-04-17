---
title: webpack中SplitChunksPlugin的作用
date: 2022-11-14 15:07:22
categories: Code
tags:
  - JavaScript
  - webpack

id: "js-webpack"
recommend: true
---

### 摘要

:::note
   本文详细介绍了webpack的SplitChunksPlugin，用于优化代码分割。SplitChunksPlugin主要通过抽取公共模块和代码块，限制文件大小和请求数来提升性能。它会根据chunk类型、大小、请求数等因素进行分割，并提供了自定义配置选项。了解SplitChunksPlugin的配置和工作原理，能够帮助开发者更好地优化项目打包过程
:::

### SplitChunksPlugin

:::note
在讲这个插件前，跟大家区分下chunk和bundle的概念，
  >   chunk理解为“代码块”，例如node_module下的module，或者你自己import入页面的自定义js。

  >   bundle理解则可理解为已打包好的代码包，而代码包就是由很多chunk组成的了，就像vue-cli中build后会输出一个vender.js，这个js就是代码包了

:::

### 开始
:::note
  Originally, chunks (and modules imported inside them) were connected by a parent-child relationship in the internal webpack graph. The CommonsChunkPlugin was used to avoid duplicated dependencies across them, but further optimizations were not possible 

  译文：chunks(“代码块们”)和引入进去组成“代码块”的module们，它们之间的引用联系是通过webpack生成的父子关系图，而CommonsChunkPlugin 可以用来解决它们交叉或者重复调用的关系，但想进一步优化就显得不好搞了。所以webpack v4开始，就开始弃用了CommonsChunkPlugin
:::
### 特点

:::note
触发这个splitChunksPlugin切割代码的有以下几点：

  1. 共用的module (node_module文件夹的那些模块)和公共的chunk

  2. 输出的chunk体积大于30kb的（指还没有gz和min喔）

  3. 当加载请求数要求最大并行数小于或等于5时

  4. 初始化页面，加载请求数要求最大并行请求要小于或等于3时

  ps：想要满足第3和第4点，体积越大的代码块越容易
:::
### 配置

 据官方介绍，一般使用默认配置就能满足大部分的开发者了，如果你想针对你自己的网站进行优化，也可以自定义的。

    看下面的配置项，就是默认的配置项了

```js
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async',
      minSize: 30000,
      maxSize: 0,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      automaticNameDelimiter: '~',
      name: true,
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: 10
        },
        default: {
          minChunks: 2,
          priority: 20,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

  #### chunks：
  这个属性的可取值：all | async | initial

  all：entry入口所有的文件（不管是异步还是同步引入的）都会抽离出公共的，好处是文件会小且公共的文件只会加载一次，不好的地方是这个公共包有可能会很大。

  async：只抽离属于动态引入的文件。

  initial：只从入口模块进行拆分

  #### minSize：
  打包出来的chunks最小的size，单位是bytes ---- 1024bytes = 1kb

  #### maxSize：
  限制打包出来最大的size，这个是限制包的大小，切割包的优先级是最高的maxInitialRequest/maxAsyncRequests < maxSize < minSize，如果设置了这个最大size，其它一些的分包条件将不会起作用，因为这个maxSize的优先级高，所以像cacheGroups下的切分条件如果和maxSize的有冲突，会以maxSize为准   

  #### maxAsyncRequests：
  最大的动态异步文件请求数，首先要了解，什么是异步文件请求呢？使用import('xxxx/xxxx/xx.js').then()引入的模块，都是属于异步的，这里要区分一下import 'xx' from 'xxx/xx.js'语句,这个是同步静态引入的。maxAsyncRequests该属性的作用就是限制异步文件进行切分后的请求数，如下图

  ![aa2ea63ad8c5e7a46901598f721c5834.jpeg](https://wp-cdn.4ce.cn/v2/6D9H4Vn.jpeg)

  #### maxInitialRequests：
  这是用来限制入口的拆分数量

  #### automaticNameDelimiter：
  默认情况下，webpack将使用块的名称和名称生成名称（例如vendors~main.js），此选项允许您指定用于生成的名称的分隔符

  #### name:
  该属性值为true时，生成的

  #### cacheGroups:
  这个属性可以说是切分代码的核心属性了，该属性下每个键值对属性对象都是一个切分的配置项，每个配置可以顶掉上面所说那些属性如miniSize等，由于属性很多，大家可以看官方的介绍，我挑几个常用的来说下。

```js
cacheGroups: {
    vendors: {
    minChunks:2,
    chunks: 'initial',
    test: /[\\/]node_modules[\\/]/,
    priority: -10，
    filename: '[name].bundle.js',
    enforce: ''
},
    default: {
        minChunks: 2,
        priority: -20,
        reuseExistingChunk: true
    }
}
 
1、vendors，default是切分后代码块的命名，如果cacheGroups下面一个块名都没，切分出来的代码块还是有这两个的，是默认的。
 
2、test属性是指符合这个属性下设置的条件才会切分到这个块，上面是正则表达式，也可以是一个函数 
    test(module, chunks) {
        return module.type === 'javascript/auto';
    }
 
3、priority属性是设置切分优先级，假如有一个文件同时符合两个块的切分条件，那就是谁的priority值大，谁话事
 
4、filename属性是用来设置块的名称，值是'[name].bundle.js'这种
 
5、chunks和miniChunks和上面讲的作用一样，在这里设置的话，可以顶掉上面设置的
 
6、enforce属性就是强制切分到这个块，可以不受上面什么miniSize等的控制
 
7、reuseExistingChunk属性是重用切分的包，不切新的
```

### 总结：
学会一些webpack的常用配置，在项目中也不会至于对模块化和打包的机制一脸盲，特别现在的框架都需要打包工具打包生成代码，自己想要一些自定义化的打包，也还是能动手捣腾一下出来。
