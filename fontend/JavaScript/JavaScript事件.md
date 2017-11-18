# JavaScript事件


参考：<br>
[JavaScript学习笔记整理（10）：Event事件](http://ghmagical.com/article/page/id/nXCnaSLsuyWd)<br>
[JavaScript 和事件](http://yujiangshui.com/javascript-event/)<br>
[阮一峰教程：事件模型 ](http://javascript.ruanyifeng.com/dom/event.html#toc15)
<br><br>
一个事件应该包含下面的几个内容：<br>
### 事件对象(event)
与特定事件相关且包含油管该事件详细信息的对象，再IE9+和其他现代浏览器，它`作为参数`传递给事件处理函数，所有事件对象都用来指定事件类型的type数学和指定事件目标的target属性<br>
### 事件类型
说明发生什么事件，比如"mousemove"<br>
### 事件目标
与事件相关的对象，不是事件对象。。。比如给一个button绑定了click事件，那么button就是click事件的目标<br>
### 事件处理函数
### 事件传播方式
在文档树结构中从下往上传播的成为”冒泡“，反之成为”捕获“<br>

#### 容易混淆的概念
`事件对象event`和`event.target`有什么区别<br>


## 事件注册
