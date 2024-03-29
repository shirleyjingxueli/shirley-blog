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

## Math
### Math.pow(base, exponent)

  返回基数（base）的指数（exponent）次幂，即 base^exponent

  ```Math.pow(10,4)``` 求10^4的值

### Math.max()
  返回作为输入参数的最大数字，如果没有参数，则返回 ```-Infinity```

  ```Math.max(1,2,3)```

### Math.min()

  返回作为输入参数的数字中最小的一个，如果没有参数，则返回 ```Infinity```
  
  ```Math.min(1,2,3)```

## Object.is()
### 特性
  - Object.is() 与 == 运算符不等价。**不会对两个值进行类型转换**
  - Object.is() 与 === 运算符不等价。**在处理带符号的 0 和 NaN时不同。=== 或者（==）运算符会将 ```-0 和 +0 视为相等，会将 NaN视为不相等。```**

### Object.is() 判断两个值相同的场景
  - 都是 undefined
  - 都是 null
  - 都是 true 或者 都是 false
  - 都是长度相同、字符相同、顺序相同的字符串
  - 都是相同的对象（两个值都引用内存中的同一对象）
  - 都是 BigInt 且具有相同的数值
  - 都是 symbol 且引用相同的 symbol 值
  - 都是数字且
    - 都是 +0
    - 都是 -0
    - 都是 NaN
    - 都是相同的值，非零且都不是 NaN

### 例子
```js
    // 案例 1：评估结果和使用 === 相同
    Object.is(25, 25); // true
    Object.is("foo", "foo"); // true
    Object.is("foo", "bar"); // false
    Object.is(null, null); // true
    Object.is(undefined, undefined); // true
    Object.is(window, window); // true
    Object.is([], []); // false
    const foo = { a: 1 };
    const bar = { a: 1 };
    const sameFoo = foo;
    Object.is(foo, foo); // true
    Object.is(foo, bar); // false
    Object.is(foo, sameFoo); // true

    // 案例 2: 带符号的 0
    Object.is(0, -0); // false
    Object.is(+0, -0); // false
    Object.is(-0, -0); // true

    // 案例 3: NaN
    Object.is(NaN, 0 / 0); // true
    Object.is(NaN, Number.NaN); // true
```

## JSON
### JSON.stringify
#### **JSON.stringify()将值转换为相应的 JSON 格式：**

  1. 转换值如果有 toJSON() 方法，该方法定义什么值将被序列化。
  2. 非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。
  3. 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
  4. undefined、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 null（出现在数组中时）。函数、undefined 被单独转换时，会返回 undefined，```如JSON.stringify(function(){}) or JSON.stringify(undefined).```
  5. 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
  6. 所有以 symbol 为属性键的属性都会被完全忽略掉，即便 replacer 参数中强制指定包含了它们。
  7. Date 日期调用了 toJSON() 将其转换为了 string 字符串（同 Date.toISOString()），因此会被当做字符串处理。
  8. NaN 和 Infinity 格式的数值及 null 都会被当做 null。
  9. 其他类型的对象，包括 Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性。

####  **JSON.stringify() 出现异常的情况：**
  1. 当在循环引用时会抛出异常TypeError ("cyclic object value")（循环对象值）
  2. 当尝试去转换 BigInt 类型的值会抛出TypeError ("BigInt value can't be serialized in JSON")（BigInt 值不能 JSON 序列化）.