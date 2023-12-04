# react

## react

### fetch VS ajax VS axios

#### fetch
  - fetch 不是对 ajax 的进一步封装，而是原生的js，没有使用 XMLHttpRequest 对象
  - fetch 返回的 promise 不会因为 HTTP 的错误状态而被拒绝，即使响应是 HTTP 404 或 500.
  - 除非在 init 对象中设置 credentials，否则 fetch() 将不会发送跨源 cookie

#### ajax
  - 基于 XMLHttpRequest

#### axios
  - 基于 Promise 实现，符合最新的 ES 规范。


### 说说你对 React 的理解
  React 起源于 facebook，是一个声明式，高效且灵活用于构建用户界面的 js 库。使用 React 可以将一些简短、独立的代码片段组合成复杂的 UI 界面，这些代码片段被称为“组件”。

#### 特点：
  - **组件(在 react 中是一个组件)：** 
  - **标记语言(JSX)：**
  - **函数式编程，增加 Hooks:**
  - **高效-虚拟 DOM：**
  - **灵活：** 可以和已知的库和框架很好的配合，社区资源丰富
  - **单项数据流：** React 是单项数据流，数据通过 props 从父节点传递到子节点，如果父级的某个 props 改变了， react 会重新渲染所有的子节点。

### React18 新特性
1. Automatic batching(自动批量更新)；
  
  **将多个状态更新合并成一个重新渲染以取得更好的性能的一种优化**

  #### V18前默认不 batching 的场景
  - promise
  - setTimeout
  - 原生事件处理（native event handlers）

  #### V18
  - 所有更新自动 batching

  #### 若不想 batching
  - 使用 flushSync


2. startTransition

  **可以让页面在多个数据更新里保持响应。这个 API 通过标记某些更新为'transitions',来提高用户的交互。**

  实际：可以让我们的页面在展示时时刻保持re-render.

  #### react 中的update
    - urgent updates: reflect direct interaction, like typing, clicking, pressing, and so on;
    - Transition updates: transition the UI from one view to another.

  #### 与setTimeout的区别
    - startTransition 不会被放到下一次 event loop, 是同步立即执行的，这就意味着比 timeout update 更早，低端机体验明显。

  #### 使用场景：
    - slow rendering: re-render 需要耗费大量的工作量
    - slow network: 需要较长时间等待 response 的情况。

3. 支持 React.lazy 的 SSR 架构

  **SSR 场景**

  #### react的 SSR
  1. server: 获取数据
  2. server: 组装返回带有 HTML 的接口
  3. client: 加载 JavaScript
  4. client: hydration, 将客户端的 JS 与服务端的 HTML 结合。

  #### V18前：
  - 按序执行

  #### V18:
  - 支持拆解应用为独立单元，不影响其他模块。

  #### SSR 问题：
    1. server: 获取数据； ---> 按序执行，必须在服务端返回所有 HTML
    2. client: 加载 JavaScript ---> 必须 JS 加载完成
    3. client: hydration, 将客户端的 JS 与服务端的 HTML 结合；---> hydrate 后才能交互。

  #### 流式 HTML & 选择性 hydrate
    1. 流式 HTML
    2. client 进行选择性的 hydration： ```<Suspense>```  

4. ConCurrent Mode(并发渲染，可选)
  
#### 什么是 CM 和 suspense？ 

  在2019年 react conf提出了实验性的版本来⽀持 CM 和 Suspense（可以理解为等待代码加载，且指定 加载界⾯） 

- **CM：** 

  可帮助应⽤保持响应，并根据⽤户的设备性能和⽹速进⾏适当的调整。

  阻塞渲染：如UI update，需要先执⾏对应视图操作，如更新DOM；

  solution：

    a. **debounce：** 输⼊完成后响应，输⼊时不会更新;

    b. **throttle：** 功率低场景卡顿；

- **可中断渲染（CM）：**

  a. **CPU-bound update：** (例如创建新的 DOM 节点和运⾏组件中的代码)：中断当前渲染，切换更⾼优先级；
  
  b. **IO-bound update：** (例如从⽹络加载代码或数据)：response前先在内存进⾏渲染； suspense 以声明的⽅式来“等待”任何内容，包括数据

- **suspense**
  
  **以声明的方式来“等待”任何内容，包括数据**

  #### 误区：

  **Suspense 不是⼀个数据请求的库，⽽是⼀个机制。这个机制是⽤来给数据请求库向 React 通信 说明某个组件正在读取的数据当前仍不可⽤** 

  - **什么不是 suspense**
    1. 不是数据获取方式
    2. 不是一个可以直接用于数据获取的客户端
    3. 它不使数据获取与视图层代码耦合

  - **suspense 可以做什么** 
    1. 能让数据获取库与 React 紧密整合
    2. 能让你有针对性的安排加载状态的展示
    3. 能够消除 race conditions。

### React 组件之间的数据传值

#### 父子组件：
  - props
#### 逆向传值：
  - 父级通过函数传值，子级通过事件调用函数传递
#### 同级：
  - 提升到共同的父级传值
  - 使用三方库，如 pubsub.js. 用 pubsub.publish(事件名, 数据) 抛出数据; 用 pubsub.subscribe(监听的事件，()=>{})接收数据

#### 跨级组件：
  - createContext，useContext()

#### 层级关系复杂，多个组件需要使用
  - redux，mobox 等状态管理库


### React 中如何复用组件
  - 使用高级组件（HOC）： 用于复用组件逻辑。将组件作为参数，返回值也是组件，是纯函数，不会修改传入的组件，也不会使用继承来复制其行为。通过 HOC 将组件包装在容器组件中来组成新的组件。HOC 是纯函数，没有副作用。

#### 使用 HOC 的原因
  - 抽取重复的代码，实现组件复用：相同功能组件复用
  - 条件渲染，控制组件的渲染逻辑（渲染劫持）：权限控制
  - 捕获/劫持被处理组件的生命周期，常见场景：组件渲染性能追踪、日志打点。

#### 实现 HOC 的方式
  - **属性代理：** 使用组合的方式，将组件包装在容器上，依赖父子组件的生命周期来返回 stateless 的函数组件或者返回 class 组件。
    * 操作 props
    * 抽象 state
    * 通过 props 实现条件渲染
    * 其他元素 wrapper 传入的组件
  - **反向继承：** 使用一个函数接受一个组件作为参数传入，并返回一个继承了该传入组件的类组件，且在返回组件的 render() 方法中返回 super.render() 方法。
    * 允许HOC通过this访问到原组件，可以直接读取和操作原组件的state/ref等。如 读取/操作原组件的 state
    * 可以通过super.render()获取传⼊组件的render，可以有选择的渲染劫持。如 条件渲染，修改 React 组件树
    * 劫持原组件⽣命周期⽅法。

#### 属性代理和反向继承对比

1. **属性代理：** 从“组合”⻆度出发，有利于从外部操作wrappedComp，可以操作props，或者在 wrappedComp 外加⼀些拦截器（如条件渲染等）； 

2. **反向继承：** 从“继承”⻆度出发，从内部操作wrappedComp，可以操作组件内部的state，⽣命周期 和render等，功能能加强⼤；

### Hooks 是什么？

  Hooks是react16.8以后新增的钩⼦API；

  **⽬的：** 增加代码的可复⽤性，逻辑性，弥补⽆状态组件没有⽣命周期，没有数据管理状态state的缺陷。

### 为什么要使用 Hooks？
  1. 开发友好，可扩展性强，抽离公共的⽅法或组件，Hook 使你在⽆需修改组件结构的情况下复⽤状态逻辑； 
  2. 函数式编程，将组件中相互关联的部分根据业务逻辑拆分成更⼩的函数； 
  3. class更多作为语法糖，没有稳定的提案，且在开发过程中会出现不必要的优化点，Hooks⽆需学习 复杂的函数式或响应式编程技术；
  
### 常见的 Hooks 有哪些？
  1. useState
  2. useReducer

    可以定义 state 更新逻辑。

    **业务中经常将 useReducer + useContext 代替 Redux**

  3. useContext
  4. useRef

    不同于 useState, useRef 改变值不会使组件 re-render

  5. useEffect

    useEffect： 组件更新挂载完成 -> 浏览器 dom 绘制完成 -> 执⾏useEffect回调 ；

  6. useLayoutEffect：渲染更新之前的 useEffect

    useLayoutEffect ： 组件更新挂载完成 -> 执⾏useLayoutEffect回调-> 浏览器dom 绘制完成；

  6. useMemo
  7. useCallback  

### Hooks VS HOC
  1. Hook最典型的就是取代掉⽣命周期中⼤多数的功能，可以把更相关的逻辑放在⼀起，⽽⾮零散在各 个⽣命周期⽅法中；

  2. ⾼阶组件可以将外部的属性功能到⼀个基础 Component 中，更多作为扩展能⼒的插件（如 react- swipeable-views中的 autoPlay ⾼阶组件，通过注⼊状态化的 props 的⽅式对组件进⾏功能扩展， ⽽不是直接将代码写在主库中）；

  3. Hook 的写法可以让代码更加紧凑，更适合做 Controller 或者需要内聚的相关逻辑，⼀般与⽬标组 件内强依赖，HOC更强调对原先组件能⼒的扩展；

### React 中异步实现方式
  随着项⽬的增⻓，代码包也会随之增⻓，尤其是在引⼊第三⽅的库的情况下，要避免因体积过⼤导致加 载时间过⻓。
  
  React16.6中，引⼊了 React.lazy 和 React.Suspense 两个API，再配合动态 import() 语法就可以实现组件代码打包分割和异步加载。 
  
  **动态import是相对于静态import的 `import XX from XXX`，动态import指在运⾏时加载** ```import('./test.js').then(test => {}) // import 是实现了 promise 规范的，回调函数为返回的模块。```
  
  **传统模式：渲染组件-> 请求数据 -> 再渲染组件** 
  
  **异步模式：请求数据-> 渲染组件；**