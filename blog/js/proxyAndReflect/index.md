# js-代理和反射
  代理和反射向开发者提供了**拦截并且向基本操作嵌入额外行为的能力**。

  可以给目标对象定义一个关联的代理对象，而这个对象可以作为抽象的目标对象来使用。

  在对目标对象的各种操作影响目标对象之前，可以在代理对象中对这些操作加以控制。

## Proxy
### 定义
  - Proxy 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。
### 特性  
  - 代理是对目标对象的抽象。
  - 可以用作目标对象的替身，但又完全独立于目标对象。目标对象既可以直接地操作，也可以通过代理来操作。但直接操作会绕过代理施予的行为。直接对目标对象操作也会反映在代理对象上，因为两个对象访问的是同一个值
  - 修改代理对象也会反映到目标对象上，因为值会转移到目标对象。
  - Proxy.prototype 是undefined, 因此不能用 instanceof 操作符。
  - 严格相等可以用来区分代理和目标

### 语法
    const p = new Proxy(target, handler);

  - target: 需要被proxy包装的目标对象
  - handler: 处理程序对象

  **缺少其中任何一个参数，都会抛出TypeError**

### 捕获器
  捕获器是在处理程序中定义的"基本操作的拦截器"

  每个处理对象包含零个或者多个捕获器，每个捕获器都对应一种基本操作，可以直接或者间接在代理对象上使用。

  每次在代理对象上调用这些操作时，代理可以在这些操作传播到目标对象之前调用捕获器函数，从而拦截并修改相应的行为。

#### 特性：
  - 所有的捕获器都可以访问相应的参数，基于这些参数可以重建被捕获方法的原始行为。
  - 所有可以捕获的方法都有对应的Reflect API方法。这些方法与捕获器拦截的方法具有相同的名称和函数签名，而且也具有与被拦截方法相同的行为。
  - 捕获处理程序行为必须遵循”捕获器不变式“。捕获器不变式因方法不同而不同。
  

#### 捕获器API
下面是部分API：
  - get(target, property, receiver)
    * target: 目标对象
    * property: 属性
    * receiver: Proxy或者继承的Proxy对象
  - set(target, property, value, receiver)
    * value: 被设置的属性的值
  - has(target, prop)
    * prop: 需要检查是否存在的属性
  - ownKeys(target)
    * 返回值：返回一个可枚举的对象

#### 创建一个可以捕获所有方法，然后将每个方法转发给对应的Reflect API的空代理
```js
  const target = {
    foo: "bar"
  }

  const proxy = new Proxy(target, reflect);

  console.log(target.foo);  // bar
  console.log(proxy.foo); // bar
```
### 撤销代理对象和目标对象的关联
  - Proxy.revocable(target, handler)
    用来撤销代理对象和目标对象之间的关联。撤销操作是不可逆的。
  - revoke() 撤销函数的操作是幂等的（操作多少次结果都是一样的）。
    调用撤销函数之后再操作代理对象，会抛出TypeError错误。
  ```js
    const target = {
      foo: function(){
        console.log(111)
      }
    };

    const handler =  {
      get(target, property, receiver){
        return target[property]
      }
    }
   
    const {proxy, revoke} = Proxy.revocable(target, handler);
    console.log(proxy.foo); // f(){console.log(111)}
    
    // 执行撤销操作。
    revoke();
    console.log(proxy.foo); // Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
  ```    
### 如何使用代理代理另一个代理对象
  代理可以拦截Reflect API的操作。所以可以通过使用一个代理来代理另一个代理对象，从而实现对目标代理对象的多层拦截网。

  ```js
    const o = {
      foo: "bar"
    }

    const proxy = new Proxy(o, {
      get(target, property, receiver){
        console.log("first proxy");
        return target[property]
      }
    });

    const proxy1 = new Proxy(proxy, {
      get(target, property, receiver) {
        return Reflect.get(...arguments) + "second proxy";
      }
    })
  
    console.log(proxy1.foo);
  ```  
### 代理的问题与不足
#### this问题: 
  - this通常指向调用它的对象
  ```js
    const target = {
      thisEqualsProxy(){
        console.log(this === Proxy)
      }
    }

    const proxy = new Proxy(target, {});

    target.thisEqualsProxy(); //false
    proxy.thisEqualsProxy(); // true
  ```
  - 但是当目标对象依赖于对象标识时，这种指向会出现问题
  ```js
    const wm = new WeakMap();

    class User { 
      constructor(userId) { 
        wm.set(this, userId); 
      } 
      set id(userId) { 
        wm.set(this, userId); 
      } 
      get id() { 
        return wm.get(this); 
      } 
    }
    const user = new User(123); 
    console.log(user.id); // 123 
    
    const userInstanceProxy = new Proxy(user, {}); 
    console.log(userInstanceProxy.id); // undefined

    // 这是因为 User 实例一开始使用目标对象作为 WeakMap 的键，代理对象却尝试从自身取得这个实例。
    // 要解决这个问题，就需要重新配置代理，把代理 User 实例改为代理 User 类本身。
    // 之后再创建代理的实例就会以代理实例作为 WeakMap 的键了
    const UserClassProxy = new Proxy(User, {}); 
    const proxyUser = new UserClassProxy(456); 
    console.log(proxyUser.id); // 456
  ```
#### 代理与内部槽位
  代理与内置引用类型（比如 Array）的实例通常可以很好地协同，但有些 ECMAScript 内置类型可能会依赖代理无法控制的机制，结果导致在代理上调用某些方法会出错。

  一个典型的例子就是 Date 类型。根据 ECMAScript 规范，Date 类型方法的执行依赖 this 值上的内部槽位[[NumberDate]]。代理对象上不存在这个内部槽位，而且这个内部槽位的值也不能通过普通的 get()和 set()操作访问到，于是代理拦截后本应转发给目标对象的方法会抛出 TypeError：
  ```js
    const target = new Date(); 
    const proxy = new Proxy(target, {}); 
    console.log(proxy instanceof Date); // true 
    proxy.getDate(); // TypeError: 'this' is not a Date object 

    // 可以代理Date构造函数来优化：
    const DateClassProxy = new Proxy(Date, {});
    const dateInstance = new DateClassProxy();
    console.log(dateInstance.getDate()); //可以正确输出
  ```


## Reflect
### 定义
  Reflect是js中的内置对象，它提供拦截js操作的方法
### 特性
  - Reflect中所有的属性和方法都是静态的。
  - Reflect API不仅限于捕获处理程序
  - 很多 Reflect 方法会返回称作”状态标记“的返回值，返回值为布尔值。表示操作是否执行成功。
    
    会返回状态标记的方法：
    * Reflect.defineProperty()
    * Reflect.perventExtensions()
    * Reflect.setPrototypeOf()
    * Reflect.set()
    * Reflect.delectProperty();
  ```js
    const o = {}

    // 未优化前代码
    try {
      Object.defineProperty(o, foo, {value: "bar"});
      console.log("success");
    } catch (e) {
      console.log(e)
    }

    // 使用Reflect优化过后
    if (Reflect.defineProperty(o, foo, {value: "bar"})) {
      console.log ("success")
    } else {
      console.log("fail")
    }

  ```
  - 只有通过操作符才能完成的操作
    * Reflect.get() 可以代替对象属性访问操作符
    * Reflect.set() 可以代替 ```=``` 赋值操作符
    * Reflect.has()：可以替代 in 操作符或 with()。 
    * Reflect.deleteProperty()：可以替代 delete 操作符。
    * Reflect.construct()：可以替代 new 操作符。
  - 其他方法：
    * Reflect.apply();有时候为了防止函数上定义了apply方法，可以调用Function原型上的apply方法，这种方法可以使用Reflect.apply()方法代替。
    ```js
      Function.prototype.apply.call(myFunc, this, argumentList)

      // 上面的代码可以替代为：
      Reflect.apply(myFunc, this, argumentsList)
    ```  

## 代理模式
### 代理模式的应用
  - **跟踪属性的访问、设置**

    通过捕获 get、set 和 has 等操作，可以知道对象属性什么时候被访问、被查询。把实现相应捕获器的某个对象代理放到应用中，可以监控这个对象何时在何处被访问过

  - **隐藏属性**

    代理的内部实现对外部代码是不可见的，因此要隐藏目标对象上的属性也轻而易举。

    可以拦截对对象的读取和写入操作，并根据特定的规则进行权限验证，从而保护敏感数据。

  - **属性验证**

    因为所有赋值操作都会触发 set()捕获器，所以可以根据所赋的值决定是允许还是拒绝赋值。

  - **函数与构造函数的参数验证**

    跟保护和验证对象属性类似，也可对函数和构造函数参数进行审查。

  - **数据绑定与可观察对象**
  
    通过代理可以把运行时中原本不相关的部分联系到一起。这样就可以实现各种模式，从而让不同的代码互操作。

  - **函数节流和防抖：**

    通过使用Proxy对象，可以对函数进行包装，实现函数的节流（Throttling）和防抖（Debouncing）。

    可以在函数执行前后进行拦截和处理，控制函数的调用频率，避免频繁触发事件或请求。

  - **缓存和延迟加载：**

    使用Proxy对象可以实现缓存的功能，对某些计算密集型或耗时的操作进行缓存，避免重复计算或请求。

    可以在对象属性被访问时，判断是否已经存在缓存，如果存在则直接返回缓存结果，否则进行计算或请求并更新缓存。

  - **数据格式化和转换：**

    可以使用Proxy对象对数据进行格式化、验证和转换，例如将日期格式化、将数据类型进行转换等。

    可以在数据读取和写入时，对数据进行拦截和处理，实现自定义的数据转换逻辑。   

## 小结
  从宏观上看，代理是真实 JavaScript 对象的透明抽象层。代理可以定义包含捕获器的处理程序对象，而这些捕获器可以拦截绝大部分 JavaScript 的基本操作和方法。在这个捕获器处理程序中，可以修改任何基本操作的行为，当然前提是遵从捕获器不变式。

  与代理如影随形的反射 API，则封装了一整套与捕获器拦截的操作相对应的方法。可以把反射 API看作一套基本操作，这些操作是绝大部分 JavaScript 对象 API 的基础。

  代理的应用场景是不可限量的。开发者使用它可以创建出各种编码模式，比如（但远远不限于）跟踪属性访问、隐藏属性、阻止修改或删除属性、函数参数验证、构造函数参数验证、数据绑定，以及可观察对象。           