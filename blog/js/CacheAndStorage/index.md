# js-浏览器缓存和存储
## 缓存分类
  - Memory Cache
  - Service Worker Cache
  - HTTP Cache(也叫做Disk Cache)
  - Push Cache

### Memory Cache
  缓存在内存中。
  从优先级上讲，是浏览器最先尝试去命中的一种缓存。
  从效率上讲，它是响应速度最快的一种缓存。

#### 特性
  和渲染进程"生死相依"，进程结束之后，也就是tab关闭以后，内存中的数据就不复存在了。

#### 哪些文件会被放在内存中
  - 几乎所有的base64图片
  - 较小的js、css文件

### Service Worker Cache
  Service Worker 是一种独立于主线程之外的js线程。脱离于浏览器窗口，无法访问DOM。所以 Service Worker无法干预页面的性能。可以用来实现离线缓存、消息推送、网络代理等功能。

  Service Worker 包括三个阶段： install、active、working 三个阶段。

  Service Worker 必须以https协议作为前提。

### HTTP Cache（Disk Cache）
  存储在硬盘中，读取的速度稍微慢点。相比Memory Cache胜在容量和存储时效上。

  它是持久存储的，是实际存在于文件系统中的。而且它允许相同的资源在跨会话，甚至跨站点的情况下使用，例如两个站点都使用了同一张图片。

  在所有浏览器缓存中，Disk Cache的覆盖面是最大的。它会根据HTTP Header中的字段判断哪些资源需要缓存，哪些资源不请求可以直接使用，哪些资源已经过期需要重新请求。
  

#### 哪些资源会被放到硬盘中
  - 大文件
  - 系统内存使用率高的话，文件优先存储进硬盘。
#### 分类  
  - 强制缓存
    * 第三方库和基础模块
    * 图片、字体等静态资源
  - 协商缓存
    * HTML、JS、CSS等文本文件
    * 动态生成的内容或者接口结果
#### 强制缓存
  cache-control: max-age=xxx; s-max-age=xxx;

  缓存请求指令：
  - no cache
  - no store
  - no-transform
  - only-if-cached：表明客户端只接受已缓存的响应，并且不要向原始服务器检查是否有更新的拷贝。
  - ```max-age=<seconds>```
  - ```max-stale=[<seconds>]```：客户端愿意接受一个已经过期的资源。
  - ```min-fresh=<seconds>```：客户端希望获取一个能在指定秒数内保持其最新状态的响应。

缓存响应指令：
  - must-revalidate：一旦资源过期（比如已经超过max-age），在成功向原始服务器验证之前，缓存不能用该资源响应后续请求。
  - no-cache：在发布缓存副本之前，强制要求缓存把请求数据提交给原始服务器进行验证（**协商缓存验证**）
  - no-store：缓存不应存储有关客户端请求或者服务器响应的任何内容，**即不使用任何缓存**
  - no-transform：不得对资源进行转换或转变。Content-Encoding、Content-Range、Content-Type等 HTTP 头不能由代理修改。
  - public: 可以被任何对象缓存（包括：发送客户端、代理服务器等等）
  - private：只能被单个用户缓存
  - proxy-revalidate：与 must-revalidate 作用相同，但它仅适用于共享缓存（例如代理），并被私有缓存忽略。
  - ```max-age=<seconds>```：缓存的最大周期
  - ```s-maxage=<seconds>```：仅适用于共享缓存，私有缓存会忽略它。会覆盖 max-age 和 Expires 头

#### 协商缓存
  if-none-match/E-tag;

  if-modified-since/last-modified

### Push Cache
  HTTP2 在server push阶段存在的缓存。

#### 特性
  - Push Cache 是缓存的最后一道防线。浏览器只有在 Memory Cache、HTTP Cache 和 Service Worker Cache 均未命中的情况下才会去询问 Push Cache。
  - Push Cache 是一种存在于会话阶段的缓存，当 session 终止时，缓存也随之释放。
  - 不同的页面只要共享了同一个 HTTP2 连接，那么它们就可以共享同一个 Push Cache。

## 缓存应用场景
### 频繁变动的资源
  ```
    Cache-Control: no-cache
  ```
  对于频繁变动的资源，首先需要使用Cache-Control: no-cache 使浏览器每次都请求服务器，然后配合 ETag 或者 Last-Modified 来验证资源是否有效。这样的做法虽然不能节省请求数量，但是能显著减少响应数据大小。

### 不常变动的资源
```
  Cache-Control: max-age=31536000
```
  通常在处理这类资源时，给它们的 Cache-Control 配置一个很大的 max-age=31536000 (一年)，这样浏览器之后请求相同的 URL 会命中强制缓存。而为了解决更新的问题，就需要在文件名(或者路径)中添加 hash， 版本号等动态字符，之后更改动态字符，从而达到更改引用 URL 的目的，让之前的强制缓存失效 (其实并未立即失效，只是不再使用了而已)。

  在线提供的类库 (如 jquery-3.3.1.min.js, lodash.min.js 等) 均采用这个模式。

## 用户行为对浏览器缓存的影响
  所谓用户行为对浏览器缓存的影响，指的就是用户在浏览器如何操作时，会触发怎样的缓存策略。主要有 3 种：

  - 打开网页，地址栏输入地址： 查找 disk cache 中是否有匹配。如有则使用；如没有则发送网络请求。
  - 普通刷新 (F5)：因为 TAB 并没有关闭，因此 memory cache 是可用的，会被优先使用(如果匹配的话)。其次才是 disk cache。
  - 强制刷新 (Ctrl + F5)：浏览器不使用缓存，因此发送的请求头部均带有 Cache-control: no-cache(为了兼容，还带了 Pragma: no-cache),服务器直接返回 200 和最新内容。

## 缓存面试题
#### 强制缓存和协商缓存的使用场景。
[见链接](#缓存应用场景)
#### 浏览器的缓存策略？max-age,no-cache, no-store区别？
  指定 ```no-cache``` 或 ```max-age=0, must-revalidate``` 表示客户端可以缓存资源，每次使用缓存资源前都必须重新验证其有效性。这意味着每次都会发起 HTTP 请求，但当缓存内容仍有效时可以跳过 HTTP 响应体的下载。
  ```HTTP
    Cache-control: no-cache
  ```
  等同于
  ```HTTP
    Cache-control: max-age:0, must-revalidate
  ```
  
  如果服务器关闭或者失去连接，下面的指令可能会造成使用缓存

  ```HTTP
    Cache-control: max-age=0;
  ```

#### html，css, js 分别设置什么缓存
  协商缓存：html，css，js

  强制缓存：第三方库，图片，字体
## 缓存-参考链接
  https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/53

  https://github.com/Adamwu1992/adamwu1992.github.io/issues/12

  https://www.jianshu.com/p/54cc04190252

## 本地存储分类
  - Cookie
  - webStorage
  - IndexedDB

### Cookie
  用于存储客户端的会话信息。
  
  这个规范要求服务器在相应HTTP请求时，通过发送 Set-Cookie HTTP响应头部包含会话信息。

  浏览器会存储这些会话信息，并在之后的每个请求中都会通过 HTTP 请求头部 cookie 再将这些信息发送给服务端。

#### 特性
  - 每次请求都会携带
  - 有限制条件：
    * 不超过 300 个cookie
    * 每个 cookie 不超过 4kb
    * 每个域不超过 20 个cookie
    * 每个域的cookie 不超过 80kb
    * 不同浏览器对于每个域能设置的 cookie 总数的限制是不一样的。 safari和chrome对每个域的 cookie数没有硬性要求
    * 浏览器也会限制 cookie 的大小。大多数浏览器对 cookie 的限制是不超过 4096 字节，上下可以有一个字节的误差。为跨浏览器兼容，最好保证 cookie 的大小不超过 4095 字节。**这个大小限制适用于一个域的所有 cookie，而不是单个 cookie。**
    * 如果 cookie 总数超过了单个域的上限，浏览器会删除之前设置的cookie
    * 如果创建的 cookie 超过最大限制，则该 cookie 会被静默删除
  - 操作较为复杂
  - 是与特定的域绑定的，不允许跨域

#### 使用方法
  ```js
    document.cookie

    //  因为所有名和值都是URL编码的，因此必须使用DecodeURIComponent() 解码

  ```
#### HTTP-only
  有一种叫做HTTP-only的cookie。HTTP-only 可以在浏览器设置，也可以在服务器设置，但只能在服务器上读取，因为js无法获取这种cookie的值。

### Web Storage
  Web Storage 的目的是解决通过客户端存储不需要频繁发送回服务端的数据时使用cookie的问题。
  
  只能存储字符串。

  不受页面刷新影响。

  所有现代浏览器在实现存储写入时都使用了同步阻塞的方式，因此数据会被立即提交到存储。**通过Web Storage 写入的任何数据都可以立即被读取。**

#### 目的
  - 提供在 cookie 之外的存储会话数据的途径。
  - 提供跨会话持久化存储大量数据的机制。

#### 特性
  * 使用key-value形式存储；使用方便
  * 大小有5MB
  * key 和 value以字符串的形式存储

#### 使用方法：
  - clear()
  - getItem(name)
  - key(index)
  - removeItem(name)
  - setItem(name)

#### webStorage分类
  - localStorage: 永久存储机制
    * 不受页面刷新影响
    * 要访问同一个localStorage对象，页面必须来自同一个域（子域不可以），在相同的端口上使用相同的协议。
  - sessionStorage: 跨会话存储机制
    * 会话窗口关闭后，会删除数据。
    * 不受页面刷新影响，可以在浏览器崩溃重启后恢复。

  对于 sessionStorage 和 localStorage 上的任何更改都会触发 storage 事件，但 storage 事件不会区分这两者。

  事件对象有4个属性：
  - domain
  - key
  - newValue
  - oldValue

  ```js
    window.addEventListener("storage", (event) => alert("storage changed for ${event.domain}"))
  ```
#### 限制
  - 客户端数据的大小限制是按照每个源（协议、域和端口）来设置的，因此每个源有固定大小的数据存储空间。
  - 不同浏览器给 localStorage 和 sessionStorage 设置了不同的空间限制，但大多数会限制为每个源 5MB。

### IndexedDB
  是浏览器存储结构化数据的一种方案。

#### 目的
  创建一套API，方便js对象的存储和获取，同时也支持查询和搜索。

#### 特性
  - 使用对象存储存储数据
  - 异步API
  - 存储量没有上限（chrome的存储空间定义是：硬盘可用空间的三分之一）
  - 所有的操作都是异步的，相比 localStorage 同步操作性能更高，尤其是数据量比较大时
  - 原生支持存储js对象
  - 功能强大，数据库能做的事情都能做

#### 使用方法
  - 打开要打开的数据库：indexedDB.open()
  ```js
    let db, 
    request, 
    version = 1; //注意版本号不要使用小数，而要使用整数

    request = indexedDB.open("admin", version); 

    request.onerror = (event) => 
      alert(`Failed to open: ${event.target.errorCode}`); 
    request.onsuccess = (event) => { 
      db = event.target.result; 
    };
  ```
  - 存储对象
  - 操作事务：transaction()
  - 插入对象：add()、put()
  - 查询方法：openCursor()
  - 键范围：IDBKeyRange

#### 限制：
  - 同源，信息不能跨域共享。
  - 每个源都有可以存储的空间限制。（chrome是5MB）

## Storage小结
  Web Storage 定义了两个对象用于存储数据：sessionStorage 和 localStorage。前者用于严格保存浏览器一次会话期间的数据，因为数据会在浏览器关闭时被删除。后者用于会话之外持久保存数据。

  IndexedDB 是类似于 SQL 数据库的结构化数据存储机制。不同的是，IndexedDB 存储的是对象，而不是数据表。对象存储是通过定义键然后添加数据来创建的。游标用于查询对象存储中的特定数据，而索引可以针对特定属性实现更快的查询。

  有了这些存储手段，就可以在客户端通过使用 JavaScript 存储可观的数据。因为这些数据没有加密，所以要注意不能使用它们存储敏感信息。

## Storage面试题
#### localStorage如何设置过期时间
  - 手动管理过期时间： 
    * 存储时，存储一个时间戳，表示数据的过期时间。
    * 读取时，检查时间戳是否已经过期，如果过期则删除该数据。
  - 使用第三方库
    * store.js
    * localForage.js   
