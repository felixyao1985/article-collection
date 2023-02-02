# 较容易理解的webpack性能优化
[(2 条消息) 较容易理解的 webpack 性能优化\_CRMEB 的博客 - CSDN 博客](https://blog.csdn.net/CRMEB/article/details/123728262) 

 优化环境配置  
1-webpack 性能优化  
开发环境性能优化  
生产环境性能优化  
开发环境性能优化  
优化打包构建速度  
HMR  
优化代码调试  
source-map  
生产环境性能优化  
优化打包构建速度  
oneOf  
[babel](https://so.csdn.net/so/search?q=babel&spm=1001.2101.3001.7020)缓存  
多进程打包  
externals  
dll  
优化代码运行的性能  
缓存 (hash-chunkhash-contenthash)  
tree shaking  
code split  
懒加载 / 预加载  
pwa  
01-HMR 热模块替换  
基于开发环境的 devserver（开发环境没有 hmr 功能）

HMR: hot module replacement 热模块替换 / 模块热替换

作用：一个模块发生变化，只会重新打包这一个模块（而不是打包所有模块） 极大提升构建速度

样式文件：可以使用 HMR 功能：因为 style-loader 内部实现了~

js 文件：默认不能使用 HMR 功能–> 需要修改 js 代码，添加支持 HMR 功能的代码。

注意：HMR 功能对 js 的处理，只能处理非入口 js 文件的其他文件。

html 文件: 默认不能使用 HMR 功能. 同时会导致问题：html 文件不能[热更新](https://so.csdn.net/so/search?q=%E7%83%AD%E6%9B%B4%E6%96%B0&spm=1001.2101.3001.7020)了~ （不用做 HMR 功能）

解决：修改 entry 入口，将 html 文件引入

```sql
entry: ['./src/js/index.js', './src/index.html'],
  ...  
  devServer: {
    static: resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
    open: true,
    
    
    hot: true
  }
  
  
在需要热模块替换的非入口js文件中协商如下代码，可实现该方法的热模块替换
if (module.hot) {
  
  module.hot.accept('./print.js', function() {
    
    
    print();
  });
}

复制代码

```

02-source-map  
（开发环境） source-map: 一种 提供源代码到构建后代码映射 技术 （如果构建后代码出错了，通过映射可以追踪源代码错误）

```
devtool: '...'

    <!--[inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map-->

    <!--source-map：外部-->
    <!--  错误代码准确信息 和 源代码的错误位置-->
    <!--inline-source-map：内联-->
    <!--  只生成一个内联source-map-->
    <!--  错误代码准确信息 和 源代码的错误位置-->
    <!--hidden-source-map：外部-->
    <!--  错误代码错误原因，但是没有错误位置-->
    <!--  不能追踪源代码错误，只能提示到构建后代码的错误位置-->
    <!--eval-source-map：内联-->
    <!--  每一个文件都生成对应的source-map，都在eval-->
    <!--  错误代码准确信息 和 源代码的错误位置-->
    <!--nosources-source-map：外部-->
    <!--  错误代码准确信息, 但是没有任何源代码信息-->
    <!--cheap-source-map：外部-->
    <!--  错误代码准确信息 和 源代码的错误位置 -->
    <!--  只能精确的行-->
    <!--cheap-module-source-map：外部-->
    <!--  错误代码准确信息 和 源代码的错误位置 -->
    <!--  module会将loader的source map加入-->

    <!--内联 和 外部的区别：1. 外部生成了文件，内联没有 2. 内联构建速度更快-->

    <!--开发环境：速度快，调试更友好-->
    <!--  速度快(eval>inline>cheap>...)-->
    <!--    eval-cheap-souce-map-->
    <!--    eval-source-map-->
    <!--  调试更友好  -->
    <!--    souce-map-->
    <!--    cheap-module-souce-map-->
    <!--    cheap-souce-map-->

    <!--  --> eval-source-map  / eval-cheap-module-souce-map-->

    <!--生产环境：源代码要不要隐藏? 调试要不要更友好-->
    <!--  内联会让代码体积变大，所以在生产环境不用内联-->
    <!--  nosources-source-map 全部隐藏-->
    <!--  hidden-source-map 只隐藏源代码，会提示构建后代码错误信息-->

    <!--  --> source-map / cheap-module-souce-map-->


复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/11ab6025-0f5a-4f1b-b566-4bcd7f4c4043.png?raw=true)


```

03-oneof  
优化生产环境打包构建速度的

以下 loader 只会匹配一个

注意：不能有两个配置处理同一种类型文件

oneof 使一个文件只会匹配一个 loader，否则会进行多次匹配。不能有两个配置处理同一种类型文件。因为[eslint](https://so.csdn.net/so/search?q=eslint&spm=1001.2101.3001.7020)-loader 和 babel-loader 都匹配 js 文件，所以要把一个 loader 放在 oneof 外面。

```
 `module: {
    rules: [
      {
        
        test: /\.js$/,
        exclude: /node_modules/,
        
        enforce: 'pre',
        loader: 'eslint-loader',
        options: {
          fix: true
        }
      },
      {
        
        
        oneOf: [
          {
            test: /\.css$/,
            use: [...commonCssLoader]
          },
          {
            test: /\.less$/,
            use: [...commonCssLoader, 'less-loader']
          },
          
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
            options: {
              presets: [
                [
                  '@babel/preset-env',
                  {
                    useBuiltIns: 'usage',
                    corejs: {version: 3},
                    targets: {
                      chrome: '60',
                      firefox: '50'
                    }
                  }
                ]
              ]
            }
          },
          {
            exclude: /\.(js|css|less|html|jpg|png|gif)/,
            loader: 'file-loader',
            options: {
              outputPath: 'media'
            }
          }
        ]
      }
    ]
  },
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/988e1c07-3965-4943-86d1-78dd4b389cd4.png?raw=true)

```

04 - 缓存  
类似开发环境的 hmr 缓存：

babel 缓存 cacheDirectory: true–> 让第二次打包构建速度更快  
文件资源缓存  
hash: 每次 wepack 构建时会生成一个唯一的 hash 值。  
问题: 因为 js 和 css 同时使用一个 hash 值。 如果重新打包，会导致所有缓存失效。（可能我却只改动一个文件）  
chunkhash：根据 chunk 生成的 hash 值。如果打包来源于同一个 chunk，那么 hash 值就一样  
问题: js 和 css 的 hash 值还是一样的 因为 css 是在 js 中被引入的，所以同属于一个 chunk  
contenthash: 根据文件的内容生成 hash 值。不同文件 hash 值一定不一样 --> 让代码上线运行缓存更好使用。

```
 `{
    test: /\.js$/,
    exclude: /node_modules/,
    loader: 'babel-loader',
    options: {
      presets: [
        [
          '@babel/preset-env',
          {
            useBuiltIns: 'usage',
            corejs: { version: 3 },
            targets: {
              chrome: '60',
              firefox: '50'
            }
          }
        ]
      ],
      
      
      cacheDirectory: true
      }
},

  output: {
    filename: 'js/built.[contenthash:10].js',
    path: resolve(__dirname, 'build')
  },
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/46329ddf-cdcd-461b-814b-fe94ae44f3f4.png?raw=true)


```

05-tree-shaking  
tree shaking：去除无用代码

前提：1. 必须使用 ES6 模块化 2. 开启 production 环境  
作用: 减少代码体积  
在 package.json 中配置

```sql
 "sideEffects": false 所有代码都没有副作用（都可以进行tree shaking）
    问题：可能会把css / @babel/polyfill （副作用）文件干掉
  "sideEffects": ["*.css", "*.less"]
复制代码

```

06 - 代码分割  
入口几个文件 出口就几个文件

```sql
 entry: {
    
    index: './src/js/index.js',
    test: './src/js/test.js'
  },
 output: {
    
    filename: 'js/[name].[contenthash:10].js',
    path: resolve(__dirname, 'build')
  },
复制代码

```

可以将 node_modules 中代码单独打包一个 chunk 最终输出

```sql
 
  optimization: {
    splitChunks: {
      chunks: 'all'
    }
  },
复制代码

```

单入口

```
 `import('./test')
  .then(({ mul, count }) => {
    // 文件加载成功~
    // eslint-disable-next-line
    console.log(mul(2, 5));
  })
  .catch(() => {
    // eslint-disable-next-line
    console.log('文件加载失败~');
  });
    
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/1bb86d75-99a8-4e0a-8e2b-8ee8692a934f.png?raw=true)



```

07 - 懒加载  
懒加载：当文件需要使用时才加载 预加载 prefetch：会在使用之前，提前加载 js 文件 正常加载可以认为是并行加载（同一时间加载多个文件）  
预加载 prefetch：等其他资源加载完毕，浏览器空闲了，再偷偷加载资源 (预加载兼容问题有点严重)

console.log(‘index.js 文件被加载了~’);

```sql


document.getElementById('btn').onclick = function() {
  
  
  
  
  import('./test').then(({ mul }) => {
    console.log(mul(4, 5));
  });
};

复制代码

```

08-PWA 渐进式网络开发应用程序 （离线可访问）  
workbox ---->workbox-wepack-plugin

webpack.config.js

```sql
plugins: [
    new WorkboxWebpackPlugin.GenerateSW({
      
      clientsClaim: true,
      skipWaiting: true
    })
  ],

复制代码

```

入口 js 文件配置

```
 `if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker
      .register('/service-worker.js')
      .then(() => {
        console.log('sw注册成功了~');
      })
      .catch(() => {
        console.log('sw注册失败了~');
      });
  });
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/3bf29817-9f45-4d10-a957-749b96bdbdf1.png?raw=true)



```

09 - 多进程打包  
下载 thread-loader，对谁进行多进程打包就把 thread-loader 配置在它的后面

开启多进程打包。 （有利有弊） 进程启动大概为 600ms，进程通信也有开销。 只有工作消耗时间比较长，才需要多进程打包

```
 `{
    test: /\.js$/,
    exclude: /node_modules/,   
    use: [
      
      {
        loader: 'thread-loader',
        options: {
          workers: 2 
        }
      },
      {
        loader: 'babel-loader',
        options: {
          presets: [
            [
              '@babel/preset-env',
              {
                useBuiltIns: 'usage',
                corejs: { version: 3 },
                targets: {
                  chrome: '60',
                  firefox: '50'
                }
              }
            ]
          ],
          
          
          cacheDirectory: true
        }
      }
    ]
  },
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/91c5e7c4-c6c9-443a-ba91-2aa5caa306a9.png?raw=true)



```

10-externals  
有些包需要用 cdn 引入进来，防止将这些包打包进来，使用 externals 忽略这个包名。

```sql
  mode: 'production',
  externals: {
    
    jquery: 'jQuery'
  }
复制代码

```

```sql
<script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
复制代码

```

11-dll  
对代码单独打包

新建 webpack.dll.js

```
 `const { resolve } = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    
    
    jquery: ['jquery'],
  },
  output: {
    filename: '[name].js',
    path: resolve(__dirname, 'dll'),
    library: '[name]_[hash]' 
  },
  plugins: [
    
    new webpack.DllPlugin({
      name: '[name]_[hash]', 
      path: resolve(__dirname, 'dll/manifest.json') 
    })
  ],
  mode: 'production'
};

复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/7efdd541-120a-40c2-93ef-789357b27c18.png?raw=true)



```

webpack.config.js

```
`const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'built.js',
    path: resolve(__dirname, 'build')
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    }),
    
    new webpack.DllReferencePlugin({
      manifest: resolve(__dirname, 'dll/manifest.json')
    }),
    
    new AddAssetHtmlWebpackPlugin({
      filepath: resolve(__dirname, 'dll/jquery.js')
    })
  ],
  mode: 'production'
};` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/c5af9eab-6972-4dfc-8438-7c0887e5e013.png?raw=true)



```

我们首先封装一个响应式处理的方法 defineReactive，通过 defineProperty 这个方法重新定义对象属性的 get 和 set 描述符，来实现对数据的劫持，每次 读取数据 的时候都会触发 get ，每次 更新数据 的时候都会触发 set ，所以我们可以在 set 中触发更新视图的方法 update 来实现一个基本的响应式处理。

```
 `function defineReactive(obj, key, val) {
  
  Object.defineProperty(obj, key, {
    
    get() {
      console.log('🚀🚀~ get:', key);
      return val
    },
    
    set(newVal) {
      
      if (newVal !== val) {
        console.log('🚀🚀~ set:', key);
        val = newVal
        
        update()
      }
    }
  })
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/6a086b6b-2bd5-4971-a4dd-e3322ae4402a.png?raw=true)


```

我们写点代码来测试一下，每 1s 修改一次 obj.foo 的值 ， 并定义一个 update 方法来修改 app 节点的内容。

```
 `<div id='app'>123</div>

const obj = {}
defineReactive(obj, 'foo', '')

obj.foo = new Date().toLocaleTimeString()

setInterval(() => {
  obj.foo = new Date().toLocaleTimeString()
}, 1000)

function update() {
  app.innerHTML = obj.foo
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/160f3397-ea8c-4a95-845c-f585caafefaa.png?raw=true)


```

可以看到，每次修改 obj.foo 的时候，都会触发我们定义的 get 和 set ，并调用 update 方法更新了视图，到这里，一个最简单的响应式处理就完成了。

处理深层次的嵌套  
一个对象通常情况下不止一个属性，所以当我们要给每个属性添加响应式的时候，就需要遍历这个对象的所有属性，给每个 key 调用 defineReactive 进行处理。

```
 `function observe(obj) {
  
  if (typeof obj !== 'object' || obj === null) {
    return
  }
  
  Object.keys(obj).forEach(key => {
    defineReactive(obj, key, obj[key])
  })
}

const obj = {
  foo: 'foo',
  bar: 'bar',
  friend: {
    name: 'aa'
  }
}


obj.bar = 'barrrrrrrr' 
obj.foo = 'fooooooooo' 


obj.friend.name = 'bb' 
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/f6d2e1ae-f417-4d8b-8f93-308e44e39eae.png?raw=true)


```

当我们访问 obj.friend.name 的时候，也只是打印出来 get: friend ，而不是 friend.name ， 所以我们还要进行个 递归，把 深层次的属性 同样也做响应式处理。

```sql
function defineReactive(obj, key, val) {
  
  observe(val)
  
  
  Object.defineProperty(obj, key, {
    ... ...
  })
}


obj.friend.name = 'bb' 
复制代码

```

递归的时机在 defineReactive 这个方法中，如果 value 是对象就进行递归，如果不是对象直接返回，继续执行下面的代码，保证 obj 中嵌套的属性都进行响应式的处理，所以当我们再次访问 obj.friend.name 的时候，就打印出了 set: name 。

处理直接赋值一个对象  
上面已经实现了对深层属性的响应式处理，那么如果我直接给属性赋值一个对象呢？

```sql
const obj = {
  friend: {
    name: 'aa'
  }
}
obj.friend = {           
  name: 'bb'
}
obj.friend.name = 'cc'   
复制代码

```

这种赋值方式还是只打印出了 get: friend ，并没有劫持到 obj.friend.name ，那怎么办呢？我们只需要在 触发 set 的时候，判断一下 value 的类型，如果它是个对象类型，我们就对他执行 observe 方法。

```
`function defineReactive(obj, key, val) {
  Object.defineProperty(obj, key, {
    ... ...
    set(newVal) {
      
      if (newVal !== val) {
        console.log('🚀🚀~ set:', key);
        
        if (typeof obj === 'object' && obj !== null) {
          observe(newVal)
        }
        val = newVal
      }
    }
  })
}

obj.friend = {
  name: 'bb'
}

obj.friend.name = 'cc'  
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/36c21780-f515-4cb5-89fe-feb2c9edb793.png?raw=true)



```

处理新添加一个属性  
上面的例子都是操作 已经存在 的属性，那么如果我们 新添加 一个属性呢？

```sql
const obj = {}
obj.age = 18
obj.age = 20
复制代码

```

当我们试图修改 obj.age 的时候，什么都没有打印出来，说明并没有对 obj.age 进行响应式处理。这里也非常好理解，因为新增加的属性并没有经过 defineReactive 的处理，所以我们就需要一个方法来手动处理新添加属性这种情况。

```
 `function $set(obj, key, val) {
  
  defineReactive(obj, key, val)
}


$set(obj, 'age', 18)

obj.age = 20 
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/e97b18a8-d54a-43b2-a9bb-e440344ab20e.png?raw=true)



```

新定义的 $set 方法，内部也是把目标属性进行了 defineReactive 处理，这时我们再次更新 obj.age 的时候，就打印出了 set: age ， 也就实现了一个响应式的处理。

VUE 中的数据响应式  
实现简易的 Vue  
这是 Vue 中最基本的使用方式，创建一个 Vue 的实例，然后就可以在模板中使用 data 中定义的响应式数据了，今天我们就来完成一个简易版的 Vue 。

```
``<div id='app'>
  <p>{{counter}}</p>
  <p>{{counter}}</p>
  <p>{{counter}}</p>
  <p my-text='counter'></p>
  <p my-html='desc'></p>
  <button @click='add'>点击增加</button>
  <p>{{name}}</p>
  <input type="text" my-model='name'>
</div>

<script>
  const app = new MyVue({
    el: "#app",
    data: {
      counter: 1,
      desc: `<span style='color:red' >一尾流莺</span>`
    },
    methods: {
      add() {
        this.counter++
      }
    }
  })
</script>
复制代码`` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/3dcff2b7-5f36-43df-a53d-8cf7f44522eb.png?raw=true)


```

原理

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/299737f3-750d-46e5-8848-e62c4864f9c3.png?raw=true)

设计类型介绍  
MyVue： 框架构造函数  
Observer：执行数据响应化（区分数据是对象还是数组）  
Compile：编译模板，初始化视图，收集依赖（更新函数，创建 watcher）  
Watcher：执行更新函数（更新 dom ）  
Dep：管理多个 Watcher 批量更新  
流程解析  
初始化时通过 Observer 对数据进行响应式处理，在 Observer 的 get 的时候创建一个 Dep 的实例，用来通知更新。  
初始化时通过 Compile 进行编译，解析模板语法，找到其中动态绑定的数据，从 data 中获取数据并初始化视图，把模板语法替换成数据。  
同时进行一次订阅，创建一个 Watcher ，定义一个更新函数 ，将来数据发生变化时，Watcher 会调用更新函数 把 Watcher 添加到 dep 中 。  
Watcher 是一对一的负责某个具体的元素，data 中的某个属性在一个视图中可能会出现多次，也就是会创建多个 Watcher，所以一个 Dep 中会管理多个 Watcher。  
当 Observer 监听到数据发生变化时，Dep 通知所有的 Watcher 进行视图更新。  
代码实现 - 第一回合 数据响应式  
observe  
observe 方法相对于上面，做了一小点的改动，不是直接遍历调用 defineReactive 了，而是创建一个 Observer 类的实例 。

```sql

function observe(obj) {
  
  if (typeof obj !== 'object' || obj === null) {
    return
  }
  new Observer(obj)
}
复制代码

```

Observer 类  
Observer 类之前有解释过，它就是用来 做数据响应式 的，在它内部区分了数据是 对象 还是 数组 ，然后执行不同的响应式方案。

```
 `class Observer {
  constructor(value) {
    this.value = value
    if (Array.isArray(value)) {
      
    } else {
      
      this.walk(value)
    }
  }

  
  walk(obj) {
    
    Object.keys(obj).forEach(key => {
      defineReactive(obj, key, obj[key])
    })
  }
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/136cf5c8-7048-486d-9c5e-cd85e9e78a90.png?raw=true)




```

MVVM 类（MyVue）  
这一回合我们就先在实例初始化的时候，对 data 进行响应式处理，为了能用 this.key 的方式访问 this.$data.key，我们需要做一层代理。

```sql
class MyVue {
  constructor(options) {
    
    this.$options = options
    this.$data = options.data

    
    observe(this.$data)

    
    proxy(this)
  }
}
复制代码

```

proxy 代理也非常容易理解，就是通过 Object.defineProperty 改变一下引用。

```
 `function proxy(vm) {
  Object.keys(vm.$data).forEach(key => {
    
    Object.defineProperty(vm, key, {
      get() {
        return vm.$data[key]
      },
      set(newValue) {
        vm.$data[key] = newValue
      }
    })
  })
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/60c0ce70-f458-452d-b385-2f8f6cbdcd47.png?raw=true)




```

代码实现 - 第二回合 模板编译  
这一趴要实现下面这个流程，VNode 不是本文的重点，所以先去掉 Vnode 的环节，内容都在注释里啦~

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/16dcccb0-b002-47c3-9bec-16e43f1b31d9.png?raw=true)

```
 `class Compile {
  
  constructor(el, vm) {
    this.$vm = vm
    this.$el = document.querySelector(el)

    
    if (this.$el) {
      this.compile(this.$el)
    }
  }

  
  compile(el) {
    
    const childNodes = el.childNodes
    childNodes.forEach(node => {
      
      if (node.nodeType === 1) { 
        
        const attrs = node.attributes
        
        Array.from(attrs).forEach(attr => {
          
          
          const attrName = attr.name
          
          const exp = attr.value
          
          if (attrName.startsWith('my-')) {
            
            const dir = attrName.substring(3)
            
            this[dir] && this[dir](node, exp)
          }
        })
      } else if (this.isInter(node)) { 
        
        this.compileText(node)
      }
      
      if (node.childNodes) {
        this.compile(node)
      }
    })
  }

  
  compileText(node) {
    
    
    
    node.textContent = this.$vm[RegExp.$1]
  }

  
  text(node, exp) {
    
    
    node.textContent = this.$vm[exp]
  }

  
  html(node, exp) {
    
    
    node.innerHTML = this.$vm[exp]
  }

  
  isInter(node) {
    return node.nodeType === 3 && /\{\{(.*)\}\}/.test(node.textContent)
  }
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/5b5335c2-c006-44c2-9b8d-809969af9a7d.png?raw=true)



```

代码实现 - 第三回合 收集依赖  
视图中会用到的 data 中的属性 key 的地方，都可以被称为一个 依赖 ，同一个 key 可能会出现多次，每次出现都会创建一个 Watcher 进行维护，这些 Watcher 需要收集起来统一管理，这个过程叫做 收集依赖。

同一个 key 创建的多个 Watcher 需要一个 Dep 来管理，需要更新时由 Dep 统一进行通知。

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/f7824b4e-f1a5-478b-8c37-9fdf3aa12308.png?raw=true)

上面这段代码中，name1 用到了两次， 创建了两个 Watcher ，Dep1 收集了这两个 Watcher ，name2 用到了一次， 创建了一个 Watcher ， Dep2 收集了这一个 Watcher。

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/b647b22d-6485-4cb4-96fb-bee25084ec40.png?raw=true)

收集依赖的思路  
defineReactive 时为每一个 key 创建一个 Dep 实例  
初始化视图时，读取某个 key，例如 name1，创建一个 Watcher1  
由于触发 name1 的 getter 方法，便将 Watcher1 添加到 name1 对应的 Dep 中  
当 name1 发生更新时，会触发 setter，便可通过对应的 Dep 通知其管理的所有 Watcher 进行视图的更新  
Watcher 类  
收集依赖的过程，在 Watcher 实例创建的时候，首先把实例赋值给 Dep.target，手动读一下 data.key 的值 ，触发 defineReactive 中的 get ，把当前的 Watcher 实例添加到 Dep 中进行管理，然后再把 Dep.target 赋值为 null。

```
 `class Watcher {
  
  constructor(vm, key, updateFn) {
    this.vm = vm
    this.key = key
    this.updateFn = updateFn

    
    Dep.target = this
    
    this.vm[this.key]
    
    Dep.target = null
  }

  
  update() {
    
    this.updateFn.call(this.vm, this.vm[this.key])
  }
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/da5a15a3-c534-4c3a-b902-fdb9f5756df6.png?raw=true)


```

Dep 类  
addDep 方法把 Watchers 收集起来 放在 deps 中进行管理，notify 方法通知 deps 中的所有 Watchers 进行视图的更新。

```sql
class Dep {
  constructor() {
    this.deps = [] 
  }
  
  addDep(dep) {
    this.deps.push(dep)
  }

  
  notify() {
    this.deps.forEach(dep => dep.update())
  }
}
复制代码

```

升级 Compile  
在第二回合中，我们的 Compile 类只实现了视图的初始化，所以在第三回合中要把它升级一下，支持视图的更新。

Watcher 实例就是在初始化后创建的，用来监听更新。

```
`class Compile {
  ... ... 
    
  
  update(node, exp, dir) {
    
    const fn = this[dir + 'Updater']
    
    fn && fn(node, this.$vm[exp])
    
    new Watcher(this.$vm, exp, function(val) {
      fn && fn(node, val)
    })

  }

  
  compileText(node) {
    
    
    
    this.update(node, RegExp.$1, 'text')
  }

  
  text(node, exp) {
    this.update(node, exp, 'text')
  }

  
  textUpdater(node, value) {
    
    
    node.textContent = value
  }

  
  html(node, exp) {
    this.update(node, exp, 'html')
  }

  
  htmlUpdater(node, value) {
    
    
    node.innerHTML = value
  }

  
  isInter(node) {
    return node.nodeType === 3 && /\{\{(.*)\}\}/.test(node.textContent)
  }

}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/9939485b-596c-4098-97af-3547c3e7259b.png?raw=true)



```

Watcher 和 Dep 建立关联  
首先在 defineReactive 中创建 Dep 实例，与 data.key 是一一对应的关系，然后再 get 中 调用 dep.addDep 进行依赖的收集，Dep.target 就是一个 Watcher。在 set 中 调用 dep.notify() 通知所有的 Watchers 更新视图。

```
`function defineReactive(obj, key, val) {
  ... ... 
  
  const dep = new Dep()

  
  Object.defineProperty(obj, key, {
    
    get() {
      console.log('🚀🚀~ get:', key);
      
      Dep.target && dep.addDep(Dep.target)

      return val
    },
    
    set(newVal) {
      
      if (newVal !== val) {
        console.log('🚀🚀~ set:', key);
        
        if (typeof obj === 'object' && obj !== null) {
          observe(newVal)
        }
        val = newVal
        
        
        dep.notify()
      }
    }
  })
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/a8e62dec-eae9-4204-bd80-a8bfcaa6935f.png?raw=true)



```

代码实现 - 第四回合 事件和双向绑定  
事件绑定  
事件绑定也很好理解，首先判断节点的属性是不是以 @ 开头，然后拿到事件的类型，也就是例子中的 click， 再根据函数名找到 methods 中定义的函数体，最后添加事件监听就行了。

```
`class Compile {
  ... ... 
  compile(el) {
      
      if (this.isEvent(attrName)) {
        
        const dir = attrName.substring(1) 
        
        this.eventHandler(node, exp, dir)
      }
  }
  ... ... 
  
  isEvent(dir) {
    return dir.indexOf("@") === 0
  }
  eventHandler(node, exp, dir) {
    
    const fn = this.$vm.$options.methods && this.$vm.$options.methods[exp]
    
    node.addEventListener(dir, fn.bind(this.$vm))
  }
  ... ... 
}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/4542a063-8e82-4faf-a5c5-fb5eea9c6099.png?raw=true)


```

双向绑定  
my-model 其实也是一个指令，走的也是指令相关的处理逻辑，所以我们只需要添加一个 model 指令和对应的 modelUpdater 处理函数就行了。

my-model 双向绑定其实就是 事件绑定 和修改 value 的一个语法糖，本文以 input 为例，其它的表单元素绑定的事件会有不同，但是道理是一样的。

```
`class Compile {

  
  model(node, exp) {
    
    this.update(node, exp, 'model')
    
    node.addEventListener('input', e => {
      
      this.$vm[exp] = e.target.value
    })
  }

  modelUpdater(node, value) {
    
    node.value = value
  }

}
复制代码` 

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/9316948c-9b72-4495-8057-a62f06349238.png?raw=true)



```

现在也可以更新一下模板编译的流程图啦~

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2023-2-2%2011-02-29/8646fa89-6d46-4b57-94ae-4d98f722ee2e.png?raw=true)

最后  
如果你觉得此文对你有一丁点帮助，点个赞。或者可以加入我的开发交流群：1025263163 相互学习，我们会有专业的技术答疑解惑

如果你觉得这篇文章对你有点用的话，麻烦请给我们的开源项目点点 star:[http://github.crmeb.net/u/defu](http://github.crmeb.net/u/defu)不胜感激 ！

完整源码下载地址：[https://market.cloud.tencent.com/products/33272](https://market.cloud.tencent.com/products/33272)

PHP 学习手册：[https://doc.crmeb.com](https://doc.crmeb.com/)  
技术交流论坛：[https://q.crmeb.com](https://q.crmeb.com/)
