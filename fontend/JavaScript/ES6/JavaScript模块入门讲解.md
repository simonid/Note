# JavaScript模块入门讲解


什么是模块？模块可以视为实现某个特定功能的一组方法。一个有特定功能的函数可以视作一个模块，封装了多个函数的对象也可以视作一个模块<br>
<b>原始写法：</b><br>
将多个含特定功能的函数(每个函数都是该模块的成员)简单地放在一起，比如：<br>
```js
function fun1(){...}
function fun2(){...}
```
这样就组成了一个模块。这样做的缺点很明显：全局变量被污染了，和其他模块可以会有变量名冲突，而且模块间的关系不清晰<br>
<b>封装到一个对象：</b><br>
将多个特定功能的函数放到一个对象内，比如：<br>
```js
var module = new Object({
    _status = true,
    fun1:function(){...},
    fun2:function(){...}
});
```
调用：`module.fun2()`。这样做的缺点在于会暴露所有模块成员，内部状态可以被外部改写:`module._status = false;`<br>
<b>立即执行函数：</b><br>
```js
var module = function({
    _status = true,
    fun1:function(){...};
    fun2:function(){...};
    return {
        fun1 : fun1,
        fun2 : fun2
    };
})();
```
这样，外部代码就无法读取内部的私有变量了<br>
<b>放大模式：</b><br>
如果一个模块内容很多，需要分割为多个部分，或者一个模块需要继承另外一个模块，这时就需要用放大模式：
```js
var module = (function(mod){
    mod.fun3 = function(){...};
    return mod;
})(module);
```
这样，就给module模块添加一个新方法fun3()，然后返回新的module模块<br>
<b>宽放大模式：</b><br>
在浏览器中，模块每个部分通常在网上获取，有时无法分辨哪部分先加载，前面执行的部分可能加载到一个不存在的空对象，这时需要宽放大模式<br>
```js
var module = (function(mod){
    return mod;
})(window.module||{});
```
<b>输入全局变量：</b><br>
模块的特点是独立性，所以最好不要和其他部分直接交互，为了让模块内也能调用全局变量，必须显式将其他变量输入模块<br>
```js
var module = (function($,arg){
    ...
})(jQuery,arg);
```
上面，module需要用到jQuery和另外一个库，那么就将这两个模块作为参数传入module<br>
上述内容参考:<br>
[Javascript模块化编程（一）：模块的写法](http://www.ruanyifeng.com/blog/2012/10/javascript_module.html)<br>


参考：<br>
[JavaScrip 模块系统详解](https://zhanglun.github.io/2017/01/11/JavaScrip-%E6%A8%A1%E5%9D%97%E7%B3%BB%E7%BB%9F%E8%AF%A6%E8%A7%A3/)<br>

在JavaScript中通常将所有模块相关的代码通过闭包包装在一个匿名函数中。<br>
每一个ES6模块都是一个包含js的文件，模块本质上是一段脚本，加载之后只会执行一次，而不是用module关键字定义一个模块。在模块中可以声明变量、函数、类等(默认这些生命都是这个模块的局部声明；用户可以将一些声明到处以让其他模块使用)<br>
本质上，模块和脚本还是有一定区别的：<br>
