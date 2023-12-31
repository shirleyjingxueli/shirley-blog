# webpack原理
### webpack打包原理
  - 查找webpack入口
    * node_modules/.bin/webpack.js
  - webpack.js
    * 找到并运行对应的cli，调用```runCli(cli)```方法
    * runCli: 创建并执行cli，并传入命令行参数
      ```js
        // 调用 WebpackCli 创建 cli 实例
        const cli = new WebpackCli();
        // 调用cli.run()方法
        await cli.run(args);
      ```  
  - webpack-cli.js
    * run()：
      调用 ```parseAsync(args)```,解析参数，生成配置项 
  - 引入webpack，根据配置项进行编译和构建
  - webpack-cli 执行结果
    * 根据对命令行参数和配置文件进行转换，生成配置选项options
    * 根据配置选项实例化webpack对象

### webpack打包流程
  - webpack打包主要分为初始化阶段、build阶段、生成（seal）阶段、写入（emit）阶段
#### ** webpack-初始化阶段 **
  - 调用  ```wepback(options,callback)``` 进行初始化，返回值为 ```{compiler, watch, watchOptions}```
  - 如果有传递callback参数时，会判断是否有```compiler.watch```方法，如果有会调用 ```compiler.watch()```, 否则调用```compiler.run()```方法
  - compiler.run()
    * 调用了 beforeRun -> run -> readRecords 钩子, 调用compile() 方法进行编译
    * compile() 方法： 调用 beforeCompile -> compile --- make -> finishMake 钩子
    * 在 process.nextTick 中调用 compilation.finish -> complication.seal -> afterCompiler钩子
#### ** webpack-构建阶段 **
  - hooks.make: 从entry开始递归的 分析依赖，对每个 依赖模块进行build
    * 收集 entryData 并保存在 entries
      ```js
        // 默认entryData 
        entryData = {
          dependencies: [],
          includeDependencies: [],
          options: {
            name: undefined,
            ...options
          }
        }
      ```
    * 通过 moduleFactory ，构建 moduleTree，每个模块都会生成一个module   
    * 调用 handleModuleCreation 对入口模块进行构建  
    * 调用 factorize、addModule、buildModule 进行构建，在build阶段涉及loader代码转换和依赖收集；模块构建完成后，若存在子依赖（module.dependencies），回到第三步开始子依赖的构建。
    * 在 compilaction.finish 阶段，make阶段任务完成
#### **webpack-生成阶段（seal）**    
  - seal：所有依赖build完成，进行优化
    * 根据 entires 创建对应 chunk 文件，并将它所依赖 module 的代码拼接生成 assets 对象。
    * assets 中存储了即将写入磁盘文件中的内容，它包含了入口模块和依赖模块的代码，以及像 __webpack_require__、__webpack_modules__ 运行时代码的拼接。
    * 和 seal相关的优化工作
#### **webpack-输出阶段（emit）**    
  - emit：输出到dist目录  
    * 根据seal阶段生成的内容，调用 ```compiler.emitAssets``` , 根据配置的output将文件写入系统中
  - compiler.emitAssets
    * 写入文件：hooks.emit ---> 根据output配置，生成目录 ---> 遍历 assets 写入  
    * 生成模块统计数据：new Stats(compilation)
    * 写入完成：hooks.done
    * 将 stats 统计数据传入并执行最初调用 run() 方法时所传入的 callback。

### webpack 热更新原理
  ![Alt text](image/hmr.png)

  webpack的热更新又称为热替换，缩写为 HMR。这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块。

  首先，server端和 client 端都做了处理工作。

  1. webpack 在 watch 模式下，文件系统重的某一个文件发生修改，webpack 监听到文件变化，根据配置文件对模块进行重新编译打包，并将打包后的代码通过简单的 JS 对象保存在内存中。

  2. webpack-dev-server 和 webpack 之间的接口交互：这一步主要是 dev-server 的中间件 webpack-dev-middleware 和 webpack 之间的交互，webpack-dev-middleware 调用 webpack 暴露的 API 对代码变化进行监控，并且告诉 webpack，将代码打包到内存中。

  3. 当我们在配置⽂件中配置了 devServer.watchContentBase 为 true 的时候，webpack-dev-server 会监听这些配置⽂件夹中静态⽂件的变化，变化后会通知浏览器端对应⽤进⾏ live reload。注意，这⼉是浏览器刷新，和 HMR 是两个概念。

  4. 通过 sockjs（webpack-dev-server的依赖）在浏览器和服务器之间建立一个 websocket 长连接，将 webpack 编译打包的各个阶段的状态信息告知浏览器端，同时也包括第三步中 server 监听静态文件变化的信息。浏览器根据这些 socket 消息进行不同的操作。当然服务端传递的最主要的信息还是新更新模块的 hash 值，后面的步骤根据这一 hash 值来进行模块热替换。

    webpack-dev-server/client 端并不能够请求更新的代码，也不会执行热更模块操作，而把这些工作又交回给了 webpack，webpack/hot/dev-server 的工作就是根据 webpack-dev-server/client 传给它的信息以及 dev-server 的配置决定是刷新浏览器呢还是进行模块热更新。当然如果仅仅是刷新浏览器，也就没有后面的步骤了。

    HotModuleReplacement.runtime 是客户端 HMR 的中枢，它接收到上一步传递给它的新模块的 hash 值， 它通过 JsonpMainTemplate.runtime 向 server 端发送 Ajax 请求，服务端返回一个 json，该 json 包含了所有要更新的模块的 hash 值，获取到更新列表之后，该模块再次通过 jsonp 请求，获取到最新的模块代码，这就是上图中的 7,8,9 步骤。

    第 10 步是决定 HMR 成功与否的关键步骤，在该步骤中， HotModulePlugin 将会对新旧模块进行对比，决定是否更新模块，在决定更新模块后，检查模块之间的依赖关系，更新模块的同时更新模块间的依赖引用。
  
  5. 最后一步，当 HMR 失败后，回退到 live reload 操作，也就是进行浏览器刷新来获取最新打包代码。

### module
  - NormalModule: 普通模块
  - ContextModule: ```'../src/xxx'```
  - ExternalModule: ```module.exports = jQuery```
  - DelegatedModule: mainfest
  - MultiModule: ```entry:[a,b]```

### Tapable
  - Tapable 是一个类似于node中的 EventEmitter 库，主要控制着钩子函数的发布和订阅，控制着webpack的插件系统
  - Tapable 使用
    * new Hook 创建钩子

### chunk 生成算法
  1. webpack 先将 entry 中对应的 module 都生成一个新的 chunk 
  2. 遍历 module 的依赖列表，将依赖的 module 也加入到 chunk 中 
  3. 如果一个依赖 module 是动态引入的模块，那么就会根据这个 module 创建一个 新的 chunk，继续遍历依赖 
  4. 重复上面的过程，直至得到所有的 chunks    

### 常见的几种模块方式
  - **es module** 
    * 加载方式：编译时确定模块之间的依赖关系，同步和异步都有
    * 运行环境： node，浏览器
    ```js
      import * as largeNumber from 'large-number';
      // ... largeNumber.add('999', '1'); 
    ```
  - **commonjs**
    * 加载方式：运行时加载，同步加载
    * 运行环境：nodejs
    ```js
      // 导出
      module.exports = {}
      // 导入
      const largeNumbers = require('large-number'); 
      // ... largeNumber.add('999', '1'); 
    ```
  - **amd**
    * 加载方式：运行时加载，异步加载
    * 推崇依赖前置
    * 运行环境：浏览器
    ```
      define([id], [dependencies], factory)
      require(['large-number'], function (large-number) { // ... largeNumber.add('999', '1'); })
    ```
  - **cmd**
    * 加载方式：运行时加载，同步和异步都有，最大的特点是懒加载  
    * 推崇就近依赖
    * 运行环境：浏览器
  - **umd**
    * 可以在多个环境中使用，结合了commonjs, amd 和 全局变量的导入导出方式
    * 运行环境： node，浏览器
### 模块化方式对比
  || ES Module | commonJs | Amd | Cmd |
  |:----| :----   | :----  | :----| :----|
  | 执行时机 | 编译时确定模块依赖关系，同步和异步都有 | 运行时加载，同步 | 运行时加载，异步 | 运行时加载，同步和异步都有 |
  | 输出 | 值的引用 | 值的拷贝 | 值的拷贝 | 值的拷贝 |

#### 参考文献：
  * https://juejin.cn/post/7128705370335215630/