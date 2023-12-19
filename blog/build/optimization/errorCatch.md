# 性能优化-错误监控

  错误监控的一般步骤：
  1. 分析错误类型
  2. 捕获错误
  3. 错误格式化
  4. 错误上传
  5. 分析错误

## 错误类型
### JS 错误
  - **Error**: 基类，其他的错误类型继承该类型。浏览器很少抛出 Error 类型的错误，该类型主要用于开发者抛出自定义错误。
  - **InternalError**: 底层 JavaScript 引擎抛出异常时浏览器抛出。比如：递归多导致了栈溢出。
  - **EvalError**: 使用 eval() 函数发生异常时抛出。基本上，只要不把 eval() 当成函数调用就会报告该错误。
  - **RangeError**: 在数值越界时抛出。
  - **ReferenceError**: 会在找不到对象时抛出。
  - **SyntaxError**: 在给 eval() 传入的字符串包含 JavaScript 语法错误时发生。
  - **TypeError**: 主要发生在变量不是预期类型，或者访问不存在的方法时抛出。
  - **URIError**: 在使用 encodeURI 或者 decodeURI() 时传入了格式错误的 URI。

### 网络错误
  - **ResourceError**: 资源加载错误
  - **HttpError**: Http 请求错误

### promise 异常

### 跨域脚本执行异常

### 捕获错误和抛出错误的区别
  应该旨在确切知道接下来该做什么的时候捕获错误。

  捕获错误的目的是阻止浏览器以其默认方式响应；抛出错误的目的是为错误提供有关其发生原因的说明。

## 错误处理策略
### 识别错误
  通常需要注意 3 类错误：
  - 类型转换错误
  - 数据类型错误
  - 通信错误

#### 解决办法：
  1. **静态代码分析器。** 通过在代码构建流程中添加静态代码分析器或者代码检查器，可以预先发现非常多的错误。常用的静态分析工具有JSHint, JSLint, Google Closure, TypeScript.
  2. **良好的代码书写风格。**类型转换错误常见于使用了会自动转换某个值的数据类型操作符或者语言构造。比如使用（==）或者 （!=）操作符，以及在if、for 或者 while 等控制语句中使用非布尔值。对于这种错误应尽可能使用严格相等和严格不相等操作符，流程控制语句中尽可能使用布尔值
  3. **encodeURIComponent。** 对于通信错误中出现的 URL 格式或者发送数据的格式不正确的问题，可以通过 encodeURIComponent() 对 url 进行编码。

### 区分重大与非重大错误
#### 非重大错误：

  具有以下一个或者多个特性的错误属于非重大错误：

  - 不会影响用户的主要任务；
  - 只会影响页面中的某个部分
  - 可以恢复
  - 重复操作可能成功。

  **对于非重大错误，无需明确给用户发送消息。可以将受影响的页面区域替换成一条消息，表示该功能暂时不能使用，但不需要中断用户体验。**

#### 重大错误：
  重大错误具备的特征：
  
  - 应用程序无法继续运行
  - 错误严重影响了用户的主要目标
  - 会导致其他错误发生。

  **当重大错误发生时，应该立即发送消息让用户知晓自己不能继续使用应用程序了。如果必须刷新页面才能恢复应用程序，那就应该明确告知用户，并提供一个自动刷新页面的按钮。**

### 把错误记录到服务器中
  对于复杂的 web 应用程序而言，最好把 JavaScript 错误发送回服务器记录下来。

  要建立 JavaScript 错误日志系统，首先需要再服务器上有页面入口可以处理错误数据。该页面只要从查询字符串中取得错误数据，然后把它们保存到错误日志中即可。

### 分析错误
#### SourceMap
  利用 webpack 的 hidden-source-map 构建。 与 source-map 相比少了末尾的注释，但 output 目录下的 index.js.map 没有少。 线上环境避免 source-map 泄露。
### 错误警报
  可以接入到邮箱、钉钉、slack等工作平台。

## 错误捕获具体方法
### try/catch
  **try/catch 语句最好用在自己无法控制的错误上。**比如：使用了某个大型 JS 库中的一个函数，这个函数可能会发生错误，但是不能修改这个库的代码，此时，可以用 try/catch 把函数调用包裹起来，对可能的错误进行处理。

  **注意：只要代码中包含了 finally 子句，try 块或者 catch 块中的 return 语句就会被忽略。**

  **注意：try/catch 只能捕获到同步的错误，无法捕获异步错误。**

### throw
  **throw 操作符用于在任何时候抛出自定义错误。throw 操作符必须有一个值，但是值的类型不限。**

  使用 throw 操作符时，代码会立即停止执行，除非 try/catch 语句捕获了跑出的值。

### 监听 window.onerror
  任何没有被 try/catch 语句处理的错误都会在 window 对象上触发 error 事件。

  在 error 事件处理程序中，任何浏览器都不会传入 event 对象。 会传入三个参数： 错误消息、发生错误的 URL 和 行号。

  ```js
    window.onerror = (message, url, line) => {
      console.log(message);
      return false; // 通过 return false 来阻止浏览器默认报告错误的行为。
    }
  ```
  **window.onerror 函数只有在返回 true 时，异常才不会向上抛出，否则即使是知道异常的发生，控制台还是会显示 Uncaught Error: xxxxxx**
 
  **当一项资源（如图片或者脚本）加载失败时，加载资源的元素会触发一个 Event 接口的 error 事件，并执行该元素上的 onerror() 处理函数。这些 error 事件不会冒泡到 window 上。**

  **对于 iframe 的异常捕获，也可以借助于 window.onerror**,比如：

  ```js
    <iframe src="./iframe.html" frameborder="0"></iframe> 
    
    <script>
      window.frames[0].onerror = function(message, source, lineno, colno, error) {
        console.log("捕获到 iframe 异常"，{message, souirce, lineno, colno, error});
        return true;
      }
    </script>
  ```

### window.addEventListener("error")
  对于请求不到资源或者静态资源失效的报错，可以使用 ```window.addEventListener("error")```来监听报错信息。

  使用 addEventListener 捕获资源错误时，一定要将 第三个选项设为true，因为资源错误没有冒泡，所以只能在捕获阶段。同理，由于 window.onerror 是通过在冒泡阶段捕获错误，所以无法捕获资源错误。

### unhandledrejection 事件
  为了防止有漏掉的 promise 异常，可以通过 unhandledrejection 来全局监听 Uncaught promise error

####  promise 异常漏掉的常见场景：

  - 只使用了 try/catch 捕获异常，但是 try/catch 只能捕获到同步代码的异常，无法捕获异步代码的异常
  - promise 内部为使用 catch() 方法进行捕获异常。**在使用 promise 时，当 promise 被 reject 且并没有被 catch 处理的时候，就会抛出 promise 异常；同样的，在使用 promise 的过程中，报了 JS 错误，同样也被以 Promise异常的形式抛出。**

  因为抛出 promise 异常时，会触发 unhandledrejection 事件，所以可以通过监听 unhandledrejection 事件来捕获 promise 中未处理的错误。
  比如：

  ```js
    // 捕获不到异常
    try {
      new Promise((resolve) => {
        JSON.parse("")
        resolve();
      })
    } catch(e){
      // 这里捕获不到异常
      console.log("error in try", e)
    }

    // 可以捕获到异常
     try {
      new Promise((resolve) => {
        JSON.parse("")
        resolve();
      }).catch(e => {
        // 在这里可以捕获到异常
        console.log("error in promise", e)
      })
    } catch(e){
      // 这里捕获不到异常
      console.log("error in try", e)
    }

  ```

#### 解决办法：

```js
  window.addEventListener("unhandledrejection", function(e) {
    console.log("全局监听未捕获到的promise错误：", e)
  })
```
### 跨域脚本错误：script error
  由于我们一般会将静态资源存放在 cdn 等第三方域名上，所以当前业务域名中的 window.onerror 会将这类错误同一展示为 script error

#### 解决方案
  - 后端配置 Access-Control-Allow-Origin, 前端在 script 标签配置 cross origin
  - 劫持原生方法，使用 try/catch 绕过，将错误抛出。  

    ```js
    const originAddEventListener = EventTarget.prototype.addEventListener;
    EventTarget.prototype.addEventListener = function (type, listener, options) {
      const wrappedListener = function (...args) {
        try {
          return listener.apply(this, args);
        }
        catch (err) {
          throw err;
        }
      }
      return originAddEventListener.call(this, type, wrappedListener, options);
    }

    ```
  - 动态加载脚本

  比如：

  ```js
    // 动态加载脚本
    const script = document.createElement("script");
    script.crossOrigin = "anonymous";
    script.src= url;
    document.body.appendChild(script);
  ```

### 网络错误
  针对网络错误，项目中一般会使用像 axios 这样子的库，可以使用 interceptor 拦截器来预处理 request 和 response， 所以我们在这里可以进行统一的网络错误拦截

  ```js
    import axios from "axios";

    axios.interceptors.response.use(
      response => response,
      error => {
        // 拦截网络错误
        processErrorInfo(error);
        return Promise.reject(error);
      }
    )
  ```

### 网页崩溃或者卡顿
  1. 利用 window 对象的 load 和 beforeunload 事件实现对网页崩溃的监控

  ```js
    window.addEventListener('load', function () {
      sessionStorage.setItem('good_exit', 'pending');
      setInterval(function () {
          sessionStorage.setItem('time_before_crash', new Date().toString());
      }, 1000);
    });

    window.addEventListener('beforeunload', function () {
      sessionStorage.setItem('good_exit', 'true');
    });

    if(sessionStorage.getItem('good_exit') &&
      sessionStorage.getItem('good_exit') !== 'true') {
      /*
          insert crash logging code here
      */
      alert('Hey, welcome back from your crash, looks like you crashed on: ' + sessionStorage.getItem('time_before_crash'));
    }
  ```
  **为什么使用 sessionStorage，而不是 localStorage？**

  **因为 localStorage 在 同一域的选项卡之间是共享的。如果您打开了此页面的两个不同选项卡，并且 tab1 将崩溃，并且您切换到 tab2 并且成功刷新页面，则 tab1 上的崩溃信息将丢失。**

  **sessionStorage 是与每个会话隔离的存储，它们不会在选项卡之间共享，这使得他们在记录有关选项卡中发生的事情的信息时更理想。**

2. 使用 Service Worker 来实现网页崩溃监控

  使用 Service Worker 来监控网页崩溃的原因：

  - Service Worker 有自己独立的工作线程，与网页分开，网页崩溃了，Service Worker 一般情况下不会崩溃。
  - Service Worker 生命周期一般比网页还要长，可以用来监控网页的状态
  - 网页可以通过 ``` navigator.serviceWorker.controller.postMessage ``` API 向掌管自己的 SW 发送消息。

### vue 错误处理
#### 全局错误处理
- app.config.errorHandler
- app.config.warnHanlder

```js
  app.config.errorHandler = (err, instance, info) => {
    
  }
```

**errorHandler 可以从下面这些来源中捕获错误：**
  - 组件渲染器
  - 事件处理器
  - 生命周期钩子
  - setup() 函数
  - 侦听器
  - 自定义指令钩子
  - 过渡(Transition)钩子
#### errorCaptured
  用于在父组件中处理子孙组件的错误。
    
  另外，```<Suspense>``` 组件自身目前还不提供错误处理，不过你可以使用 errorCaptured 选项或者 onErrorCaptured() 钩子，在使用到 ``` <Suspense> ``` 的父组件中捕获和处理异步错误。
    
#### renderTracked
  renderError 和组件相关，只适用于非生产环境

### react 错误处理

## 基于 sentry 的错误监控
### sentry 原理
  sentry 实现了异常详情获取、异常自动推送、用户行为获取。
#### 异常详情获取
  - 劫持重写了 window.onerror 和 window.unhandledrejection 两个 api
  - 内部对异常放生的特殊上下文，做了标记。这些特殊上下文包括：DOM 节点事件回调、setTimeout/setInterval 回调、xhr 接口调用、requestAnimationFrame 回调等
    
    具体步骤为：
    - 标记 setTimeout / setInterval / requestAnimationFrame
    - 为了标记 setTimeout / setInterval / requestAnimationFrame 类型的异常，Sentry 劫持覆写了原生的 setTimout / setInterval / requestAnimationFrame 方法。新的 setTimeout / setInterval / requestAnimationFrame 方法调用时，会使用 try ... catch 语句块包裹 callback。
    - 当 callback 内部发生异常时，会被 catch 捕获，捕获的异常会标记 setTimeout。
    - setInterval、requestAnimationFrame 的劫持覆写逻辑和 setTimeout 基本一样
    - 标记 DOM 事件 handler
    - 所有的 DOM 节点都继承自 window.Node 对象，DOM 对象的 addEventListener 方法来自 Node 的 prototype 对象。
    - 为了标记 DOM 事件 handler，Sentry 对 Node.prototype.addEventListener 进行了劫持覆写。新的 addEventListener 方法调用时，同样会使用 try ... catch 语句块包裹传入的 handler。
    - 当 handler 内部发生异常时，会被 catch 捕获，捕获的异常会被标记 handleEvent, 并携带 event name、event target 等信息。

    - 其实，除了标记 dom 事件回调上下文，Sentry 还可以标记 Notification、WebSocket、XMLHttpRequest 等对象的事件回调上下文。可以这么说，只要一个对象有 addEventListener 方法并且可以被劫持覆写，那么对应的回调上下文会可以被标记。

    - 标记 xhr 接口回调

      为了标记 xhr 接口回调，Sentry 先对 XMLHttpRequest.prototype.send 方法劫持覆写, 等 xhr 实例使用覆写以后的 send 方法时，再对 xhr 对象的 onload、onerror、onprogress、onreadystatechange 方法进行了劫持覆写, 使用 try ... catch 语句块包裹传入的 callback。

      当 callback 内部发生异常时，会被 catch 捕获，捕获的异常会被标记对应的请求阶段。

#### 用户行为获取
  常见的用户行为可以归纳为页面跳转，鼠标 click 行为、键盘 keypress 行为、fetch/xhr 接口请求、console 打印信息。

  sentry 接入应用后，会在用户使用应用过程中，将上述行为一一收集起来。等到捕获到异常时，会将收集到的用户行为和异常信息一起上报。

  **具体实现过程如下：**
  
  - **收集页面跳转行为：**

    劫持重写原生 history 的 pushState, replaceState 方法 和 window 的 onpopstate 事件。

  - **收集鼠标 click / 键盘 keypress 行为：**

    为了收集用户鼠标 click 以及键盘 keypress 行为，sentry 做了双保险操作： 通过 document 代理 click、keypress 事件来收集 click、keypress 行为；通过劫持 addEventListener 方法来收集 click、keypress 行为。

    **详细步骤：**

    - 首先， Sentry 使用 document 代理了 click、keypress 事件。通过这种方式，用户的 click、keypress 行为可以被感知，然后被 Sentry 收集。

    - 但这种方式有一个问题，如果应用的 dom 节点是通过 addEventListener 注册了 click、keypress 事件，并且在事件回调中做了阻止事件冒泡的操作，那么就无法通过代理的方式监控到 click、keypress 事件了。

    - 针对这一种情况， Sentry 采用了覆写Node.prototype.addEventListener 的方式来监控用户的 click、keypress 行为。

    - 由于所有的 dom 节点都继承自 Node 对象，Sentry 劫持覆写了Node.prototype.addEventListener。当应用代码通过 addEventListener 订阅事件时，会使用覆写以后的 addEventListener 方法。

    - 新的 addEventListener 方法，内部里面也有很巧妙的实现。如果不是 click、keypress 事件，会直接使用原生的 addEventListener 方法注册应用提供的 listener。但如果是 click、keypress 事件，除了使用原生的 addEventListener 方法注册应用提供的 listener 外，还使用原生 addEventListener 注册了一个 handler，这个 handler 执行的时候会将用户 click、keypress 行为收集起来。

    - 也就是说，如果是 click、keypress 事件，应用程序在调用 addEventListener 的时候，实际上是调用了两次原生的 addEventListener。

    - 另外，在收集 click、keypress 行为时，Sentry 还会把 target 节点的的父节点信息收集起来，帮助我们快速定位节点位置

  - **收集 fetch/xhr 接口请求：**

    对原生的 fetch/xhr 进行了劫持重写。

    **具体步骤：**

    - Sentry 是通过劫持覆写 XMLHttpRequest 原型上的 open、send 方法的方式来实现收集接口请求行为的。

    - 当应用代码中调用 open 方法时，实际使用的是覆写以后的 open 方法。在新的 open 方法内部，又覆写了 onreadystatechange，这样就可以收集到接口请求返回的结果。新的 open 方法内部会使用调用原生的 open 方法。

    - 同样的，当应用代码中调用 send 方法时，实际使用的是覆写以后的 send 方法。新的 send 方法内部先收集接口调用信息，然后调用原生的 send 方法。

  - **收集 console 打印行为：**

    对 console的 debug、info、warn、error、log、assert 这几个 api 进行劫持重写。

### 在 vue 中接入 sentry
  **sentry 除了可以用作 bug 追踪以外，sentry 还可以用来做性能监控。**
#### 具体步骤：
  - 安装 sentry-webpack-plugin
  - 在 config 中配置, 在生产环境打包的时候需要把 sourcemap 文件上传到 sentry
  ```js
    const { sentryWebpackPlugin } = require("@sentry/webpack-plugin");

    Plugins.push(
      new SentryCliPlugin({
        url: 'xxx',
        org: 'xxx',
        project: process.env[`SENTRY_PROJECT_${sentryEnv}`],
        authToken: process.env[`SENTRY_UPLOAD_KEY_${sentryEnv}`],
        release,
        include: './dist',
        ignore: ['node_modules'],
        ignoreFile: '.gitignore',
        cleanArtifacts: true,
        urlPrefix,
        sourceMapReference: true,
        errorHandler: (err, invokeErr, compilation) => {
          compilation.warnings.push('Sentry CLI Plugin: ' + err.message);
        },
      }),
      new DeleteSourcemapPlugin(), // 需要在处理完成之后删除 sourcemap
    );
  ```
  - 在 main.js 中初始化 sentry
    ```js
      import * as Sentry from "@sentry/vue";

      Sentry.init({
        app,
        dsn: "https://examplePublicKey@o0.ingest.sentry.io/0",
        integrations: [
          new Sentry.BrowserTracing({
            routingInstrumentation: Sentry.vueRouterInstrumentation(router),
          }),
          new Sentry.Replay(),
        ],

        // Set tracesSampleRate to 1.0 to capture 100%
        // of transactions for performance monitoring.
        // We recommend adjusting this value in production
        tracesSampleRate: 1.0,

        // Set `tracePropagationTargets` to control for which URLs distributed tracing should be enabled
        tracePropagationTargets: ["localhost", /^https:\/\/yourserver\.io\/api/],

        // Capture Replay for 10% of all sessions,
        // plus for 100% of sessions with an error
        replaysSessionSampleRate: 0.1,
        replaysOnErrorSampleRate: 1.0,
      });
    ```
  - 在需要的场景下进行上报
    ```js
      import { captureException } from '@sentry/browser';

      captureException(err);
    ```  
  - 识别用户
    可以在 store 中设置用户身份

    ```js
      Sentry.setUser(null);
    ```  
  - 查看数据并解决问题

## 总结：
  开发 Web 应用程序时，应该认真考虑可能发生的错误，以及如何处理这些错误。

  首先应该学会识别和区分重大和非重大错误，通过静态代码检测预测可能发生的错误，比如：类型转换错误，数据类型错误，向服务器发送错误的数据或者从服务器接收到错误数据。

  可以通过 try/catch，throw, 监听 onerror 事件等具体的方法来更准确的追踪与报告错误。

  对于网络错误，可以使用像 axios 的 interceptor 进行统一处理

  对于页面级别的崩溃可以使用 window.load, window.unload 和 sessionStorage 结合来处理，也可以使用 serviceWorker 来处理。

## 参考文献
1. [SourceMap 与前端异常监控](https://mp.weixin.qq.com/s/BbvJ-OfcS7Sa-e0Zq6iF1w)
2. [Sentry 捕获异常原理](https://www.51cto.com/article/740742.html)

## 技术选型需要考虑什么问题
## nuxt在项目中是怎么使用的
## lerna在项目中是怎么使用的
## 为什么选择uniapp
## CICD主要做了哪些工作
## 项目分支管理