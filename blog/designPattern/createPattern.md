#### demo-创建型
  + 工厂模式
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
  + 建造者模式
  + 单例模式