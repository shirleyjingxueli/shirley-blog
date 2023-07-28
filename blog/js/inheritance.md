# js-继承与原型链
## js中如何创建一个对象
### 字面量
```js
  let person = {
    name: 'shirley',
    age: 18,
    sayName: function(){
      console.log(this.name)
    }
  }
```
### 使用Object构造函数
```js
  let person = new Object();
  person.name = "shirley";
  person.age = 18,
  person.sayName = function(){
    console.log(this.name)
  }
```
### 工厂模式
```js
  // 工厂函数
  function createPerson(name, age){
    let person = new Object();
    person.name = name;
    person.age = age;
    person.sayName = function(){
      console.log(this.name)
    }
    return person
  }
  // 使用
  const person1 = createPerson("shirley", 18);
  const person2 = createPerson("mike", 26);
```
### 构造函数
```js
  // 构造函数
  function Person(name,age){
    this.name = name;
    this.age = age;
    this.sayName = function(){
      console.log(this.name)
    }
  }
  // 使用
  const person1 = new Person("shirley", 18)
  const person2 = new Person("mike", 26)
```
## 调用new操作符发生了什么
  - (1) 在内存中创建一个新对象。
  - (2) 这个新对象内部的[[Prototype]]特性被赋值为构造函数的 prototype 属性。
  - (3) 构造函数内部的 this 被赋值为这个新对象（即 this 指向新对象）。
  - (4) 执行构造函数内部的代码（给新对象添加属性）。
  - (5) 如果构造函数返回非空对象，则返回该对象；否则，返回刚创建的新对象
## 原型链
### 原型链例子
![Alt text](OOP.png)

## js中的继承方式
### 原型链继承
  ```js
    
  ```
### 构造函数继承
### 原型式继承
### 组合继承
### 寄生式继承
### 寄生组合式继承（最成熟的方法，也是现在库实现的方法）
### 混入方式继承
### es6的class继承

## es5 与 es6中继承的区别
  - es5中的继承是基于prototype或者构造函数的，es6中的继承是基于class和super的
    * es5-基于prototype
    ```js
      function Parent();
      function Child();
      Child.prototype = new Parent();
      Child.constructor = Child
      const child = new Child();
      Child.__proto__ === Function.prototype
    ```
   * es5-基于构造函数
    ```js
      function Parent () {}
      function Child () {
        Parent.apply(this, arguments)
      }
      var child = new Child();
      Child.__proto__ === Function.prototype
    ``` 
   * es6-基于super
   ```js
    class Parent{}
    class Child extends Parent {
      constructor(){
        super();
      }
    }
    Child.__proto === Parent;
   ``` 
  - this的构造顺序是不一样的。
    * es5的继承是先构造子对象，再产生的父对象，然后将父对象里面的属性复制到子对象中
    * es6是先用 ```super``` 创建的父对象，然后再创建的子对象

    这种差异导致es6的继承可以继承内置对象，通过 ```Child.__proto__``` 可以找到父类（Parent）