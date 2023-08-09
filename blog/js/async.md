# js-异步编程

## 概念
### 同步
  同步行为对应内存中顺序执行的处理指令。每条指令都会严格按照它们出现的顺序来执行，而每条指令执行之后也能立即获得存储在本地（如寄存器或者系统内存）的消息。

### 异步
  异步行为类似于系统中断，即当前进程外部的实体可以触发代码执行。

  **当不想为等待某个异步操作而阻塞线程的执行时，可以使用异步操作。**

## 以往的异步编程
### 回调
  在早期的js中，只支持定义回调函数来表明异步操作完成。串联异步操作是一个常见的问题，通常需要深度嵌套的回调函数（回调地狱）来解决。

#### 解决方案
  - 异步返回值：给异步操作提供一个回调，这个回调中包含要使用异步返回值的代码（作为回调的参数）。
  - 失败处理：提供成功回调和失败回调。
  - 嵌套异步回调：如果一个异步返回值依赖另一个异步返回值，那么就要依赖于嵌套回调。

## es6中的异步编程
### Promise
#### 状态机
  - pending: 尚未开始执行或者正在执行过程中（默认）

    **Promise并非一开始就必须处于pending状态，然后通过执行器才能转换状态为落定状态。**

    **可以通过Promise.resolve()静态方法，实例化一个解决的Promise**

  - fullfilled: 成功完成
  - rejected: 没有成功完成
#### 状态机特点
  - 状态是不可逆的。状态一经兑现或者拒绝后，将不再改变
  - 状态是私有的，不能直接通过js检测到。避免外部读取到状态后，以同步方式处理Promise对象
  - 状态不能被外部javascript代码修改。Promise故意将异步行为封装起来，从而隔绝外部的同步代码。
#### 执行器
  - 执行器的主要职责是初始化Promise的异步行为和控制状态的最终转换。
  - 通过执行函数控制Promise状态（调用函数参数 resolve() 和 reject()）
#### then方法
  - then方法为Promise实例添加处理程序的主要方法。接收最多两个参数，onResolved 和 onRejected。
  - then方法返回的是一个新的Promise实例
#### catch方法
  - catch方法用于给Promise添加拒绝处理程序。这个方法只接收一个参数，onRejected 处理程序。相当于调用Promise.prototype.then(null, onRejected)
  - catch方法返回的是一个新的Promise实例(返回一个已解决的Promise)
#### finally方法
  - finally方法用于给Promise添加onFinally处理程序，这个程序在Promise转为解决或者拒绝状态时都会执行。  
  - finally方法返回一个新的Promise实例。这个新Promise实例不同于 then()或 catch()方式返回的实例。因为 onFinally 被设计为一个状态无关的方法，所以在大多数情况下它将表现为父Promise的传递。对于已解决状态和被拒绝状态都是如此。
#### all 和 race 方法
  - all方法接受一个Promise可迭代对象作为输入，并且返回一个Promise. 当所有输入的 Promise 都被兑现时，返回的 Promise 也将被兑现（即使传入的是一个空的可迭代对象），并返回一个包含所有兑现值的数组。**如果输入的任何 Promise 被拒绝，则返回的 Promise 将被拒绝，并带有第一个被拒绝的**
  - race方法接受一个 promise 可迭代对象作为输入，并返回一个 Promise。这个返回的 promise 会随着第一个 promise 的敲定而敲定。
#### 非重入方法
  - **当Promise进入落定状态时，与该状态相关的处理程序仅仅会被排期，而非立即执行。**
  - 跟在这个处理程序的代码之后的同步代码一定会在处理程序之前执行。即使Promise一开始就是落定的状态，执行顺序也是这样的。
  - 这个特性由javascript运行时保证，被称为"非重入"（non-reentrancy）
  ```js
    // 创建已经解决的promise
    let p = Promise.resolve();
    
    // 添加处理程序
    // 直觉上这个处理程序会等到Promise一解决就执行
    p.then(()=>{
      console.log("onResolved handler")
    });

    // 同步输出，证明then()已经返回
    console.log("then() returns");

    // 实际输出结果：
    // then() returns
    // onResolved handler
  ``` 
#### 邻近处理程序的执行顺序
  如果给Promise添加了多个处理程序，当Promise状态变化时，相关的处理程序会按照添加的顺序依次执行。无论是then()、catch()、finally()添加的处理程序都是如此。
#### 传递解决值和拒绝理由
  - resolve();
  - reject(); 

  解决的值和拒绝的理由会分别作为resolve()和reject()方法的第一个参数往后传。然后，这些值又会传递给它们各自的处理程序，作为onResolved 和 onRejected 处理程序的唯一参数。
#### 拒绝Promise和拒绝错误处理
  - 在Promise的执行函数或者处理程序中抛出错误会导致拒绝，对应的错误对象会成为拒绝的理由。比如：
  ```js
    let p1 = new Promise((resolve, reject) => reject(Error("foo")));
    let p2 = new Promise((resolve, reject) => { throw Error("foo") });
    let p3 = Promise.resolve().then(() => {throw Error("foo")});
    let p4 = Promise.reject(Error("foo"));
    let p5 = p1.catch(()=> {throw Error("ceshi")});

    setTimeout(console.log, 0, p1); // Promise <rejected>: Error: foo
    setTimeout(console.log, 0, p2); // Promise <rejected>: Error: foo
    setTimeout(console.log, 0, p3); // Promise <rejected>: Error: foo
    setTimeout(console.log, 0, p4); // Promise <rejected>: Error: foo
    setTimeout(console.log, 0, p5); // Promise <rejected>: Error: ceshi
  ```   
  - 在Promise中抛出错误时，因为错误实际上是从消息队列中异步抛出的，所以并不会阻止运行时继续执行同步指令。 比如：
  ```js
    Promise.reject(Error("foo"));
    console.log("bar");

    //实际输出顺序为：
    // bar
    // uncaught (in promise) Error: foo;
  ```
  - then() 和 catch() 的 onRejected 处理程序在语义上相当于try/catch. 出发点都是捕获错误之后将其隔离，同时不影响正常逻辑执行。**为此，onRejected 处理程序的任务应该是在捕获异常错误之后返回一个解决的Promise**

  例子：

  ```js
    new Promise((resolve, reject) => {
      console.log("begin async exec");
      reject(Error("bar"));
    }).catch((e) => {
      console.log("caught error", e)
    }).then(()=>{
      console.log("continue async exec")
    })

    // 输出结果：
    // begin async exec;
    // caught error Error: bar;
    // continue async exec;
  ```

### async/await
  异步函数，是es6 Promise在es函数中的应用。这个特性从行为和语法上都增强了Javascript,**让以同步方式写的代码能够异步执行。**  
#### 背景
  **es8的async/await旨在解决利用异步结构组织代码的问题。** 为此ES对函数进行了扩展，增加了两个关键字async、await。

  async: 
  - 用于声明异步函数。这个关键字可以用在函数声明，函数表达式，箭头函数和方法上。 
  - 让函数具有异步特征，但总体上其代码执行仍然是同步求值的。
  - 异步函数如果使用return关键字返回了值（如果没有return则会返回undefined），这个值会被Promise.resolve()包装成一个Promise对象。
  - **异步函数始终返回Promise对象。在函数外部调用这个函数，可以得到它返回的Promise对象。**
  - 异步函数的返回值期待（但实际上并不要求）一个实现 thenable 接口的对象，但常规的值也可以。如果返回的是实现 thenable 接口的对象，则这个对象可以由提供给 then()的处理程序“解包”。如果不是，则返回值就被当作已经解决的Promise。
  - 与Promise处理程序一样，在异步函数中抛出错误会返回拒绝的Promise
  - 拒绝Promise的错误不会被异步函数捕获。 例如：
    ```js
      // 之所以未捕获到错误，是因为这个异步函数的返回值是一个值为undefined的resolved的promise。而不是一个rejected的Promise
      async function foo() { 
        console.log(1); 
        Promise.reject(3); 
      } 

      // Attach a rejected handler to the returned promise 
      foo().catch(console.log); 
      console.log(2); 
      // 1 
      // 2 
      // Uncaught (in promise): 3
    ```

  await: 
    - 使用await关键字可以暂停异步函数代码的执行，等待Promise的解决。
    - await会暂停执行异步函数后面的代码，让出js运行时的执行线程。这个行为与生成器函数中的yield关键字是一样的。
    - await关键字同样是尝试解包对象的值，然后将这个值传给表达式，再异步恢复异步函数的执行。
    - await 关键字期待（但实际上并不要求）一个实现 thenable 接口的对象，但常规的值也可以。如果是实现 thenable 接口的对象，则这个对象可以由 await 来“解包”。如果不是，则这个值就被当作已经解决的Promise。
    - await 会抛出错误的同步操作，会返回拒绝的Promise
    - 单独的Promise.reject()不会被异步函数捕获，而会抛出未捕获错误。不过对拒绝的Promise使用 await 则会释放错误值（返回拒绝的Promise）。 例如：
    ```js
      async function foo(){
        console.log(1);
        await Promise.reject(3);
        console.log(4);
      }
      
      // foo()的返回值为 Promise {<rejected>: 3}
      foo().catch(console.log);
      console.log(2);
      //1 -> 2 -> 3
    ```
    - await 关键字必须在 async 函数中使用，不能再顶级上下文如```<script>```标签中使用。不过定义并立即调用异步函数是没有问题的。
    - 执行原理：
      - **js运行时在碰到await关键字时，会记录在哪里暂停执行。等到await右边的值可用了，js运行时会向消息队列中推送一个任务，这个任务会恢复异步函数的执行。即使await后面跟着一个立即可用的值，函数的剩余部分也会被异步求值。**
      - await 后面如果跟的是Promise，比如Promise.resolve(8)也只会生成一个异步任务。
## 小结
  期约的主要功能是为异步代码提供了清晰的抽象。可以用期约表示异步执行的代码块，也可以用期约表示异步计算的值。在需要串行异步代码时，期约的价值最为突出。作为可塑性极强的一种结构，期约可以被序列化、连锁使用、复合、扩展和重组。

  异步函数是将期约应用于 JavaScript 函数的结果。异步函数可以暂停执行，而不阻塞主线程。无论是编写基于期约的代码，还是组织串行或平行执行的异步代码，使用异步函数都非常得心应手。

## 面试题
1. 说出下面代码的执行顺序

  ```js
    async function foo() { 
      console.log(2); 
      await null; 
      console.log(4); 
    } 

    console.log(1); 
    foo(); 
    console.log(3); 
    // 1 
    // 2 
    // 3 
    // 4 
  ```

  控制台中输出结果的顺序很好地解释了运行时的工作过程：

  (1) 打印 1；

  (2) 调用异步函数 foo()；

  (3)（在 foo()中）打印 2；

  (4)（在 foo()中）await 关键字暂停执行，为立即可用的值 null 向消息队列中添加一个任务；

  (5) foo()退出；

  (6) 打印 3；

  (7) 同步线程的代码执行完毕；

  (8) JavaScript 运行时从消息队列中取出任务，恢复异步函数执行；

  (9)（在 foo()中）恢复执行，await 取得 null 值（这里并没有使用）；

  (10)（在 foo()中）打印 4；

  (11) foo()返回

2. 说出下面代码的执行顺序

    ```js
      async function foo() {
        console.log(2);
        console.log(await Promise.resolve(8));
        console.log(9)
      }

      async function bar() {
        console.log(4);
        console.log(await 6);
        console.log(7);
      }

      console.log(1);
      foo();
      console.log(3);
      bar();
      console.log(5);
      
      // 输出结果： 1, 2, 3, 4, 5, 8, 9, 6, 7
    ```



