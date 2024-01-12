# CSS样式隔离
  - CSS Modules
  - CSS-in-JS
  - scoped css
CSS Modules 和 Scoped CSS 是用于实现 CSS 的模块化和作用域限定的两种不同的技术。

CSS Modules:

  CSS Modules 是一种使用构建工具或预处理器插件来实现的技术，如Webpack、Rollup、Parcel等。
  它将每个 CSS 文件视为一个独立的模块，并为每个类名生成唯一的标识符，以确保模块之间的样式不会冲突。
  使用 CSS Modules，类名是通过引用样式模块对象的属性来应用的，例如 styles.className。
  CSS Modules 提供了模块化的样式管理，可以避免全局样式冲突，并且使样式的重用和维护更加容易。
  Scoped CSS:

  Scoped CSS 是 Vue.js 框架中的一个特性，使用 ```<style scoped> ```语法来实现。
  它通过将样式规则转化为带有唯一标识符的类名或属性选择器，将样式限定在当前组件的作用域内。
  使用 Scoped CSS，样式规则只会应用于当前组件及其子组件的元素，不会影响其他组件或全局样式。
  Scoped CSS 提供了组件级别的样式隔离，避免了样式冲突，并且使样式与组件的关联更加紧密。