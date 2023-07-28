# js-事件循环机制
## js中的EventLoop
### 背景
  js最初设计的是单线程的。

  自从定时器（setTimeout() 和 setInterval()）加入到 Web API 后，浏览器提供的 JavaScript 环境就已经逐渐发展到包含任务调度、多线程应用开发等强大的特性。

  在执行 JavaScript 代码的时候，JavaScript 运行时实际上维护了一组用于执行 JavaScript 代码的代理。每个代理由一组执行上下文的集合、执行上下文栈、主线程、一组可能创建用于执行 worker 的额外的线程集合、一个任务队列以及一个微任务队列构成。除了主线程（某些浏览器在多个代理之间共享的主线程）之外，其他组成部分对该代理都是唯一的。
### 简介
  **事件循环负责收集事件（包括用户事件以及其他非用户事件等）、对任务进行排队以便在合适的时候执行回调。然后它执行所有处于等待中的 JavaScript 任务，然后是微任务，然后在开始下一次循环之前执行一些必要的渲染和绘制操作。**

  网页或者 app 的代码和浏览器本身的用户界面程序运行在相同的线程中，共享相同的事件循环。该线程就是主线程，它除了运行网页本身的代码之外，还负责收集和派发用户和其他事件，以及渲染和绘制网页内容等。

  事件循环驱动着浏览器中发生的一切，因为它与用户的交互有关，但对于我们这里的目的来说，更重要的是它负责调度和执行在其线程中运行的每一段代码

  **js中的任务分为同步任务和异步任务，浏览器任务分为宏任务和微任务**

### 具体流程
首先需要明确的是: js中的任务分为同步任务和异步任务.
  * 同步任务：所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。只有前一个任务执行完毕，才能执行后一个任务
  * 异步任务：js中的异步任务会被放到任务队列里。
  
基于此，事件循环的具体流程为：

（1）所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。

（2）主线程之外，还存在一个"任务队列"（task queue）。**只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。** 任务队列中包含了宏任务队列和微任务队列。

（3）一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"(先微任务，再宏任务)，看看里面有哪些事件，哪些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。**微任务队列中的每一个微任务会依次被执行。它会等到微任务队列为空才会停止执行——即使中途有微任务加入。换句话说，微任务可以添加新的微任务到队列中，这些新的微任务将在下一个任务开始运行之前，在当前事件循环迭代结束之前执行。**

（4）主线程不断重复上面的第三步。
### 任务队列执行
  ![Alt text](eventloop.png)
### 事件循环
  由于主线程读取的过程是不断重复的，所以叫做事件循环
### 微任务有哪些
  promise, async/await, mutationObserver, QueueMicroTask()等
### 宏观任务有哪些
  点击事件，setTimeout，setInterval，requestAnimationFrame，I/O, UI rendering等
### 面试题
#### 面试题1
  ```js
    //请写出输出内容
    async function async1() {
      console.log('async1 start');
      await async2();
      console.log('async1 end');
    }
    async function async2() {
      console.log('async2');
    }

    console.log('script start');

    setTimeout(function() {
      console.log('setTimeout');
    }, 0)

    async1();

    new Promise(function(resolve) {
      console.log('promise1');
      resolve();
    }).then(function() {
      console.log('promise2');
    });
    console.log('script end');


    /*
    script start
    async1 start
    async2
    promise1
    script end
    async1 end
    promise2
    setTimeout
    */   
  ```  
#### 面试题2
  ```js
  async function async1() {
    console.log('async1 start');
    await async2();
    setTimeout(function() {
        console.log('setTimeout1')  // 这一部分代码会放入到 promise 的微任务队列中。
    },0)
  }
  async function async2() {
    setTimeout(function() {
        console.log('setTimeout2')
    },0)
  }
  console.log('script start');
  setTimeout(function() {
    console.log('setTimeout3');
  }, 0)
  async1();
  new Promise(function(resolve) {
    console.log('promise1');
    resolve();
  }).then(function() {
    console.log('promise2');
  });
  console.log('script end');
  /** script start,
   *  async1 start, 
   *  promise1, 
   *  script end, 
   *  promise2, 
   *  setTimeout3,  
   *  setTimeout2, 
   *  setTimeout1
   **/
  ```  
### 参考文档
  * https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous
  * https://zh.javascript.info/event-loop   
## node中的EventLoop
  - 概念：
    * 事件循环是node处理非阻塞I/O的机制
  - 事件循环机制解析：
    * node.js启动 ---> 初始化事件循环 ---> 处理已经提供的输入脚本（可能调用异步的API，调度定时器，调用Process.nextTick()）---> 处理事件循环
  - 事件循环机制阶段：  
   ![Alt text](node-eventloop.png)
  - 阶段概述
    * timers: 此阶段执行由 setTimeout() 和 setInterval() 排序。
    * pending callbacks: 执行 I/O 回调推迟到下一个循环 迭代。
    * idle, prepare: 仅在内部使用。
    * poll: 检索新的 I/O 事件; 执行与 I/O 相关的几乎任何回调（由“计时器”或 “setImmediate()”所设的紧邻回调除外); node 将在适当时机在此处暂停。
    * check: setImmediate() 回调在此处被调用。
    * close callbacks：一些关闭的回调函数，如：socket.on('close', ...)。   