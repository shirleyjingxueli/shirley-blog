# 设计模式-结构型
## demo—创建型
### 适配器模式
  * 两个不兼容的接口之间的桥梁。将一个接口转换为客户希望的另一个接口，使得原本不兼容的两个接口可以工作。
  * 解决问题：需要将```现存的接口```放入到```新的环境```中，但是```现存的接口无法满足新的接口需求```。
  * 何时使用：
    1. 现有的类不符合需求
    2. 希望建立一个重复使用的类，这个类要与一些关联不大的类共同工作，包括未来可能引入进来的类。这些源类不一定有一致的接口。
    3. 通过接口转换，将一个类插入到另一个类中
  * eg:
    ```js
      // 买了一个港版的ps，需要在国内使用，需要使用插座转换器
      // 原有类
      class HKDevice {
        getDevicePlug{
          rerturn '港版插头'
        }
      }

      // 适配器
      class PlugTransform {
        constructor() {
          this.device = new HKDevice();
        }
        getDevicePlug(){
          return this.device.getDevicePlug() + '插头转换器'
        }
      }
      
      // 使用
      const newDevice = new PlugTransform();
      console.log(newDevice.getDevicePlug());
    ```  
  * 实现方式：适配器继承或者依赖已有的对象，实现需要的接口
### 装饰器模式
  * 允许对一个现有的对象添加功能，同时又不改变其结构。用于动态的给一个对象添加额外的功能
  * 解决问题：解决使用继承的方式扩展类时，随着扩展功能的增多，子类的膨胀问题
  * 何时使用：在不想增加子类的情况下，对类进行扩展
  * 实现方式：将不同的功能划分，同时继承装饰者模式
  * eg:
  ```js
    // 创建一个shape类，并且创建一个装饰器，可以给shape设置border
    class Shape {
      draw(){
        console.log("draw shape")
      }
    }

    //装饰器
    class ShapeDecorator {
      constructor(shape){
        this.shape = shape
      }
      draw(){
        console.loe('decorator draw shape')
        this.shape.draw();
        this.setBorderColor()
      }
      setBorderColor(){
        console.log(this.shape + '设置颜色后的shape')
      }
    }
    
    //使用
    const shape = new Shape();
    const shapeDecorator = new ShapeDecorator(shape);
    shapeDecorator.draw();
  ```

### 代理模式
  * 一个类代表另一个类的功能。为其他对象提供一种代理来控制对这个对象的访问
  * 解决问题：直接访问对象会给使用者或者系统造成很多麻烦，可以在访问对象时，添加一个访问层
  * 何时使用：想要在访问一个类时做一些控制
  * 实现方式：添加中间层
  * eg:
  ```js
    // 通过对游戏玩家年龄的控制，从而控制对游戏的访问。
    // 游戏玩家类
    class Player {
      constructor(age){
        this.age = age;
      }
    }

    // 游戏类
    class Game {
      play(){
        console.log('玩游戏')
      }
    }

    // 游戏代理
    class GameProxy {
      constructor(player) {
        this.player = player;
        this.game = new Game();
      }
      play(){
        this.player.age >= 18 ? this.game.play() : console.log('未成年不能玩游戏S')
      }
    } 

    // 使用
    const player = new Player(16);
    const gameProxy = new GameProxy(player);
    gameProxy.play()
  ```

### 注意事项
  - 模式场景
    1. 中间转换参数、保持模块间独立的时候 - 适配器模式
    2. 附着于多个组件上，批量动态赋予功能的时候 - 装饰器模式
    3. 将代理对象与调用对象分离，不直接调用目标对象 - 代理模式

  - 实际应用
    1. 两个模块：筛选器和表格，需要做一个联动。但筛选器的数据不能直接传入表格，需要做数据结构转换 => 模块之间独立，需要做数据结构转换 => 适配器
    2. 目前有按钮、title、icon三个组件。希望开发一个模块，让三个组件同时具备相同功能 => 套一层装甲对于每个组件有统一的能力提升，且可以动态添加功能进行拓展 => 装饰器模式
    3. ul中多个li，每个li上的点击事件 => 利用冒泡做委托，事件绑定在ul上 => 代理 