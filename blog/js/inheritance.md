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