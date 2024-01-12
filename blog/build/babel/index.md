# 前端工程化-Babel
## Babel
### Babel 是什么？

  Babel 是一个工具链，主要用于将 ECMAScript 2015+ 代码转换为当前和旧版本浏览器或环境中向后兼容的 JavaScript 版本。以下是 Babel 可以为您做的主要事情：

  - 转换语法
  - 目标环境中缺少的 Polyfill 功能（通过第三方 Polyfill，例如 core-js）
  - 源代码转换（codemods）

### Babel 的原理是什么？
  ![Alt text](image/babel.png)
  
  babel 的转译过程也分为三个阶段：

  1. 解析 Parse: 将代码解析生成先后向语法树（AST），即词法解析与语法解析的过程。

  2. 转换 Transform: 对于 AST 进行变化一系列的操作，babel 接受得到 AST 并通过 babel-traverse 对其进行遍历，在此过程中进行添加、更新及移除等操作。

  3. 生成 Generate: 将变换后的 AST 再转换为 JS 代码，使用到的模块是 babel-generator

  