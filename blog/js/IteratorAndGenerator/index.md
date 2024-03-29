# js-迭代器和生成器
  迭代的意思是按照顺序多次重复执行一段代码，通常有明确的终止条件。ES6中添加了iterator和generator，能够更清晰、高效的实现迭代。

## es6之前的迭代
### 循环和其他辅助结构
  循环是迭代的基础机制。因为它可以定义迭代的次数，以及每次迭代要执行的操作。每次循环会在下一次迭代之前完成，迭代是按照预先定义好的顺序进行的。
### 循环迭代的缺点
  - 迭代之前需要事先知道如何使用数据结构
  - 遍历顺序并不是数据结构固有的

## es6之后的迭代
  es6之后,开发者无需知道迭代模式就可以进行迭代操作

### 迭代器模式
  es6中的迭代器模式，可以把有些数据结构称为”可迭代对象“，因为它们实现了正式的Iterable接口，而且可以通过迭代器Iterator消费。**任何实现Iterator接口的数据结构都可以被实现Iterator接口的对象消费。**

  可迭代对象是一种抽象的说法。可迭代对象可以理解成数组或者集合这样的集合类型的对象。它们包含的元素都是有限的，而且都具有无歧义的遍历顺序。

  **可迭代对象不一定是集合对象，也可以是仅仅具有类似数组行为的其他数据结构。**

  **迭代器是按需创建的一次性对象。每个迭代器都会关联一个可迭代对象，而迭代器会暴露迭代其关联可迭代对象的API。迭代器无需了解与其关联的可迭代对象的结构，只需要知道如何取得连续的值。**这种概念上的分离正是Iterable和Iterator的强大之处。

### 实现Iterable接口
  实现Iterable接口需要具备两种能力：
  - 支持迭代的自我识别能力
  - 创建实现Iterator对象接口的能力

  在es6中，暴露 ```[Symbol.iterator]``` 作为键的属性，这个属性引用一个迭代工厂函数，调用这个函数返回一个新迭代器。
  
#### es6中实现了Iterable接口的类型
  - String
  - Array
  - Map
  - Set
  - arguments 对象
  - NodeList 等DOM集合类型
  - Generator

#### 判断是否存在默认迭代器属性
  可以通过判断是否有 ```Symbol.Iterator```属性

#### 接收可迭代对象的原生语言特性
  在实际编写代码过程中，不需要调用 Symbol.Iterator()来生成迭代器。实现可迭代协议的所有类型都会自动兼容接收可迭代对象的任何语言特性。

  接收可迭代对象的原生语言特性包括：
  - for-of 循环
  - 数组解构
  - 扩展操作符
  - Array.from()
  - 创建集合
  - 创建映射
  - Promise.all() 接收由Promise组成的可迭代对象
  - Promise.race() 接收由Promise组成的可迭代对象
  - yield* 操作符，在生成器中使用

  这些原生语言结构都会在后台调用提供的可迭代对象的这个工厂函数，从而创建一个迭代器。
  
  如果对象原型链上的父类实现了 Iterable接口，那这个对象也实现了这个接口

### 迭代器协议
  - **迭代器是一种一次性使用的对象，用于迭代与其关联的可迭代对象。**

  - **每个迭代器表示对可迭代对象的一次性有序遍历。**不同迭代器实例相互之间没有联系，只会独立地遍历可迭代对象。

  - 迭代器API使用next()方法在可迭代对象中遍历数据。每次成功调用next(),都会返回一个 IteratorResult 对象，其中包含迭代器返回的下一个值。若不调用next(), 则无法知道迭代器的当前位置。

  next()方法返回的迭代器对象IteratorResult对象包含两个属性：
  - done
    * 布尔值。表示是否还可以再次调用next()取得下一个值
    * done为true状态称为”耗尽“
  - value
    * 包含可迭代对象的下一个值（done为false）
    * undefined(done为true)

  - 迭代器并不与可迭代对象某个时刻的快照绑定，而仅仅是使用游标来记录遍历可迭代对象的历程。如果可迭代对象中间被修改了，那么迭代器也会反应相应的变化
    ```js
      let arr = ["foo", "baz"];
      let iter = arr[Symbol.Iterator]();

      console.log(iter.next()); //{done:false, value: 'foo'};

      arr.splice(1, 0, "bar");
      console.log(iter.next()) // {done:false, value: "bar"}
      console.log(iter.next()) // {done:false, value: "baz"}
      console.log(iter.next()) // {done: true, value: undefined}
    ``` 

  - 迭代器维护着一个指向可迭代对象的引用，因此迭代器会阻止垃圾回收机制回收可迭代对象。

### 提前终止迭代器
  可选的return()方法用于指定在迭代器提前关闭执行的逻辑。
  
  提前关闭迭代器的情况可能包括：
  - for-of 循环通过 break、continue、 return 或者trow提前退出
  - 解构操作并未消费所有值。

  return() 方法必须放回一个有效的IteratorResult对象。简单情况下，可以只返回 ```{done:true}```。因为这个值只会用在生成器的上下文中。

  如果迭代器没有关闭，则还可以从上次离开的地方继续迭代。

  **因为return()方法是可选的，所以并非所有的迭代器都可以关闭。要知道某个迭代器是否可以关闭，可以测试这个迭代器实例的return属性是不是函数对象。**。不过，仅仅给一个不可关闭的迭代器增加这个方法并不能让它变成可关闭的。这是因为调用 return()不会强制迭代器进入关闭状态。即便如此，return()方法还是会被调用。

### 自定义迭代器
  ```js

    class Counter { 
      constructor(limit) { 
        this.limit = limit; 
      } 
      [Symbol.iterator]() { 
        let count = 1, 
        limit = this.limit; 
        return { 
          next() { 
            if (count <= limit) { 
              return { done: false, value: count++ }; 
            } else { 
              return { done: true, value: undefined }; 
            } 
          };
          return() { 
            console.log('Exiting early'); 
            return { done: true }; 
          }
        }; 
      } 
    } 
    let counter = new Counter(3); 
    for (let i of counter) { console.log(i); } 
    // 1 
    // 2 
    // 3 
  ```      

## 生成器
  生成器拥有在一个函数内部暂停和恢复代码执行的能力。

### 语法
  函数名称前面加 * 号。可以定义函数的地方就可以定义生成器。
  ```js
    function * generatorFn(){}
  ```
#### 方法：
  - next()
  - return(): 用于提前终止迭代器。强制生成器进入关闭状态。所有的生成器都有return()方法。
  - throw() : 在暂停的时候讲一个提供的错误注入到生成器对象中。如果错误未被处理，生成器就会被关闭。

### yield关键字
  生成器通过 yield 中断执行。

  yield关键字可以让生成器停止和开始执行。生成器函数在遇到 yield 关键字之前正常执行。 遇到这个关键字之后，执行会停止，函数作用域的状态会被保留。停止执行的生成器函数只能通过在生成器对象上调用 next() 方法来恢复执行。

#### 特性
  - 通过 yield 关键字退出的生成器会停留在 done:false 状态；通过return关键字退出的生成器会函数会处于 done:true 状态。
  - 生成器函数内部的执行流程会针对每个生成器对象区分作用域。在一个生成器上调用 next() 并不会影响其他生成器。
  - yield 关键字只能在生成器内部使用，用在其他地方会抛出错误。

### 生成器的使用
#### 生成器对象作为可迭代对象
```js
  function* generatorFn(){
    yield 1;
    yield 2;
    yield 3
  }

  for (const item of generatorFn()) {
    console.log(item)
  }
  // 1 2 3
```
#### yield 实现 输入和输出
```js
  function* generatorFn() {
    return yield "foo"
  }

  let generatorObj = generatorFn();

  console.log(generatorObj.next()); //{done:false, value: foo};
  console.log(generatorObj.next("bar")) // {done: true, value: "bar"}
```
#### 产生可迭代对象
```js
  // 等价的 generatorFn： 
  // function* generatorFn() { 
  // for (const x of [1, 2, 3]) { 
  // yield x; 
  // } 
  // } 
  function* generatorFn() { 
    yield* [1, 2, 3]; 
  } 
  let generatorObject = generatorFn(); 
  for (const x of generatorFn()) { 
  console.log(x); 
  } 
  // 1 
  // 2 
  // 3
  // yield*实际上只是将一个可迭代对象序列化为一连串可以单独产出的值，所以这跟把 yield放到一个循环里没什么不同。
```
#### 使用yield* 实现递归
```js
  function* nTimes(n) { 
    if (n > 0) { 
      yield* nTimes(n - 1); 
      yield n - 1; 
    } 
  } 
  for (const x of nTimes(3)) { 
    console.log(x); 
  } 
  // 0 
  // 1 
  // 2
```
## 小结
  迭代器是一个可以由任意对象实现的接口，支持连续获取对象产出的每一个值。任何实现 Iterable接口的对象都有一个 Symbol.iterator 属性，这个属性引用默认迭代器。默认迭代器就像一个迭代器工厂，也就是一个函数，调用之后会产生一个实现 Iterator 接口的对象。

  迭代器必须通过连续调用 next()方法才能连续取得值，这个方法返回一个 IteratorObject。这个对象包含一个 done 属性和一个 value 属性。前者是一个布尔值，表示是否还有更多值可以访问；后者包含迭代器返回的当前值。这个接口可以通过手动反复调用 next()方法来消费，也可以通过原生消费者，比如 for-of 循环来自动消费。

  生成器是一种特殊的函数，调用之后会返回一个生成器对象。生成器对象实现了 Iterable 接口，因此可用在任何消费可迭代对象的地方。生成器的独特之处在于支持 yield 关键字，这个关键字能够暂停执行生成器函数。使用 yield 关键字还可以通过 next()方法接收输入和产生输出。在加上星号之后，yield 关键字可以将跟在它后面的可迭代对象序列化为一连串值。

