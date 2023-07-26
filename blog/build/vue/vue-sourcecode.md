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