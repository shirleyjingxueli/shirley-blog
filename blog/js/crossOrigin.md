# js-同源&跨域问题
## 同源策略
### 同源策略的要求
  - 协议相同
  - 域名相同
  - 端口相同
### 同源策略体现在哪几个方面
  - DOM 层面
  - Web 数据层面
  - 网络层面
#### DOM 层面
  同源策略限制了不同源的 JS 脚本对当前 DOM 对象的读取和操作。

  在同源策略中，可以通过 ```Window.opener ``` 来操作其他页面的 DOM

  ```window.opener``` 返回对打开窗口的窗口的引用，比如在A窗口中打开了B窗口，那么在B窗口中获取 ```window.opener``` 得到的就是对 A 窗口的引用。

  以下几种情况浏览器不会填充 ```window.opener```
  * 带有 ```rel="noopener"``` 属性
  * 链接的 target 为 "_blank", 除非明确指定 ```rel="opener"```
  * 设置了 Cross-Origin-Opener-Policy 标头，防止同源设置 opener。由于新窗口是在不同的浏览上下文中加载的，因此它不会引用打开的窗口。

#### Web 数据层面
  同源策略限制了不同源的站点读取当前站点的 Cookie, Web Storage, IndexedDB 等数据。

#### 网络层面
  同源策略限制了通过 XMLHttpRequest 等方式将站点的数据发送给不同源的站点。

### 同源策略下的安全和便利的权衡
  - 页面中可以嵌入第三方资源

    页面可以引入第三方资源，暴露了很多 XSS 的安全问题，因此在这种开放的基础上引入了 内容安全策略（CSP） 来限制其自由程度。

  - 跨域资源共享（CORS）和 跨文档消息机制

    跨域资源共享：对跨域访问进行控制，从而使数据传输安全的进行

    跨文档消息机制：解决了两个不同源的 DOM 之间无法进行通信的问题。可以通过  ```window.postMessage``` 的 JS 接口来和不同源的 DOM 进行通信。

### 同源策略，CSP， CORS之间的关系
  同源策略允许同源页面之间进行任何操作，不同源页面之间只能通过浏览器提供的方式来进行操作

  - 对 DOM 的读取和操作，要使用跨文档消息机制
  - 对于跨域请求要遵循 CORS 机制
  - 引用第三方资源要遵循 CSP 机制

## **跨域是什么**
  跨域是浏览器的一种安全策略,要求访问资源必须遵循协议相同，域名相同，端口相同, dom的操作，cookie, localstorage,indexDB,ajax请求都不允许跨域，script src, style的src是允许跨域的。

  跨域只存在于浏览器端
  
## **为什么不能跨域**
  不能跨域是为了保护本地资源不被js改掉，保证本地资源的安全性，防止xss，csrf攻击。

## **跨域的解决方案**
### jsonp
利用script src可以请求跨站资源的特性，像服务端发起一个带有callback参数的请求，服务端将响应的数据拼接成函数调用的方式，把请求结果放在callback参数里面传递给浏览器

缺点：jsonp只支持get请求，不支持post请求

```js
  /* 前端代码 */
  <script src="https://www.xxx.com:8080/api/xxx?callback=handleCrossCallback">
  </script>
  <script>
    function handleCrossCallback(res){
      // 此处的res为服务端放好的请求结果
      console.log(res)
    }
  </script>

  /* 服务端代码 */
  const http = require('http');
  const url = require('url');
  const querystring = require('querystring');
  const server = http.createServer();
  server.on('request', (req, res) => {
    const { query } = url.parse(req.url);
    const { callback } = querystring.parse(query);
    res.end(${callback}('hello'))
  });
  server.listen(8080, () => {
    console.log('8080端口监听中')
  })
```
### CORS 

（https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS）

- 简单请求：
  前端请求头自带 origin ，服务端响应头设置Access-Control-Allow-Origin: *；
  当响应的是附带身份凭证的请求时，服务端必须明确 Access-Control-Allow-Origin 的值，而不能使用通配符“*”。
  这种情况，前端请求头还需要设置 withCredentials，服务端响应头还需要设置Access-Control-Allow-Credentials: true
- 复杂请求：
例子：比如服务端采用的是token机制，前端就需要将token放到请求头中，将content-type设置为application/json.
### nginx反向代理（原理：服务器无跨域说法）
  反向代理指的是不管内部还是外部网络都能访问内部网络数据。

  反向代理：客户端 --- 代理服务器(和页面在同一个源) --- 源服务器；客户端直接访问代理服务器，代理服务器访问源服务器获取数据，并返回给客户端。

  ```conf
  server {
    listen 8080
    server_name localhost
    #localhost:8080/api/*** 会被转发到这里，同时服务端会接收到https://192.168.1.2:9000/api/xxx的请求
    location /api/ {
      proxy_pass: https://192.168.1.2:9000
    }
    #localhost:8080/api2/*** 会被转发到这里，同时服务端会接收到https://192.168.1.2:8000/api2/xxx的请求
    /api2/ {
        proxy_pass: https://192.168.1.2:8000
    }
  }
  ```
### websocket
  原理：websocket是html5的一个持久化的协议，实现了浏览器和服务器的全双工通信，同时也是跨域的一种解决方案。
  websocket 和http都属于应用层协议，都是基于tcp协议

  但是websocket是一种双向通信协议，在建立链接之后，websocket 的服务器和客户端都能主动向对方发送或者接收数据。
  websocket在建立连接时需要借助http协议，链接建立好了以后就服务端和客户端的通信就和http协议无关

### postMessage
调用postMessage方法可以实现父子窗口互相发送消息。

它可以用于解决以下问题：
  * 页面和其打开窗口的数据传递
  * 多窗口之间消息传递
  * 页面与嵌套的iframe消息传递
  * 上面三个场景的跨域数据传递

```js
  var url = 'https://xxxxx'
  var openWindow = window.open(url);
  // 发送message
  openWindow.postMessage('hello world', url)

  // 监听发送的消息
  window.addEventListener('messge', function(e) {
    console.log(e.source || e.origin || e.data || xxxxx)
  })
```

### webpack, vue-cli, react-cli等生态中提供的
  ```js
    devServer: {
      port: 8080,
      proxy: {
        '^/api': {
          target: "https://192.168.2.3:9000"
        }
      }
    }
  ```
### document.domain
  因为浏览器通过domain来判断是否是同源，可以将document.domain设置成同一个，两个页面就可以共享cookie（此方案仅适用于主域名相同，子域名不同的跨域场景）