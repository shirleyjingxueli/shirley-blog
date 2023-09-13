# 性能优化
### 性能衡量指标
  * 长任务：超过50ms的任务
  * TBT应该控制在300ms以内  
    https://web.dev/tbt/?utm_source=devtools 
  * FP:
  * FCP: 0 -- 1.8s -- 3s
  * LCP（视口中最大内容的加载完成时间）: 0 -- 2.5S -- 4S
  * FID: 0 -- 100ms -- 300ms
  * CLS: 0 -- 0.1 -- 0.25 分数越小


### 页面解析过程
前一个页面的卸载 -- dns解析 -- tcp建立 -- http请求（注意缓存）- 关键渲染路径（DOM + CSSOM = render树） -- 布局 -- 绘制


### 一. 包体积优化
#### code spliting
#### 公共资源分离
#### scopeHosting（webpack默认自带）
#### 资源压缩|使用特定格式
#### 剔除无用代码
  - js: treeShaking
  - css: purify css | uncss
#### 使用DeadCodePlugin
#### 动态polyfill
#### 利用 CDN 加速

  在构建过程中，将引用的静态资源路径修改为 CDN 上对应的地址。可以利用 webpack 对于 output 参数和各 loader 的 publicPath 来修改资源路径。


### 二. 资源存放
#### 存放到cdn上
  - 衡量资源是否加载过慢的标准：
    chrome devtool
  - 哪些资源适合存放到cdn？
    * 第三方类库
    * html，css，图片等静态资源  


### 三. 合理的缓存策略
#### 打包缓存
  - 文件名加hash
  - runtime 代码单独打包
    ```js
      {
        runtimeChunk: 'single',
      }
    ```
  - 第三方类库等进行```模块标识符```优化
    ```js
      optimization: {
        moduleIds: 'deterministic',  // 模块id摄者为deterministic
        runtimeChunk: 'single',
        splitChunks: {
          cacheGroups: {
            vendor: {
              test: /[\\/]node_modules[\\/]/,
              name: 'vendors',
              chunks: 'all',
            },
          },
        }
      }
    ```
#### 浏览器缓存（参考地址：https://juejin.cn/post/7071976605748297736）
  - 强制缓存
  - 协商缓存
#### 数据缓存 
  - cookie
  - webStorage: localStorage && sessionStorage
  - IndexedDB
  - Cache
  - Mainfest离线存储


### 四. preload、prefetch
#### 定义：
  - preload: 对当前页面需要的内容进行预加载
  - prefetch: 用来告诉浏览器在页面加载完成后，可以在空余时间加载的内容
#### 哪些资源需要preload？哪些资源需要prefetch
  - preload:
    * 页面初始化渲染所需要的js、css文件
    * 图片资源预加载
  - prefetch
    * dns-prefetch: 屏蔽dns解析延迟
    * vue-cli 中所有通过 async chunk 生成的文件，设置prefetch（可以删除）

### 五. dns解析时间过长？
  - 衡量标准：
    window.performance.timing
      .domainLookupStart
      .domainLookupEnd
  - 解决办法：
    - 使用cdn
    - 设置合理的TTL：较短的TTL使得DNS解析比较快
    - DNS缓存
    - 使用快速的DNS服务器
    - 合并域名：减少页面的域名，减少对DNS的解析次数
    - 预解析域名: rel="dns-prefetch" 或者 rel="perconnect"

### 六. 规范的代码编写
  - 动画
  - 图片
  - 长任务
  - 回流重绘

## 七. 常见问题解决方案
### 长任务的优化方式
1. Web Workers：网络工作者：

  使用 Web Workers 将长任务移到后台线程中运行，以避免阻塞主线程。这允许用户在执行长任务时继续与应用程序进行交互。
  Web Workers 可以并行处理任务，例如数据处理或密集计算。
  
2.  分批处理：

  将长任务拆分成较小的批次，逐步执行。这有助于防止主线程长时间被占用，从而提高应用的响应性。

  使用 requestAnimationFrame 或 setTimeout 将任务拆分成多个小步骤，以便让浏览器有时间渲染和响应用户输入。

3. 节流和防抖：

  对于用户输入或滚动等事件，使用节流和防抖技术来控制事件处理函数的频率，以减轻长任务对主线程的压力。
  节流确保一定时间内只执行一次函数，而防抖则在一定时间内只执行最后一次函数。

3. 虚拟化和懒加载：

  对于大型列表或表格等需要渲染大量数据的情况，使用虚拟化技术仅渲染可见区域内的元素，而不是全部渲染。

  使用懒加载加载图片或其他资源，只在它们进入可视区域时再加载。

4. 缓存和优化：

  针对网络请求和数据处理，使用缓存来减少不必要的重复计算或请求。

  优化数据库查询和算法，以提高数据检索和处理的效率。

5. 异步加载：

  使用异步加载来延迟加载不是立即必需的资源，例如在页面滚动到某个区域时再加载图片或其他内容。

6. 服务端渲染（SSR）：

  对于需要大量计算或数据处理的页面，考虑使用服务器端渲染来减轻客户端的负担，以便更快地呈现页面。

7. 代码拆分：

  使用代码拆分（Code Splitting）技术将应用程序分成更小的块，以便只加载和执行当前页面所需的代码，而不是全部代码。

8. 性能分析和监测：

  使用性能分析工具来识别长任务和性能瓶颈，例如浏览器的开发者工具或第三方工具。监测和记录应用的性能，以便及时发现和解决问题。

9. 延迟加载第三方库：

  仅在需要时加载第三方库，而不是在应用启动时全部加载。这可以通过动态导入（Dynamic Import）来实现。

10. 浏览器缓存：

  利用浏览器缓存来存储静态资源，以减少重复下载和加载。

11. 使用 WebAssembly：

  对于特别需要高性能的任务，可以考虑将计算部分移至 WebAssembly，以便以更高的性能运行。

#### 参考文献
  - https://juejin.cn/post/7025935054014513166
