# js-知识总结
## 数组
### 数组有哪些方法？
![Alt text](Array.png)
### 数组的哪些方法会生成新的数组?
  * concat()
  * map()
  * flat()
  * flatMap()
  * Array.from() 
   
## 捕获错误
### window.onerror 和 window.addEventListener("error") 的区别
  1. **触发方式：**

  window.onerror：是一个全局的事件处理器属性。当页面上发生未被 try...catch 捕获的 JavaScript 错误时，会触发这个事件处理器。

  window.addEventListener：是一个用于注册事件监听器的方法。它可以用于捕获特定类型的事件，包括错误事件，以及其他事件，如点击、键盘事件等。

  2. **错误信息：**

  window.onerror：提供了有关错误的详细信息，包括错误消息、文件名、行号和列号等。

  window.addEventListener：通常需要在事件处理函数中使用其他手段来获取错误信息，例如 try...catch。

  3. **事件类型：**

  window.onerror：只能捕获 JavaScript 运行时的错误，例如语法错误或运行时异常。

  window.addEventListener：可以用于捕获各种类型的事件，包括非 JavaScript 错误，如 DOM 事件等。

  4. **多次注册：**

  window.onerror：只能注册一个全局的错误处理器，因此它不能用于多个不同类型的错误处理。

  window.addEventListener：可以注册多个事件监听器，因此您可以同时处理多个不同类型的事件。

  5. **事件冒泡/捕获：**

  window.onerror：不支持事件冒泡或捕获，它仅用于捕获全局 JavaScript 错误。
  
  window.addEventListener：支持事件冒泡和捕获，可以在不同阶段捕获事件，如捕获阶段和冒泡阶段。