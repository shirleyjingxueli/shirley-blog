# vue-基础
## mvc和mvvm
### 定义
  - mvc:
    * m: model - 应用程序的业务和逻辑
    * v: view - 负责展示数据给用户
    * c: controller - 负责接收用户的输入，更新模型和视图
  - mvvm:
    * m: model - 应用程序的业务和逻辑
    * v: view - 负责展示数据给用户
    * vm: viewModel - 作为view和model之间的连接，处理视图的行为和状态，并通过视图绑定的形式将model和view关联
### mvc 和mvvm的区别
  - 数据绑定的方式
    * mvc为单项数据流（只能通过controller将数据传递给模型，然后更新视图），
    * mvvm为双向数据流（通过viewmodel将视图和模型进行双向绑定，viewmodel层将数据转换为视图所需要的格式，并且以数据绑定的格式使得视图和模型保持同步）
### vue解除双向绑定
  * 使用 v-once 只会渲染一次，之后将会作为静态内容，直接跳过，不再做渲染
  * 使用 v-bind 代替 v-model
  * 使用 object.freeze() 设置值
  * 使用计算属性
  * 父组件传递给子组件props，子组件内部克隆数据 
## vue插槽(实现模版定制化，信息需要从外部传入内部)
  * 默认插槽
  * 具名插槽：多个插槽一起使用，以name为标识在组件内部做区分

  ```js
  // 组件内部管
    <slot name="slotA"></slot>
    <slot name="slotB"></slot>
  // 外部调用
    a. <template v-slot:slotA></template>
    b. <template #slotA>

  ```
  * 作用域插槽：父组件需要读取子组件的数据（slot的结构描述写在外部，内部对slot赋值，并且内部的值会反馈到外部）
  ```js
    <current-component>
      <template v-slot:default="slotProps">
        <div> {{slotProps.name}}</div>
      </template>
    </current-component>

    // 组件内部使用v-bind来绑定数据
    // user: {name: 111}
    <div>
      <slot :slotProps="user"></slot>
    </div>
  ```
## 模版数据的二次加工
  * computed watch 不建议
  * 函数
  * v-html
  * filter （无上下文，可以理解为纯函数）
  ```js
    <template>
      <div>{{num | numFilter}}</div>
    </template>
    export default {
      name: 'filter',
      data() {
        return {
          num: 100
        }
      }
      filters: {
        numFilter(num){
          return num > 99 ? 99 : num;
        }
      }
    }
  ```

## render函数
  jsx更自由的基于js编写
  vue的编译原理： template => render() => vdom render => dom render
  使用render函数会损失掉一部分的性能优化

## vue逻辑复用
  * mixin: 实现逻辑复用
  * extends: 拓展独立的逻辑；和mixin区别：mixin是数组形式传入，extends传入方式：extends: extendsDemo
  * 整体拓展：extend => 从预定义的配置中拓展出来一个独立的配置，进行合并
  * 插件
    a.注册外部插件，作为整体实例的补充
    b.会除值，不会重复注册
    c.手写插件
      i. 外部调用使用vue.use(pluginnanme,options)
      ii. 内部封装install方法

### 组件的高级引用
  * 递归组件：自身调用，实现无限嵌套
  * 动态组件： ```<component :is="currentComponentName"></component>```
  * 异步组件  ```const component = () => import('xxxxx.vue');```

### vue中实现懒加载的方式
  * 使用异步组件
  * 使用路由懒加载
    ```js
    const router = new VueRouter({
      routes: [{
        path:'xxxx',
        component: () => import('..../xxx.vue')
      }]
    })
    ```
  * 使用webpack动态导入
    const {default: component} = await import('xxx.vue');
  * chunk分包
    a. webpack配置项
      ```
        module.exports = {
          entry: 'xxx.js',
          output: {
            filename: [name].bundle.js,
            chunkFilename: [name].chunk.js
          }
        }
      ```
    b. require.ensure()
      ```
      const Home = resolve => require.ensure('./views/Home.vue'), () => {
        resolve(require('./views/Home.vue'))
      })
      ```
### chunk分包实现懒加载的缺点
  * 加载额外资源：
    使用 chunk 分包时，会将代码和资源进行切分，需要额外加载许多模块和资源文件。如果没有得到很好的处理，可能会导致加载时间变慢，影响网页的性能和速度。
  * 多次请求：
    使用 chunk 分包会将应用程序切分为多个块，这些块可能需要在不同的情况下进行请求。如果请求过程中发生了网络中断或缓存被清除等情况，可能会导致网页加载失败或变慢。
  * 调试不方便：
    使用 chunk 分包会将代码切分成许多块，使得调试变得更加困难。这时，需要使用特定的工具来处理问题，这可能会增加调试难度。
  * 依赖关系复杂：
    使用 chunk 分包时，需要考虑模块之间的依赖关系。如果模块之间的依赖关系非常复杂，可能会导致分包过程变得更加困难，需要更多的工具和平台来处理依赖关系。
  * 代码维护困难：
    使用 chunk 分包时，需要对代码进行切割处理，这使得代码维护变得更加困难。如果应用程序非常庞大，可能需要在切割过程中增加更多的标准和规则。  


## 面试题
### vue中key的作用
  key是虚拟dom的唯一标识。key的作用主要用在vue的diff算法上。
  
  在新旧dom对比时，key可以帮助快速准确定位更新的元素，管理可以复用的元素，减少不必要的更新，能够高效的重用和渲染优化。

### v-for 中为什么不建议使用index作为key
- **用 index 作为 key 时，在对数据进行，逆序添加，逆序删除等破坏顺序的操作时，会产生没必要的真实 DOM更新，从而导致效率低**
  ```js
    <template>
      <div class="hello">
        <ul>
          <li v-for="(item,index) in studentList" :key="index">{{item.name}}</li>
          <br>
          <button @click="addStudent">添加一条数据</button>
        </ul>

      </div>
    </template>

    <script>
    export default {
      name: 'HelloWorld',
      data() {
        return {
          studentList: [
            { id: 1, name: '张三', age: 18 },
            { id: 2, name: '李四', age: 19 },
          ],
        };
      },
      methods:{
        addStudent(){
          const studentObj = { id: 3, name: '王五', age: 20 };
          this.studentList=[studentObj,...this.studentList]
        }
      }
    }
    </script>
  ```
- **用 index 作为 key 时，如果结构中包含输入类的 DOM，会产生错误的 DOM 更新**
  ```js
    <template>
      <div class="hello">
        <ul>
          <li v-for="(item,index) in studentList" :key="index">{{item.name}}<input /></li>
          <br>
          <button @click="addStudent">添加一条数据</button>
        </ul>
      </div>
    </template>

    <script>
    export default {
      name: 'HelloWorld',
      data() {
        return {
          studentList: [
            { id: 1, name: '张三', age: 18 },
            { id: 2, name: '李四', age: 19 },
          ],
        };
      },
      methods:{
        addStudent(){
          const studentObj = { id: 3, name: '王五', age: 20 };
          this.studentList=[studentObj,...this.studentList]
        }
      }
    }
    </script>

  ```
### vue2和vue3的区别
