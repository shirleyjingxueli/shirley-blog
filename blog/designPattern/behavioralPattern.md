# 设计模式-行为型
## demo-行为型
### 命令模式
  * 请求以命令的形式包裹在对象中，并且传递给调用对象。调用对象寻找可以处理该命令的对象，并把命令传递给对象。该对象执行命令。将请求封装成对象，从而可以用不同的请求对客户进行参数化
  * 解决问题：解决行为请求者与行为实现者之间高耦合的关系
  * 何时使用：在某些场合，比如要对行为进行"记录、撤销/重做、事务"等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将"行为请求者"与"行为实现者"解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。
  * 实现方式：通过调用者调用接受者执行命令，顺序：调用者→命令→接受者。
  ```js
    // 命令
    class Command {
      constructor(receiver){
        this.receiver = receiver;
      }
      run(){
        this.receiver.execute();
      }
    }

    // 接收者
    class Receiver {
      execute(){
        console.log('execute')
      }
    }

    // 调用者
    class Operator {
      constructor(command){
        this.command = command;
      }
      run(){
        this.command.execute();
      }
    }

    // 使用：
    const command = new Command(new Receiver())
    const operator = new Operator(command)
  ```
### 模版模式
  * 定义一个操作中算法的骨架，将一些步骤延迟到子类中去实现。子类在不改变算法结构的情况下，可以定义该算法的某个特定步骤
  * 解决问题：一些方法通用，却在每个子类中都实现了一遍
  * 何时使用：有一些通用的方法
  * 实现方式：将通用的算法抽离出来
  * eg:
  ```js
    // 基础类
    class Device {
      powerOn() {
        console.log('打开电源');
      }
      login() {
        console.log('登录账号');
      }
      clickIcon() {
        console.log('点击开始游戏');
      }
      enterGame() {
        console.log('进入战场');
      }

      play() {
        this.powerOn();
        this.login();
        this.clickIcon();
        this.enterGame();
      }
    }

    // 对device类中的方法进行重写
    class BigScreenDevice extend Device {
      powerOn(){
        console.log(打开电源并链接到大屏幕)
      }
    }
  ```
### 观察者模式
  * 定义对象中一对多的依赖关系。当对象发生改变时，所有依赖于它的对象都会被通知更新
  * 解决问题：一个对象改变给其他对象通知的问题，而且要考虑到易用性和低耦合，保证高度的协作
  * 何时使用：一个对象（目标对象）的状态改变，所有的依赖对象（观察者）都要得到通知，进行广播通知。
  * 实现方式：在抽象的类中有一个arrayList存放观察者们
  * eg:
  ```js
    // 通过智能家居一键开始游戏
    // 抽象类
    class MediaCenter {
      constructor() {
        this.state = '';
        this.observers = [];
      }
      attach(observer) {
        this.observers.push(observer);
      }
      getState() {
        return this.state;
      }
      setState(state) {
        this.state = state;
        this.notifyAllobservers();
      }
      notifyAllobservers() {
        this.observers.forEach(ob => {
          ob.update();
        })
      }
    }

    // 观察者
    class observer {
      constructor(name, center) {
        this.name = name;
        this.center = center;
        this.center.attach(this);
      }
      update() {
        console.log(`${this.name} update, state: ${this.center.getState()}`);
      }
    }

    // 使用
    const center = new MediaCenter();
    const ps = new Observer('ps', center);
    const tv = new Observer('tv', center);

    center.setState('on');
  ```
### 职责链模式
  * 避免请求发送者与接收者耦合在一起，让多个对象都可以接收请求，将这些对象连成一条链，并且沿着这条链传递请求，直到有对象处理它为止。
  * 解决问题：职责链上的处理者只负责处理请求，客户只需要将请求发送到职责链上就可以，无需关心请求的处理细节和传递细节，所以职责链将请求的发送者和请求的处理者解耦了
  * 何时使用：在处理消息的时候需要过滤很多道
  * 实现方式：拦截的类都实现统一的接口
  * eg:
  ```js
    // 拦截类
    class Action {
      constructor(name) {
        this.name = name;
        this.nextAction = null;
      }
      setNextAction(action) {
        this.nextAction = action;
      }
      handle() {
        console.log(`${this.name}请审批，是否可以打游戏`);
        if (this.nextAction !== null) {
          this.nextAction.handle();
        }
      }
    }

    // 使用
    const dad = new Action('爸');
    const mom = new Action('妈');
    const wife = new Action('夫人');

    dad.setNextAction(mom);
    mom.setNextAction(wife);

    dad.handle();
  ```
  
### 注意事项
  - 模式场景
    1. 发出指令，中间层传递命令本身，命中包含执行对象 - 命令模式
    2. 通过模板定义执行顺序，做独立操作 - 模板模式
    3. 通过观察者，可以让被观察值统一发生变化，触发相应依赖值的统一更新 - 观察者模式
    4. 独立职责的单元通过链式执行，逐步操作流程 - 职责链

  - 实际应用
    1. 提交表单进行表单逐行校验，链式调用validate，依次执行 => 职责链
    2. echart准备工作：canvas、config、init、draw()，规划顺序执行 => 模板模式
    3. 调度器在接受到一组新的数据时候，解析数据，并且根据数据类型包裹在对象中传递到下级helper，helper再去执行相应操作 => 命令模式
    4. 输入框输入的值去判断下拉框显示与否 => 观察input设置show => 观察者模式  
