# 网络安全-内容安全策略
## 概念：
  CSP 的实质是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于白名单。

  它的视线和执行全部由浏览器完成，开发者只需提供配置。

  内容安全策略的实现基于 HTTP 头部字段 ```Content-Security-Policy```.

## 启用 CSP 的两种方式
### 通过 HTTP 头部信息
  通过HTTP头部信息的 ```Content-Security-Policy``` 字段。
  ```http
    Content-Security-Policy: script-src 'self'; object-src 'none';style-src cdn.example.org third-party.org; child-src https:
  ```
### 通过 meta 标签
  ```js
    <meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">
  ```
**启用 CSP 之后，不符合 CSP 设置的资源就会被阻止加载**  

## CSP 选项
### 资源加载的限制
  - script-src: 外部脚本

    特殊值：
    * 'unsafe-inline': 允许执行页面内嵌的```&lt;script>标签和事件监听函数```
    * unsafe-eval: 允许将字符串当作代码执行，比如使用eval、setTimeout、setInterval 和 Function 函数
    * nonce值：每次 HTTP 响应会给出一个授权的 Token, 页面内嵌脚本必须有这个 Token, 才会执行
    * hash值：列出允许执行的脚本代码的 Hash值，页面内嵌脚本的 Hash值只有吻合的情况下才可以执行。
  - style-src: 样式表
  - img-src: 图像
  - media-src: 媒体文件（音频和视频）
  - font-src: 字体文件
  - object-src: 插件（比如 Flash）
  - child-src: 框架
  - frame-ancestors: 嵌入的外部资源（比如 ```<frame> <iframe> <embed> 和 <applet>```）
  - connect-src: HTTP 连接（通过 XHR、WebSockets、EventSource等）
  - worker-src: worker脚本
  - mainfest-src: mainfest文件

### default-src
  用来设置上面各个选项的默认值
  ```http
    // 限制所有的外部资源，只能从当前域名加载。
    Content-Security-Policy: default-src 'self'
  ```

### URL 限制
  当网页跟其他 URL 发生关系，这时也可以加以限制。

  - frame-ancestors: 限制嵌入框架的网页
  - base-uri: 限制 ```<base#href>```
  - form-action: 限制 ```<form#action>```

### 其他限制
  - block-all-mixed-content: HTTPS 网页不能加载 HTTP 资源（浏览器默认开启）
  - upgrade-insecure-requests: 自动将网页上加载的 HTTP 链接换成 HTTPS 协议
  - plugins-types: 限制可以使用的插件格式
  - sandbox: 浏览器行为的限制，比如不能弹出窗口等。

### report-uri
  希望不仅仅防止 XSS，还希望记录此类行为。

  report-uri 用来告诉浏览器，应该把注入行为报告给哪个网址。

  ```http
    // 将注入行为报告给 /my_amazing_csp_report_parser
    Content-Security-Policy: default-src 'self'; ...; report-uri /my_amazing_csp_report_parser;
  ```  

  浏览器会使用 POST 方法，发送一个 JSON 对象。

### 选项值
  - 主机名: example.org
  - 路径名: example.org/resources/js
  - 通配符: *://*.example.com:*
  - 协议名: https:、data
  - 关键字'self': 当前域名，需要加引号
  - 关键字'none': 禁止加载任何外部资源，需要加引号：

  **多个值可以并列，用空格分隔；如果同一个限制选项使用多次，只有一次会生效。**


## Content-Security-Policy-Report-Only
  表示不执行限制选项，只是记录违反限制的行为。

  它必须与 report-uri 选项配合使用。

## 注意事项
  - script-src 和 object-src 是必须设置的，除非设置了 default-src;
  - script-src 不能使用 unsafe-inline 关键字（除非伴随一个nonce值），也不能允许设置data:URL.
  - 必须特别注意 JSONP 的回调函数。
  ```js
    // 虽然加载的脚本属于当前域名，但是通过改写回调函数，攻击者依然可以执行恶意代码。
    <script src="/path/jsonp?callback=alert(document.domain)"></script>
  ```