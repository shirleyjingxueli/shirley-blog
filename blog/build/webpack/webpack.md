# webpack基础知识
### webpack 定义
  - webpack是一个基于js的静态模块打包工具。会根据一个或者多个入口点构建依赖图，将项目中所需的模块构建成一个或者多个bundles，它们均为静态资源，用于展示页面。
  - webpack本质：基于事件流的编程范例，一系列的插件运行

### bundle，chunk， module 是什么？
  - bundle: 由 webpack 打包出来的文件
  - chunk: 代码块，一个 chunk 由多个模块组合而成，用于代码合并和分割。
  - module: 开发中的单个模块，在 webpack 的世界，一切皆模块，一个模块对应一个文件， webpack 会从配置的 entry 中递归开始找出所有依赖的模块。

### webpack、grunt、gulp、rollup、vite、parcel区别

#### webpack 和 grunt、gulp 的不同？

  - **Grunt、Gulp 是基于任务运行的工具**：

  它们会自动执行指定的任务，像流水线，把资源放上去然后通过不同插件进行加工，它们包括活跃的社区，丰富的插件，能方便的打造各种工作流。

  - **Webpack 是基于模块化打包的工具**：

  自动化处理模块，Webpack 把一切都当成模块，当 Webpack 处理程序时，它会递归地构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将这些模块打包成一个或者多个 bundle

  **Webpack 和 Grunt、Gulp 是完全不同的两类工作，而现在主流的方式是用 npm script 代替 Grunt、Gulp, npm script 同样可以打造任务流。**  

#### webpack、rollup、parcel 优劣？

  - **webpack 适用于大型复杂的前端站点构建：** 

    webpack 由强大的 loader 和 插件生态，打包后的文件实际上就是一个立即执行函数，这个立即执行函数接收一个参数，这个参数就是模块对象，键为各个模块的路径，值为模块的内容。立即执行函数内部则处理模块之间的引用，执行模块等，这种情况更适合文件依赖复杂的应用开发

  - **rollup 适用于基础库的打包（如 vue、d3等）：**

    rollup 就是将各个模块打包进一个文件中，并且通过 Tree-shaking 来删除无用的代码，可以最大程度上降低代码体积，但是 rullup 没有 webpack 如此多的功能， 如：代码分割，按需加载等高级功能，其更聚焦于库的打包，因此更适合库的开发。

  - **parcel 适用于简单的实验性项目：**

    它可以满足低门槛的快速看到效果，但是生态差、报错信息不够全面都是它的硬伤，除了一下玩具项目或者实验项目，其他不建议使用。

#### vite
  vite 是一种新型的前端构建工具，能够显著提升前端开发体验。
  
  **特点：**

  - 极速的服务启动：

    使用原生 ESM 文件，无需打包
    
    Vite 将会使用 esbuild 预构建依赖。

    Vite 只需要在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入代码，即只在当前屏幕上实际使用时才会被处理。

  - 轻量快速的热重载

    无论应用程序大小如何，都始终极快的模块热替换（HMR）

    在 Vite 中，HMR 是在原生 ESM 上执行的。当编辑一个文件时，Vite 只需要精确地使已编辑的模块与其最近的 HMR 边界之间的链失活[1]（大多数时候只是模块本身），使得无论应用大小如何，HMR 始终能保持快速更新。

    Vite 同时利用 HTTP 头来加速整个页面的重新加载（再次让浏览器为我们做更多事情）：源码模块的请求会根据 304 Not Modified 进行协商缓存，而依赖模块请求则会通过 Cache-Control: max-age=31536000,immutable 进行强缓存，因此一旦被缓存它们将不需要再次请求。

  - 丰富的功能

    对 TS, JSX, CSS 等支持开箱即用

  - 优化的构建

    可选 多页应用 或者 库 模式的预配置 rollup 构建

  - 通用的插件

    在开发和构建之间共享 rollup-superset 插件接口

  - 完全类型化的API

    灵活的 API 和 完整的 TS 类型。

### webpack 基础用法
  - **entry: 入口**
    单入口： string ---> './src/index.js';
    多入口： object ---> {key: path}
    ```js
      entry: {
        index: './src/index.js',
        test: './src/test.js'
      }
      // 动态摄者多个entry
      entry: glob.sync(path.join(__dirname, './src/*/index.js'))
    ```
  - **output: 输出**
    单个bundle: string ---> './dist/bundle.js',
    多个bundle: object
    ```js
      output: {
        filename: [name].js,
        path: path.resolve(__dirname, 'dist'),
        clean: true, // 构建前会清理dist文件夹
      }
    ```
  - **loader: 用于对非js资源的解析，配置在module中**
    * 特点：
      1. loader 可以链式调用，最终的loader期待返回js
      2. loader 有先后顺序。webpack中， loader 的执行顺序是从右向左执行的。 因为 webpack 选择了 compose 这样的函数式编程方式，这种方式的表达式执行是从右向左的。
    * 使用方法：
      ```js
        module: {
          rules: [{
            test: /\.css$/i, // 指定匹配规则
            use: ['style-loader', 'css-loader'], // 指定loader
            parser: {  // 自定义parser，一般用不上
              parse: toml.parse 
            }
          }]
        }
      ```
    * 常见的loader：
      - css: style-loader, css-loader
        1. style-loader: 将css注入到dom中
        2. css-loader: 解析css，加载.css，并转换为commonjs对象
        3. scss-loader: 解析scss
        4. less-loader: 解析 less
      - images | fonts: ```type: 'asset/resource'``` ---> 相当于是file-loader
      - 数据加载
        1. json: webpack内置处理
        2. csv | tsv: csv-loader
        3. xml: xml-loader
      - 语法转换: babel-loader
      - ts: ts-loader  
      - 文件以字符串的形式插入: raw-loader
      - 多进程打包js和css: thread-loader
      - 语法转换: babel-loader
      - eslint 检查: eslint-loader
  - **plugins:**
    * 用于bundle文件的优化，资源管理，环境变量的注入，作用于整个构建过程中
    * 常见的plugin
      1. HtmlWebpackPlugin: 构建html承载bundle文件 ---> 构建html文件并且以script标签的形式引入打包好的bundle
      2. CleanWebpackPlugin: 清理webpack构建目录，类似output.clean
      3. UglifyjsWebpackPlugin: 压缩混淆js
      4. CommonsChunkPlugin: 将chunks相同的代码提取为公共js
      5. ExtractTextWebpackPlugin: 将css从bundle文件里提取为一个独立的css文件
      6. CopyWebpackPlugin: 将文件或者文件夹拷贝到构建的输出目录
      7. ZipWebpackPlugin: 将打包的资源生成一个zip包
      8. DefinePlugin: 定义环境变量
      9. WebpackParallelUglifyPlugin: 多核压缩，提高压缩速度
      10. WebpackBundleAnalyzer: 可视化 webpack 输出文件体积
      11. MiniCssExtraPlugin: CSS 提取到单独的文件中，支持按需加载
    * webpack插件本质
      1. webpack中的插件是一个具有apply方法的js对象，apply 方法会被 webpack compiler 调用，并且在整个编译生命周期里都可以访问compiler对象   
  - **mode：**
    * 用来指定当前的构建环境：production, development, none
    * 不同的模式会开启一些优化
  - **热更新：**
    * WDS
      1. 浏览器不刷新
      2. 不输出文件，保存在内存中
      3. 使用HotModuleReplacementPlugin
    * WDM 
      1. 输出文件传递给服务器
      2. 适用于灵活的定制场景
  - **文件指纹：**
    * Hash
      1. 使用场景：html设置hash，图片设置hash
    * ContentHash
      1. 使用场景：css设置hash
    * ChunkHash 
      1. 使用场景：js摄者hash   
  - **资源模块**
    * asset/resource: file-loader --> 发送一个单独的文件并且导出url
    * asset/inline: url-loader --> 导出一个资源的dataURI
    * asset/source: raw-loader --> 导出资源的源代码
  - **文件压缩**
    * html： html-webpack-plugin 设置压缩参数
    * js：uglifyjs-webpack-plugin
    * css: optimize-css-assets-webpack-plugin 同时使用cssnano
### 常见问题
  #### loader和plugin的区别
  **功能不同：**

  - loader用于资源解析，让 webpack 拥有了加载和解析非JS文件的能力。
  - plugin用于bundle优化，资源管理，环境变量的注入，作用于整个构建过程中

  **用法不同**
  
  - loader 在 module.rules中配置，它是作为模块的解析规则而存在的。类型为数组，每一项都是一个 Object，里面描述了对什么类型的文件（test），使用什么加载（loader）和使用的参数（options）
  - plugin 是在 plugins 中单独配置。类型为数组，每一项都是一个 plugin 的实例，参数都通过构造函数传入。

### webpack 进阶用法
#### env
  webpack中使用env时，需要将module.exports转换为一个函数，默认情况下module.exports指向的是一个对象
  ```js
    //  默认
    module.exports = {
      entry: xxx,
      output: xxx
    }

    // env下
    modules.exports = (env) => {
      return {
        entry: xxx,
        output: xxx,
      }
    }
  ```
#### 自动清理构建目录
  - clean-webpack-plugin: 默认会删除output指定的输出目录
  - 设置output.clean为true
    ```js
      output: {
        clean: true, //webpack5
      }
    ```
#### 自动补齐css前缀
  - postcss 插件 autoprefixer
    ```js
      {
        loader: 'postcss-loader',
        options: {
          plugins: () => [
            require('autoprefixer')({
              browsers: ["last 2 version", "> 1%", "iOS 7"]
            })
          ]
        }
      }
    ```
#### 响应式配置
  - px2rem loader
#### 资源内联
  - 意义
    * 代码层面：
      1. 页面框架的初始化脚本
      2. 上报相关打点
      3. css内联避免页面闪动
    * 请求层面：
      1. 减少http请求（小图片或者字体内联）
  - 使用：
    * html内联：raw-loader
      ```js
        // webpack4
        <script>${require('raw-loader!babel-loader!./meta.html')}</script>
      ```
    * css内联：style-loader 或者 html-inline-css-webpack-plugin
      ```js
      // style-loader
      module.exports = {
        module: {
          rules: [
          {
            test: /\.scss$/,
            use: [
              {
                loader: 'style-loader',
                options: {
                  insertAt: 'top', // 样式插入到 <head>
                  singleton: true, //将所有的style标签合并成一个
                }
              },
              "css-loader",
              "sass-loader"
            ],
          },
          ]
        },
      };
      ```
      ```js
        // html-inline-css-webpack-plugin: 需要和mini-css-extract-plugin和html-webpack-plugin一起使用
        const MiniCssExtractPlugin = require("mini-css-extract-plugin");
        const HtmlWebpackPlugin = require('html-webpack-plugin');
        const HTMLInlineCSSWebpackPlugin = require("html-inline-css-webpack-plugin").default;

        module.exports = {
          plugins: [
            new MiniCssExtractPlugin({
              filename: "[name].css",
              chunkFilename: "[id].css"
            }),
            new HtmlWebpackPlugin(),
            new HTMLInlineCSSWebpackPlugin(),
          ],
          module: {
            rules: [
              {
                test: /\.css$/,
                use: [
                  MiniCssExtractPlugin.loader,
                  "css-loader"
                ]
              }
            ]
          }
        }
      ```
    * js内联: raw-loader
    ```js
      <script>${require('raw-loader!babel-loader!../node_modules/lib-flexible')}</script>
    ```
    * webpack5中只需要在loader配置中设置 ```type: 'asset/inline'```即可 
#### sourcemap
  - 作用
     * 将编译后的代码映射到源代码
     * 线下开启，线上关闭
     * 线上可以将sourcemap上传到错误监控系统（可参考https://www.jianshu.com/p/11ebf61aca6e）
     ```js
      // 上传sourcemap到sentry
      // 1. 配置sentry -> 初始化sentry -> 利用@sentry/webpack-plugin来上传 -> @sentry/webpack-plugin在上传后不会删除sourceMap，上传后需要删除.map代码
     ```
  - sourcemap类型
    ![Alt text](<截屏2023-07-04 下午8.14.37.png>)
#### 代码分离
  - **实现方式：**
    *  入口文件：使用 ```entry``` 配置手动分离代码
    * 防止重复：使用 ```Entry dependencies ``` 或者 ```SplitChunksPlugin ``` 去除和分离chunk
    ```js
      // entry dependencies
      module.exports = {
        mode: 'development',
        entry: {
          index: {
            import: './src/index.js',
            dependOn: 'shared',  // 重点
          },
          another: {
            import: './src/another-module.js',
            dependOn: 'shared', // 重点
          },
          shared: 'lodash',  //重点
        },
        output: {
          filename: '[name].bundle.js',
          path: path.resolve(__dirname, 'dist'),
        },
        optimization: {
          runtimeChunk: 'single',
        },
      };
      // SplitChunkPlugin
      optimization: {
        splitChunks: {
          chunks: 'all',
        },
      },
    ```
  * 动态导入：通过模块的内联函数调用来分离代码
    * 使用 es6语法： import(); ```推荐使用```
    * 使用 require.ensure();
  * 预获取、预加载模块（使用内置指令preload，prefetch）
    * preload(预加载)：当前导航下可能需要资源
      ```js
        import(/* webpackPreload: true */ './path/to/LoginModal.js');
      ```
    * prefetch(预获取)：将来某些导航下可能需要的资源
      ```js
        import(/* webpackPrefetch: true */ './path/to/LoginModal.js');
      ```
    * preload和prefetch区别
      1. preload chunk 会在父 chunk 加载时，以并行方式开始加载。prefetch chunk 会在父 chunk 加载结束后开始加载。
      2. preload chunk 具有中等优先级，并立即下载。prefetch chunk 在浏览器闲置时下载。
      3. preload chunk 会在父 chunk 中立即请求，用于当下时刻。prefetch chunk 会用于未来的某个时刻。
      4. 浏览器支持程度不同。
  - **使用场景**
    * 提取公共代码到一个公共块
    * 脚本懒加载，减小初始下载体积
#### 基础库的分离
  - 使用html-webpack-externals-plugin
  - splitChunksPlugin
    * chunks: async: 异步代码进行分离, initial: 同步代码进行分离, all: 同步和异步都进行分离
    * 使用方法：
      1. test 匹配出需要分离的包
      2. miniChunks 设置最小引用次数
      3. miniSize 分离包体积的大小
  - treeShaking
    * 原理：
      1. 利用es6模块特点：只能在顶层模块中出现；import模块名只能是字符串变量；import变量是immutable的
      2. 在uglify阶段剔除无用代码
    * 只把使用到的方法打包到bundle，未使用的方法会在uglify阶段s除掉
    * webpack默认支持，在.babelrc设置modules:false即可，production mode下默认开启
    * 要求必须是es6语法，cjs不支持
#### eslint 落地
  - 和CI/CD集成
  - 和webpack集成  
#### webpack打包库
  - output中设置library, libraryTarget, libraryExport;
  ```js
    // 将根据入口文件，打包成对应的umd形式的lib库
    // 不推荐使用数组作为库的entry
    output: {
      library: {
        name: 'webpack_lib',
        type: 'umd'
      }
    }
  ```
#### 优化命令行构建日志
  - 使用 friendly-error-webpack-plugin  
#### 预置依赖
  - 预置全局变量: ProvidePlugin
    ```js
      plugins: [
        new webpack.ProvidePlugin({
          _: 'lodash',
        }),
      ]
    ```  
   - 细颗粒度shimming: imports-loader 
   - 全局exports: exports-loader
   - 加载polyfill: babel-preset-env



## webpack进阶
### 构建配置抽离成npm包
### 冒烟测试、单元测试、测试覆盖率
### webpack进阶-构建速度和体积优化
#### 速度、体积分析
  - 速度分析: speed-measure-webpack-plugin ---> 分析打包总耗时，以及每个loader，插件打包耗时
  - 体积分析: webpack-bundle-analyzer ---> 分析依赖的第三方库的大小，业务代码体积大小  
#### 多进程、多实例
  - 资源并行解析
    * thread-loader
    * HappyPack
    * parallel-webpack
  - 并行压缩
    * parallel-uglify-plugin 
    * uglifyjs-webpack-plugin 开启 parallel 参数
    * terser-webpack-plugin 开启 parallel 参数 
#### 预编译模块
  利用 DllPlugin 和 DllRefreencePlugin 预编译资源模块。通过 DllPlugin 来对那些我们引用但是绝对不会修改的 npm 包来进行预编译，再通过 DllReferencePlugin 将预编译的模块加载进来 
#### 分包
  - 设置externals
#### 提取公共代码
  - 多入口情况下，通过 CommonChunkPlugin 来提取公共代码  
#### 缓存
  - 目的：提升二次构建速度
  - 思路：
    * babel-loader 开启缓存
    * terser-loader 开启缓存
    * 使用cache-loader 或者 hard-source-webpack-plugin
#### 缩小构建目标
  - 目的：尽可能的减少构建模块
  - 思路：
    * babel-loader不解析node_modules
#### 减少文件搜索范围
  - 优化 resolve.modules 配置
  - 优化 resolve.mainFileds 配置
  - 优化 resolve.extension 配置
  - 合理使用 alias 
#### 图片压缩
  - image-webpack-loader
#### treeshaking
  - 剔除无用的代码
#### 剔除无用的css
  - PurifyCSS: 遍历代码，识别已经用到的 CSS class 
  - uncss: HTML 需要通过 jsdom 加载，所有的样式通过PostCSS解析，通过 document.querySelector 来识别在 html 文件里面不存在的选择器
#### 构建体积优化
  - 根据体积分析结果，动态引入
  - 动态polyfill: polyfill-service ----> 原理：识别userAgent，下发不同的polyfill
  
### 体积优化总结：
  - 压缩代码：

    删除多余的代码、注释、简化代码的写法等等方式。可以利用  webpack 的 UglifyJsPlugin 和 ParalleUglifyPlugin 来压缩 JS 文件，利用 cssnano(css-loader?minimize) 来压缩 css

  - Scope Hosting
  - Tree-shaking 
  - code splitting
  - 公共资源分离 
  - 图片压缩 
  - 动态 Polyfill   


## 名词解释
### Scope Hosting
  webpack 中的 scope Hosting 是 webpack 的一项优化功能，旨在减小打包文件的大小并提高性能。它主要通过分析模块之间的依赖关系并进行代码优化来实现这一目标。

  Scope Hosting 的原理是将具有相同作用域（Scope）的模块合并到一个函数中，从而减少函数声明的树龄。这种优化在支持 ES6 模块的现代浏览器和 JavaScript 引擎中特别有效。

  **Scope Hosting 的优势**

  - 减小打包文件的大小：通过减少函数声明的数量，可以减小打包后 JavaScript 文件的大小
  - 提高性能：减少函数声明也意味着减少了函数调用和上下文切换，从而提高了代码的执行性能。
  - 改善可读性：合并具有相同作用域的模块可以改善代码的可读性，因为它们在同一个函数中，更容易理解和维护。

  **启用 Scope Hosting**

  要启用 Scope Hosting，通常需要确保你的 webpack 配置采用了支持此功能的最新版本，并且你的代码符合相应的要求。此外，还可以使用 webpack 插件和工具来进一步优化代码。

  在 webpack 5 中，Scope Hosting 是默认开启的，但也可以通过配置文件进行自定义。

  **可以使用 `optimization.concatenateModules` 选项来配置 Scope Hosting 相关的参数**

  ```js
    module.exports = {
      optimization: {
        concatenateModules: true.
      }
    }
  ```