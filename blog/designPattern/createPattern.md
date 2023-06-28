## demo-创建型
  ### 工厂模式
  * 定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。
  * 解决问题：解决接口的选择问题
  * 何时使用：不同条件下，生成不同的对象
  * 实现方式：工厂抽象出对象，创建的过程在子类中实现
  * eg1：
    ```
      //抽象出的游戏工厂类
      class gameFactory{
        constructor(name){
          this.name = name;
        }
        run(){
          console.log('run');
        }
        jump(){
          console.log('jump');
        }
      }

      // 创建游戏类
      class gameStore {
        create(name){
          return new gameFactory(name)
        }
      }

      // 使用方法：
      const store = new gameStore()
      const lol = store.create('LOL');
      const genshin = store.create('GENSHIN')
    ```
    * eg2：
    ```
      // 通用游戏工厂类
      class gameFactory{
        constructor(name){
          this.name = name;
        }
        run(){
          console.log('run');
        }
        jump(){
          console.log('jump');
        }
      }

      // lol工厂类
      class Lol extends gameFactory{
        constructor(name){
          super(name)
        }
        kill(){
          console.log(kill)
        }
      }

      // 创建游戏
      class gameStore{
        create(options){
          if (options.type === 'lol') {
            return new Lol(options.name)
          } else {
            return new GameFactory(options.name)
          }
        }
      }
      
      // 调用方法：
      const gameStore = new gameStore();
      const lol = gameStore.create({type: 'lol', name:'lol'});
      const genshin = gameStore.create({type: 'genshin', name: 'lol'})

    ```
### 建造者模式
  * 将一个复杂的功能拆分成多个简单的模块，独立运行。将构建和表示分离，使用相同的构建过程，可以构建出不同的展示。
  * 解决问题：解决复杂对象创建过程中，各个部分需求变动很大，但是将各个部分组合在一起却相对较稳定。
  * 何时使用：一些基本部件不常发生变化，组合经常发生变化。
  * 实现方式：建造者 --> 创建和提供实例; 导演 --> 组合实例之间的依赖关系
  * eg:
    ```
      // 游戏具有基础功能（跑、跳）和 皮肤设置功能，可以将游戏和皮肤一起打包售卖
      // 构建者：游戏
      class Game {
        constructor(name) {
          this.name = name;
        }
        run(){
          console.log("run")
        }
        jump(){
          console.log(jump)
        }
      }

      // 构建者：皮肤
      class Skin {
        constructor(name){
          this.name = name;
        }
        getSkinName(){
          return this.name
        }
      }
      
      // 导演：游戏打包
      class gamePackager {
        constructor(name){
          this.game = new Game(name);
          this.skin = new Skin(name);
        }
        init(){
          this.game.run();
          this.skin.getSkinName();
        }
      }

      // 调用方法：
      const gamePackager = new GamePackager('testgame');
      gamePackager.init();
    ```
### 单例模式
  * 保证一个类只有一个实例，并且提供一个访问它的全局访问点
  * 解决问题：解决一个全局类频繁的创建和销毁问题
  * 何时使用：想控制实例的数量，节省系统资源
  * 实现方式：判断系统是否已经存在这个实例，存在的话直接返回，否则则创建。
  * eg:
    ```
      // 全局只希望存在一个modal
      class Modal{
        constructor(){
          this.instance = null
        }
        init(){
          if (this.instance){
            return this.instance;
          } else {
            this.instance = new Modal();
            return this.instance;
          }
        }
      }

      //使用方法：
      const modalInstance = new Modal().init();
    ```