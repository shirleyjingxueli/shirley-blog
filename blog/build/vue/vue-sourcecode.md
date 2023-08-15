# vue源码解析
## 事件设计 - 为何vue把事件写在模板上，而不是js中
  - 模板定位事件触发源 + 触发源寻找触发事件逻辑 —— 更方便定位问题
  - js与事件本身解耦 —— 更便于测试隔离
  - viewModel销毁，自动解绑事件 —— 更便于回收
## vue-响应式实现原理
![Alt text](vue-reactive.png)
## vue-diff算法
  - vue2 遍历
  - vue3 最大递升子序列
## keep-alive
### keep-alive组件生命周期
  * created: 组件被创建，但此时尚未进入缓存。
  * beforeMount: 组件开始挂载，但在挂载之前，它可能被缓存组件替代。
  * mounted: 组件被挂载，但只会触发一次。
  * activated: 缓存组件激活时触发，即组件从缓存中取出并显示。
  * deactivated: 缓存组件失活时触发，即组件被放回缓存中并不再显示。
  * beforeDestroy: 组件开始销毁，但在销毁之前，它可能被缓存组件替代。
### keep-alive组件具体流程  
  * 当一个组件被包裹在 keep-alive 标签中时，它首先会触发 created 钩子。
  * 当组件第一次渲染时，它会触发 beforeMount 和 mounted 钩子。
  * 当组件切换到其他路由或被 v-if 等条件隐藏时，它会触发 deactivated 钩子，此时组件会被缓存。
  * 当再次进入缓存的组件时，它会触发 activated 钩子，此时组件会从缓存中取出，并触发 beforeMount 和 mounted 钩子。
  * 如果缓存的组件被销毁，它会触发 beforeDestroy 钩子。
### 使用keep-alive的优缺点
  * 好处：可以有效的减少组件的销毁和创建，提高性能
  * 缺点：因为组件一直被缓存在内存中，可能会提高内存的占用  

## vue-router 
### 什么是vue-router,router的发展历程
    router是vue的一个路由管理，是基于spa的产物,spa之前都是由服务端来控制的；只有一个html，前端可以控制路由

    通过监听hash，go,back,foward等行为来控制路由

    后端路由是通过url访问对应的controller，进行数据和模版引擎的拼接，返回给前端

    前端路由是通过js根据url加载对应的组件

    所以前端路由包括两部分：处理url， 加载对应的组件

### 路由有哪几种
    hash route; history route(go | back | forward | push | replace ); memory 路由；
  * hash route 和 history route的区别
    * hash 路由 ⼀般会携带 ⼀个 # 号，不够美观； history 路由不存在这个问题； 
    * 默认 hash 路由是不会像浏览器发出请求的，主要是⼀般⽤于锚点；history 中 go / back / forward 以及浏览器的前进、后退按钮⼀般都会像服务端发起请求；-- history 的所有 url 内容，服 务端都可以获取到 
    * 基于此，hash 模式，是不⽀持SSR的，但是 history 模式可以做 SSR 
    * history 在部署的时候，如 nginx， 需要只渲染⾸⻚，让⾸⻚根据路径重新跳转。
    ```
      # 单个的服务器部署
      location / {
        try_files uri $uri /xxx/main/index.html
      }
      # 存在代理的情况
      location / {
        rewrite ^ /file/index.html break; # 这⾥代表的是xxx.cdn 的资源路径
        proxy_pass https://www.xxx.cdn.com;
      } 
    ```

### 异步组件
  通过import(); react.lazy() 对代码进行动态拆分的技术, 等需要时再进行加载

### 路由守卫
  - 全局：beforeEach afterEach
  - 组件：beforeRouteEnter beforeRouteUpdate beforeRouteLeave

### 路由守卫的流程
  - 导航被触发。
  - 在失活的组件里调用 beforeRouteLeave 守卫。
  - 调用全局的 beforeEach 守卫。
  - 在重用的组件里调用 beforeRouteUpdate 守卫(2.2+)。
  - 在路由配置里调用 beforeEnter。
  - 解析异步路由组件。
  - 在被激活的组件里调用 beforeRouteEnter。
  - 调用全局的 beforeResolve 守卫(2.5+)。
  - 导航被确认。
  - 调用全局的 afterEach 钩子。
  - 触发 DOM 更新。
  - 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

### vue router实现原理
  a. createWebHistory()
  - 包含当前路径 ====> 根据location.path, search, hash 进行拼接
  - 状态 ====> 通过window.location.state来获取浏览器的状态
    i 有时候是无状态的，需要维护一个自己的状态 buildstate (比如 go, forward, push, replace, 第一次刷新界面时)
    iii 同步状态给location 和historystate
    ```
      function changeLocation(to, state, replace) {
        window.history[replace?'replace':'pushState'](state, null, to);
        historyState.value = state;
      } 
    ```
  - push, replace等方法 (利用useHistoryNavigation() 方法实现push和state，实际上记录的是push和replace时的state的信息)
  - 监听器，监听popstate，执行listener，listener里面存储的是state的信息{to,from,isBack}

## vue-cli  
  cli工具地址：
    - https://github.com/sindresorhus/ora
    - https://github.com/tj/commander.js
    - https://www.npmjs.com/package/inquirer

  download-git-repo: 下载模版
  commander: 命令行工具
  path: mode模块
  ora: node模块-node的命令行界面增强工具，可以在命令行中增加提示，只依赖于node.js中的process模块
  chalk: node.js 中的一个模块，用于在控制台输出彩色的文字，提升文本可读性    
  inquirer: 命令行与开发者交流工具

  * vue-cli 是用来做什么的
    用于快速搭建一个项目
    下载模版, metadata配置，渲染生成
    利用download-git-repo下载模版，利用commader构建命令行工具，inquirer构建可交互的命令行界面，ora增加提示
    根据git config --get user.name, git config --get user.email 获取用户的git名称以及邮件
    核心逻辑：
      a. 判断是local-template 或者是remote-template
      b. 如果是local-template, 判断项目目录，如果存在 => 构建项目
      c. 如果是remote-template, 检查版本号 => 是否是官方模版（加载官方或者第三方模版）=> 构建项目
      d. 生成过程
         -- 获取模版配置
         -- 初始化metalsmith
         -- 模版的注册 => before执行 => 问询 & 主流程 => after执行 => complete => 打印message

## vuex
### vuex为什么要自己设置告警
  * 可以实现对告警级别的控制  

## 面试题：
### vue2的响应式原理
  * proxy(): 实现this.$data.message => this.message
  * observer();
    a. 调用walk函数遍历data
    b. 利用Object.defineProperty(obj, key, {})对属性拦截
      i.在get时调用dep.add(Dep.target)添加依赖
      ii.在set时调用dep.notify()来更新视图 => 调用watcher.update();
    c. Dep类存储的是一个watcherList
    d. Watcher类: 用来监听数据的变化，更新视图 new Watcher(vm, 'num', () => {更新视图})
       设置Dep.target = Watcher, 设置update方法用来更新函数
  * complier(): 对模版的编译，创建watcher实例

### vue3的响应式原理
  * 全局的targetMap, activeEffect
  * reactive(), ref();
    a. 利用proxy实现代理, reactive => get(), set(), has(), ownKeys(), deleteProperty(); ref() => get value(), set value(),挂载在value上
    b. get操作时进行track, set时进行trigger
    c. track => 调用dep.add(activeEffect),依赖项实际上为ReactiveEffect; trigger => 遍历deps并执行effect.run()
    d. ReactiveEffect(fn): 设置全局activeEffect(实际上为ReactiveEffect实例)，具有一个run方法，用来执行传入进来的fn
  * mount(): 获取setup上导出的data，实例化 ReactiveEffect(),调用instance.render()方法把页面渲染出来


### proxy()和Object.defineProperty()的区别
共同点：都可以用来实现对对象的监听和代理

不同点：
  * 语法不同: new Proxy(target, handler) Object.defineProperty(obj, key, descriptor);
  * proxy是创建一个代理对象，是对代理对象的监听，Object.defineProperty是对当前对象定义属性或者修改属性的拦截
  * proxy可以实现对多种功能的拦截，get, set, has, deleteProperty,ownKeys, apply等,Object.defineProperty只对对象属性的书写或者修改做拦截
  * 性能：proxy因为要动态代理代理对象，所以性能相对较低，如果不需要多种拦截器的时候，建议使用Object.defineProperty
  * 兼容性不同：proxy是es6中的新增功能，Object.defineProperty是早期版本就有的功能，可以在所有浏览器以及node环境中使用

### vue3 diff算法
- 采用最大递增子序列
  如果 e1 < i1, 说明第一个已经跑完，那么i2到e2为新增节点
  // a, b, c, e, f
           e1 i1
  // a, b, c, h, d, e, f
              i2 e2

  反之，则说明是删除节点
  // a, b, c, h, d, e, f
              i1 e1  
  // a, b, c, e, f
           e2 i2  
  其他情况，掐头去尾，中间最小移动（最大上升子序列）  
  
### Vue2.x 和 Vue3.x 的对比
- Vue 3.x 中使用了 Proxy 作为响应式，天生的代理，不用考虑属性重写、数组这些2.x 中hack的情况；
- diff, 增加了最大递增子序列的算法，让我移动节点，更高效；
- 架构采用 monorepo 的方式，分层清晰，同时把编译部分也进行了一些拆解；
- vue3 对编译的内容，进行了重写，template -- render 函数。
    - vue2 正则， vue3 状态机; -- [ast 编译原理]
    - patchFlag, 标记哪些元素包含哪些属性
    - 静态提升
- vue3 使用 blockTree, 对比需要改变的，优化性能，如果你要用 jsx 的写法，就不会优化，但是可以自己去标记。
- ts 重构。
- compiler 拆成了四个包。方便你去重写。
- vue2 options API -- vue3 composition API
- vue3，使用了 rollup 打包，支持 treeshaking。
- 实例化方式也有区别；


