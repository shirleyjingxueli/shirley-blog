# 项目相关
## 自适应设计方案
### 响应式设计(比如PC界面在不同屏幕上有不同的布局)
#### 特点：
  **通过媒体查询以及弹性布局技术，使得网站或者应用程序能够根据设备的屏幕尺寸和分辨率变化而变化**

  **可以实现内容的重新排列、图像和媒体的缩放、导航的变化等，以适应不同大小的屏幕**
#### 具体应用：
  [媒体查询列表](https://developer.mozilla.org/en-US/docs/Web/API/MediaQueryList)

  1. 调用 ```window.matchMedia()``` 创建一个 ```MediaQueryList```对象。**当媒体查询状态发生更改时，生成的对象向侦听器发送通知。**
  2. 监听 ```MediaQueryList```对象的change事件 

#### 例子
  ```js
   // 1. 创建mediaQueryList;
    let mediaQueryList = {
      matchMobile: null,
      matchPC: null,
    }
    // 2. 创建mediaQuery对象
    if (window.matchMedia) {
      mediaQueryList.matchMobile = window.matchMedia('(max-width: 499px)');
      mediaQueryList.matchMobile = window.matchMedia('(min-width: 1280px)');
    }

    //matchMedia监听change的兼容方案，Safari 14之前的版本仅支持addListener
    function addChangeListener(target, cb) {
      if (target.addEventListener) {
        return target.addEventListener('change', cb);
      }

      return target.addListener(cb);
    }

    // 3. 封装方法
    /**
     * @param name string {'mediaQueryMatchMobile' ,'mediaQueryMatchPad'}
     * @param callback {function}
     */
    export function mediaQueryListenerInitial(name, callback) {
      if (mediaQueryList) {
        if (mediaQueryCallback[name]) {
          // 已经存在一个同名的,则移除上一个
          mediaQueryListenerDestroy(name);
          mediaQueryCallback[name] = null;
        }
        mediaQueryCallback[name] = callback;
        // 初始化的时候执行一次
        callback(mediaQueryList[name]);
        addChangeListener(mediaQueryList[name], callback);
      }
    }

    /**
     * @param names string { ['mediaQueryMatchMobile', 'mediaQueryMatchPad'] }
     */
    export function mediaQueryListenerDestroy(names) {
      if (mediaQueryList) {
        if (Array.isArray(names) && names.length > 0) {
          names.forEach((name) => {
            mediaQueryList[name].removeEventListener('change', mediaQueryCallback[name]);
          });
        }
        if (typeof names === 'string') {
          mediaQueryList[names].removeEventListener('change', mediaQueryCallback[name]);
        }
      }
    }

    // 4. 调用(以vue为例子，在App.vue中调用)
    // 在mounted生命周期里使用
    mounted() {
      mediaQueryListenerInitial('matchMobile', this.matchMobile);
      mediaQueryListenerInitial('matchPC', this.matchPC);
    },

    // 5. 把初始化，或者监听到的信息存储到store中，其他组件可以从store中获取到当前设备的屏幕尺寸信息。
    // 6.封装响应式组件
    // 从store中读取相应的尺寸，判断当前屏幕属于什么设备（比如PC,mobile等）,根据不同的设备可以渲染不同的组件。
  ```  
### 移动优先设计（比如内嵌H5）
#### 特点：
  **首先关注移动设备的用户体验，然后逐渐增强以适应大尺寸的屏幕。**

  **移动优先设计强调简洁性、快速加载和易用性，通过优化移动体验来提供更好的用户体验。**

#### 例子
  主要在移动端体验，pc端以最大宽度为500px居中显示。

### 使用响应式布局技术
  - **Multicol**
  - **FlexBox**
    * display: flex;
    * flex-direction: row(默认值)
    * justify-content(主轴对齐方式): flex-start(默认值)|flex-end|center|space-between|space-around|space-evenly
    * align-items(交叉轴对齐方式): stretch(默认值)|flex-start|flex-end|center|baseline
    * flex-wrap(是否换行以及换行方式):nowrap(默认值)|wrap|wrap-reverse
    * flex: none; 是 flex-grow（项目在空间剩余时放大比例）; flex-shrink（项目在空间不足时缩小比例）; flex-basis（指定项目在主轴上的初始大小）的缩写
    * align-self（控制单个项目在交叉轴上的对齐方式）: auto(默认值，继承父容器的align-items,如果父容器没有设置，则等同于stretch) 
  - **CSS Grid**


#### 参考文档
  [响应式布局](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Responsive_Design)
  [css布局方式](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout)

### 响应式排版
  - **使用媒介查询进行响应式排版**
  - **使用视口单位(vw,vh)**

### 使用flex布局实现一个九宫格
```html
  <!DOCTYPE html>
  <html>
  <head>
    <title>九宫格布局</title>
    <style>
      html, body {
        height: 100%;
        margin: 0;
      }

      .container {
        display: flex;
        flex-wrap: wrap;
        height: 100%;
        align-content: space-around; /* 使用 align-content 属性垂直居中项目 */
      }

      .item {
        flex-basis: 33.33%; /* 每个项目占据容器的三分之一宽度 */
        height: 33.33%; /* 每个项目占据容器的三分之一高度 */
        box-sizing: border-box;
        border: 1px solid #000; /* 可选：添加边框样式 */
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="item">1</div>
      <div class="item">2</div>
      <div class="item">3</div>
      <div class="item">4</div>
      <div class="item">5</div>
      <div class="item">6</div>
      <div class="item">7</div>
      <div class="item">8</div>
      <div class="item">9</div>
    </div>
  </body>
  </html>
```  

## 组件，公用utils库管理
### 发布组件，utils npm 包
### 如何发布一个npm包
### 使用npm包的好处
  - 共享和重用代码
  - 便于版本控制和更新管理
    通过发布 npm 包，可以使用版本号来管理代码的不同版本。这使得开发者可以使用特定版本的包，以确保项目的稳定性和兼容性。可以通过发布新版本来修复错误、添加功能、改进性能等。

  - 自动化构建和部署

    在发布 npm 包时，可以利用 CI/CD (持续集成和持续部署) 工具来自动化构建和部署的过程。这样可以减少人为错误，确保每次代码更改都能正确的构建和发布
### npm 包版本号规范
#### **标准的版本号由三个数字组成：主版本号（Major）、次版本号（Minor）和修订号（Patch）**
  
  - 主版本号：进行不兼容的 API 更改时，应该增加主版本号
  - 次版本号：添加向后兼容的功能时，应该增加次版本号
  - 修订号：进行向后兼容的错误修复时，应该增加修订号。修订号表示没有增加新功能，只是修复了现有代码的缺陷。

#### **除了这些版本以外，还可以添加预发布版本和构建元数据**

  - 预发布版本：在版本后面添加预发布标识，表示开发阶段的版本。例如：例如，1.0.0-alpha.1 表示预发布的 Alpha 版本。
  - 构建元数据：在版本号后面添加构建标识符，表示构建过程中的元数据。例如，1.0.0+20130313144700 表示构建的时间戳。

## 国际化（vue-i18n）
### 使用方法
  - 引入 i18n： ```npm install vue-i18n```
  - 创建并注册i18n： 
  ```js
    const VueI18n = createI18n({
      locale: "zh",
      fallbackLocale: "en",
      messages,
      datetimeFormats: {},
      numberFormates: {},
      modifiers: {
        snakeCase: (str) => str.split('').join('-')
      },
      pluralRules: {},
    })

    console.log("VueI18n",VueI18n)
    const app = createApp(App);

    app.use(VueI18n);

    app.mount('#app')

  ```
  - 在组件中使用：
    ```this.$t('key',params)``` 
     ```$t('key',params)``` 
     ```$i18n.t('key',params)``` 
  - 
### $t 和 v-t 的区别
  - $t: 

    优点：能够在模板中灵活使用```{}```语法，也可以在vue组件实例中灵活使用计算属性和方法。

    缺点： 每次重新渲染都会执行，因此有翻译成本。

  - v-t:

    优点：比 ```$t函数```具有更好的性能，因为它可以通过 vue-i18n-extensitions 提供的 Vue 编译器模块进行预翻译。因此可以进行更多的性能优化。

    缺点： 不能像 ```$t```那样灵活使用，比较复杂。带有 ```v-t``` 的翻译内容将插入到元素的 ```textContext```中。另外如果使用服务器端渲染时，您需要将自定义转换设置为 ```@vue/compiler-ssr``` 的 compile 函数的 ```directiveTransforms``` 选项。  
### 如何插入复杂的表达式，比如包含链接等
  - 定制多个字段
  - 在消息配置中配置 ```<a>``` 标记, 有可能因为使用了 ```v-html="$t(key)"``` 进行本地化而出现 XSS 漏洞。
  - 可以使用 ```<i18n-t></i18n-t>```组件来避免上面的情况
  - 使用 slots syntax（插槽语法）
### i18n 作为单文件怎么配置
  在 i18n 自定义块中，语言环境消息资源的格式默认为 json 格式。

  要使用 i18n 自定义块，需要结合插件使用

 -  **以 webpack 为例子，需要结合 @intlify/unplugin-vue-i18n/webpack 来使用。**

  ```js
  
    // webpack.config.js
    const path = require('path')
    const VueI18nPlugin = require('@intlify/unplugin-vue-i18n/webpack')

    module.exports = {
      /* ... */
      plugins: [
        /* ... */
        VueI18nPlugin({
          /* options */
          // locale messages resource pre-compile option
          include: path.resolve(__dirname, './path/to/src/locales/**'),
        })
      ]
    }
  ```

  - **如果需要使用 i18n tag，需要结合 Quasar CLI， 同时也需要安装 @intlify/vue-i18n-loader**

    ```js
      build: {
        chainWebpack: chain => {
          chain.module
            .rule('i18n-resource')
              .test(/\.(json5?|ya?ml)$/)
                .include.add(path.resolve(__dirname, './src/i18n'))
                .end()
              .type('javascript/auto')
              .use('i18n-resource')
                .loader('@intlify/vue-i18n-loader')
          chain.module
            .rule('i18n')
              .resourceQuery(/blockType=i18n/)
              .type('javascript/auto')
              .use('i18n')
                .loader('@intlify/vue-i18n-loader')
        }
      }
    ```

     ```npm i --save-dev @intlify/vue-i18n-loader```

### locale 语言如何做延迟加载
  
  [i18n官网文档](https://vue-i18n.intlify.dev/guide/advanced/lazy.html)

  大概可以理解为：
  
  **进行封装，导出三个函数：**

  - setupI18n： 采用和 ```createI18n``` 相同的选项，使用这些选项来创建 i18n 实例，执行 ```setI18nLanguage ```函数，返回 i18n 实例

  - setI18nLanguage： 通过将参数 i18n 的区域设置为参数 locale 的值来设置语言。此外，该函数还具有将 HTML 文档的 lang 属性设置为参数 locale 的值的功能。与 HTTP 客户端一样，你可以设置语言。

  - loadLocaleMessages: 这个函数是我们实际用来更改于洋的函数。加载新文件是通过 ```import``` 函数完成的，该函数由 webpack 提供，它允许我们动态加载文件，并且因为它使用的是 Promise， 所以我们可以轻松的等待加载完成。

### i18n 如何和 vue3结合？

  - 基础配置：
    ```js
      // ...

      // 2. Create i18n instance with options
      const i18n = VueI18n.createI18n({
        legacy: false, // you must set `false`, to use Composition API
        locale: 'ja', // set locale
        fallbackLocale: 'en', // set fallback locale
        messages, // set locale messages
        // If you need to specify other options, you can set other options
        // ...
      })

      // ...

      // ...

      // 3. Create a vue root instance
      const app = Vue.createApp({
        setup() {
          const { t } = VueI18n.useI18n() // call `useI18n`, and spread `t` from  `useI18n` returning
          return { t } // return render context that included `t`
        }
      })

      // ...

      //...

      //4. 在模版中使用：
      <div id="app">
        <p>{{ t("message.hello") }}</p>
      </div>
      //...
    ```

  - 其他复杂用法可以看文档：

    [文档链接](https://vue-i18n.intlify.dev/guide/advanced/composition.html#basic-usage)



## 错误监控
### 本项目实现方式
  - 基于sentry来实现的

### 完整解决方案
  [更多解决方式](/blog/build/optimization/errorCatch#基于-sentry-的错误监控)

## nuxt
### 说说你对 nuxt 的理解
  nuxt 是一个基于 vue 的能帮助我们快速构建 SSR 应用的框架。

  它有很多自动化的功能，比如：

  1. 会根据pages目录自动生成路由, (当然也可以在nuxt.config文件中自定义路由)

  2. 开箱即用的 SSR 功能,无需自己配置服务器

  3. code spliting: 把代码分割成更小的块，加快渲染速度

  4. 自动 import, tree-shaking, 优化js bundle等功能。

  5. 支持ts

  6. 配套的构建工具(vite,webpack)

  7. 数据获取实用程序

### 服务端渲染的好处有哪些？
  1. 更快的首屏加载，减少白屏时间：因为服务器会将完整的HTML发送到浏览器，可以立即使用
  2. 可访问性高：内容在初始页面加载时立即可用
  3. 更友好的SEO：拿到的是HTML,方便搜索引擎索引页面
  4. 在低性能设备上有更好的性能：因为减少了客户端下载和执行JS的数量。
  5. 更轻松的缓存：页面可以在服务端进行缓存，减少生成页面并将页面发送给客户端的时间。

### nuxt在项目中的使用
#### web项目
  1. web项目中的 ssr 主要是为了搜索引擎的爬取，所以针对这部分ssr，主要通过在页面中注释的方式来执行，形成ssr专用页面。

#### 低代码平台
  1. 项目主要分为两部分，一部分是整个低代码界面相关(editor-vue)，一部分是打包器-发布后的页面(baler-nuxt,基于nuxt的SSR应用)
  2. 会提供两套vue组件，一套用于在画布编辑区域进行编辑，一套用于在发布时使用。
  3. 做一些 SEO 相关的工作，一般为 TDK 信息等。
  
## 低代码
### lerna 进行项目管理
  - package.json 设置private:true; lerna.json 设置 useWorkspace: true;
  - 生成changelog: 
    * lerna-changelog是用来基于pull request提交时打的tag标签生成变更日志的CHANGELOG.md。用于开源项目中，合并他人提交的pr。如果只是团队内部项目，可以使用下面的conventional-changelog
    * 在lerna.json中配置changelog; changelog: {}
      a. repoHost;
      b. repo;
      c. label;
      d. md;
      e. cacheDir;
    * 使用lerna-changelog必须在从 github 获取Personal access tokens，然后本地环境变量添加GITHUB_AUTH。如果是私有仓库选择 scope repo，如果是公开的仓库选择 scope public_repo。

### graphql
  - 用 vue-apollo 在项目中集成 graphql
  - vue-apollo-link: apollo client 和 graphql 服务器; 
    * 可通过创建一个HTTPLINK来连接
    * 提供了与graphql进行数据交互的能力，连接提供了查询、变更和订阅等事件
  - apollo-link: 用来自定义 apollo-client 和graphql 服务器之间数据流的工具；用来修改graphql请求的控制流和graphql请求结果
  - graphql-tag: 是一个javascript文字模版标签，用于解析graphql查询。可以将 graphql 查询转换为标准的 AST 结构

### vue-appolo特点：
  - 集成graphql，自动更新，支持ssr，可以在vue组件中使用apollo
  - 方法中的使用：this.$apollo.query()

### 项目中关于graphql的使用
  - 集成了vue-apollo; vue-apollo-link; apollo-link
  - 用 apollo-link 添加了token；设置 context.header中的 Authorization; 返回response
  - 创建 errorLink 方法对 error进行处理


### 事件
  - 鼠标右键事件：
    * contextMenu
    * mounsedown: 0 - 左键; 1 - 滚轮; 2 - 右键
    * vue: @click.right  

### 项目路由：
  - 编辑器
    * files相关 默认: files/recents
    * builder相关 /builder/:id 具体的页面
    * demo相关：组件文档
  - 项目分包
  - nuxt渲染: 客户端页面的渲染

##### builder具体内容
  - head: 基础组件区域
    * 基础组件
      a. 通过click添加组件 调用createNode创建node
      b. 通过drag添加组件： mousedown ==> mouseup ==> click(最终在这里创建node)
        mousedown触发createDragNode事件 ===> 监听mousemove、mouseup事件，触发 onDragStart; onDragMove(判断x或者y move); onDragEnd(mouseup)    
    * 数据池: 显示对应的弹框
    * 事件编辑器: 显示对应的弹框
    * 图片库：显示对应的弹框
    * 自定义代码：显示对应的弹框
    * save: 从store中拿到页面的数据结构，并调用graphql接口存储
    * preview: 生成一个预览的链接 /domain/r-codeId
  - 画布区域
    * 左侧：node结构树
      a. 分为两个tab; 一个为当前构建的node树，一个可以用来保存当前的自定义代码或者自定义模版
      b. 对左侧容器宽度的调整
    * 中间：画布区域
      a. node节点的展示与操作
      b. 中间node渲染的为预览的组件，而不是真实的组件
    * 右侧：属性配置区域 
      a. 对齐方式
      b. 屏幕的适配
  - 无限画布实现方式：
    * 坐标转换
##### 项目分包
  * rocket-attributes: 对每个组件所能支持的属性的描述
  * rocket-blocks: 每个组件所能支持的属性的ui展示; block组件; 作为插件引入
  * rocket-dropzone: 画布区域所对应的组件以及uitls
  * rocket-storage: 文件系统的存储：比如上传图片，上传文件等
  * rocket-ui: ui库; 作为插件引入
  * rocket-utils: 函数库
  * rocket-widgets: 组件库; 作为插件引入

### baler-nuxt
  * 真实/预览页面的渲染

### nuxt生命周期以及各个目录的用法    

### yarn workspace
  * workspace 是 yarn 支持的一种特性，在 yarn 1.0.0 版本开始默认支持。该功能可以让我们在同一个代码仓库中管理多个 package，也就是 monorepo 的代码管理方式。
  * 很多大型的项目都包含有多个 package，比如 vue、react、babel、rollup等，如果每一个 package 都需要单独一个仓库，整体项目的工作流程将会变得异常复杂。采用了 monorepo 的做法，将所有 package 放在同一个代码仓库中，可以统一进行构建、发布，对于调试和依赖管理都很方便。
  * 使用 workspace 之后，yarn 会将 所有 package 的依赖统一安装在一起，提高了性能，同时节约了硬盘空间。同时，yarn 会在 node_modules 中生成 package 的软链接，指向 workspaces 中的最新代码文件，相对于 yarn link 来说，使用更加地快捷和方便。
  * 如果不同的 package 依赖同一个依赖的不同版本，为了解决冲突，低版本依赖安装在根目录，而其他依赖直接安装到对应 package 的 node_modules 下面。
  * 依赖包的依赖如果发生冲突，解决方法和 2 一致

### lerna 和 yarn workspace 的区别
  * yarn workspaces 主要解决的是多个项目依赖的问题; 而lerna是一个工具，除了解决依赖问题，还可以解决多个项目的运行、构建、发布等问题
  * https://github.com/pfan123/Articles/issues/73


## 工程化-CI/CD
  CI/CD 是一种持续的软件开发方法，我们可以持续构建、测试、部署、监视代码迭代更新等。
  
  有助于减少在错误代码或者失败版本开发新代码的机会。使得所部署的代码可以符合标准。

  **我们项目中的 CI/CD 主要包括代码合并，静态代码校验，单元测试，e2e 测试，上传图片资源，构建项目，部署项目，以及同步多语言翻译等多个功能。**

### 项目中 CI/CD 的使用
  **我们的项目中 CI/CD 主要包含了 prepare, test-and-build, images-and-assets, deploy-rnd, deploy, check, sync-lang 7个阶段**

  - prepare
    1. 动态设置git仓库
    2. 根据 tag 代码合并到指定分支
    3. 建立新的分支将代码同步到 preview 分支
    4. yarn 安装依赖
  - test-and-build
    1. yarn install安装依赖
    2. yarn lint 静态代码解析
    3. yarn test:unit 运行单元测试
    4. ssr 相关 test 和 build
    5. 使用 SonarScanner CLI 工具将代码提交到 SonarQube 服务器进行静态代码分析和质量评估。
    6. 进行 e2e 测试
    7. 构建项目并注入 env 目录，nginx 目录等
    8. 通过 storybook 构建文档
    9. 上传本地化基础语言包
  - images-and-assets
    1. 上传图片到指定地方
  - deploy-rnd
    1. 发布到 rnd
    2. 发布文档到 rnd
  - deploy
    1. 发布到生产灰度
    2. 发布到生产
  - check
    1. 检查缓存
  - sync-lang
    1. 拉取i18n语言翻译
    2. 同步语言翻译mr
    3. 上传语言  
