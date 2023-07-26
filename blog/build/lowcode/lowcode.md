# 低代码知识
## 拖拽
### 浏览器的拖拽事件
  - **drag**
    drag事件在用户拖拽元素或者选中的文本时，每隔几百毫秒执行一次
    * dragstart：在用户开始拖拽元素或者选中的文本时开始调用
    * dragend：在 ```拖放``` 事件 ```结束```时触发（通过释放鼠标或者esc键触发）
    * dragenter：在拖拽元素或者选中的文本进入一个有效的放置目标时触发
    * dragleave：在拖拽元素或者选中的文本离开一个有效的放置目标时触发
    * dragover：在可拖动的元素或者被选择的文本被拖进一个有效的放置目标时（每几百毫秒）触发
  - **drop**
    * drop：在元素或者被选择的文本被放置在一个有效的放置目标时触发 
  - **dataTransfer**
    * 用于保存拖动并放下（drag和drop）过程中的数据，它可以保存一项或多项数据。这些数据可以是一种或者多种数据类型
    * 标准属性：
      * dropEffect: 获取当前选定的拖放操作类型或者设置为一个新的类型。值：none，copy，link，move。它会影响在拖拽过程中光标的手势。
      * effectAllowed: 提供所有可以操作的类型。值：none，copy，copyLink，linkMove，all，uninitialized
      * files: 包含数据传输中可用的所有本地文件的列表。如果拖动操作不涉及拖动文件，则此属性为空列表。
      * items: 提供一个包含所有拖动数据列表的 ```DataTransferItemList``` 对象
      * types: 一个提供 ```dragstart``` 实践中设置的格式的 strings 数组。 
  - **鼠标拖放事件**
    * mousedown
    * mousemove
    * mouseup     
### 拖拽事件触发流程
  1. 拖拽元素的dragstart事件
  2. 目标区域的dragenter和dragleave事件(如果没有进入目标区域，则没有)
  3. 目标区域的dragover事件
  4. 目标区域的drop事件
  5. 拖拽元素的dragend事件
### 拖拽的实现
  - 拖拽元素监听dragstart, dragend事件
  - 目标区域监听drop事件，通过e.target拿到最终落入的目标元素 
  - 对于最终落入的目标元素的判断：
    * 找到屏幕上所有的容器节点
    * 拿到距离拖拽元素最近的容器节点的id：可能为当前domId,否则则判断拖拽元素是否在容器节点内，在则返回容器id，否则则在递归查找容器节点的父节点。
## mouseEvent.button
  - 值
    * 0：主按键，通常指主键或者左键
    * 1：辅助按键，通常指鼠标滚轮中建
    * 2: 次按键，通常指鼠标右键
    * 3: 第四个按钮，通常指浏览器后退按钮
    * 4: 第五个按钮，通常指浏览器前进按钮
## document.elementFromPoint、document.elementsFromPoint
  - 解释：
    * elementFromPoint(x,y): 返回给定坐标点下最上层的element元素,如果给定的坐标点在文档的可视范围外，或者两个坐标都是负数事，返回null;
    * elementsFromPoint(x,y): 返回在给定坐标点下的html元素数组
## 获取元素的中心点
  - 获取元素的大小及其相对于视口的位置: getBoundingClientRect();
  ![Alt text](<截屏2023-07-24 下午3.20.51.png>)
  - 元素中心点位置：(left+right)/2, (top+bottom)/2
## 元素resize
  - 通过mousedown，mousemove，mouseup事件来控制
    * mousedown时调用resize方法触发resize操作，在resize方法中监听mousemove以及mouseup事件，mousemove时触发change操作,
    mouseup时触发resize-end操作 
## 键盘事件的绑定
  - 生成键盘事件组件,在组件渲染完成后进行键盘事件绑定，在组件销毁前进行键盘事件销毁
  - 通过设置不同的快捷键，来派发不同的store事件，从而触发不同的动作
## 吸附对齐功能的实现
  - 记录组件的【左、中、右】【上、中、下】的位置  
  - 定义吸附的距离
  - 拖拽组件时，寻找离拖拽组件最近的线，并显示
  - 当在吸附距离范围内时，组件的位置定义为可吸附组件的位置  