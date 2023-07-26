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

#### 参考文献
  - https://juejin.cn/post/7025935054014513166
