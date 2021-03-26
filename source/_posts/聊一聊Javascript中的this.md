---
title: 聊一聊Javascript中的this
date: 2016-12-02 23:31:05
tags:
  - this指向
  - 原生JS
  - 技能知识图谱
categories: 原生JS
---
在大多的高级编程语言中，总能发现this的身影，然而在js中的this，跟其他语言中的this还是有区别的，在翻阅别人的js代码，总能看到类似于`self = this;`，或者`func.bind(this);`，再或者`this.xxx`类似的语句，好像认识它却又不认识他，直到我阅读了《javascript高级程序设计》和《你不知道的Javascript》系列丛书，再借助网络的一些文章，才渐渐的认识了JS中的this。

<!--more-->

## 默认this
我们先来看看，在Window和NodeJS两个宿主环境中的this默认是个什么东西？？

### 浏览器
在chrome控制台打印this,如下：
![this](/images/2016/12/02/this1.png)

可见，window是宿主环境为浏览器的JS的顶层对象，我们可以认为，在浏览器中，this默认指向的是window。

### Node.JS
在Node.js中打印this，如下：

![this](/images/2016/12/02/this1.png)

可见，this默认指向的是global。

我们可以得到结论默认this默认指向的是全局，浏览器中默认指向window顶层对象，nodeJS中指向global顶层对象。

以浏览器环境为例，举例证明：

```
//首先定义一个变量，直接var了一个全局变量
var a = 1;
//定义一个函数
function demo(){
  this.a = 2;
}
demo();
console.log(a);    //window.a也可
```

运行结果：

![alt](http://mife.io/static/upload/20171005/FuGg9hrmbVqw9iFeLn17ttPZ.png)

可以看到，函数内部改变了外部全局定义的`a`的默认值，说明在函数内部`this`指向的是`window`对象，所以全局的变量（或者也可称`window.a`）会被改变。

## 隐式绑定中的this
定义一个变量：
```
var name = 'Window DADA';
```
在定义一个对象：
```
var obj = {
  name:'mengyu mi',
}
```
在定义的一个函数：
```
function getName(){
  console.log(this.name);
};
```
```
obj.getNames = getName;
```
猜想下面两个结果：
```
getName(); //Window DADA
obj.getNames(); //mengyu mi
```
实际结果：
![alt](http://mife.io/static/upload/20171005/fy4NacKamLcNhcyLlP_D9y2X.png)

由此可见，在进行函数`getName()`调用的时候，`this`调用的是默认的`window`，当将函数作为一个值赋给obj对象的一个属性时，在调用obj的`getNames`方法，此时this被绑定在了`obj`对象上，对这种this的绑定方式称之为隐式绑定。

## 硬绑定中的this
在javascript中有三个内置方法，他们可以强制改变`this`的指向，他们分别是`apply`，`call`和`bind`,称之为硬绑定。
定义两个对象：
```
var obj1 = {
  name: 'obj1',
  getName: function(){
    console.log(this.name);
  }
};
var obj2 = {
  name:'obj2'
}
```
如上，定义了两个对象，每个对象中有一个name属性，obj1对象中还有一个getName()方法，直接调用`obj1.getName()`的话，按照上面提到的隐式绑定，输出的应该是obj1，这个结果无可厚非。

假设，我需要在obj2对象上调用obj1的getName()方法，笨方法是在obj2上定义一个跟obj1一样的getName方法，当然，这样做法也能达到目的，但是，显然不是最佳的办法，实际上，我只需要改变obj1中的getName()方法的this就可以了，这时候，`apply`、`call`和`bind`方法就可以派上用场了，这三个内置方法就是做个用的。

![alt](http://mife.io/static/upload/20171005/7hd1e79EFWomZ9j_8zJCHNEz.png)

根据显示结果，可以看到`this`被强制绑定到了call()、apply()、bind()传入的对象的上。

## 对函数构造调用中的this
看到这个标题，似乎有点绕，先简单说明一些标题。有人称这种this绑定为构造函数的this绑定，然而在ES5中并没有像其他高级编程语言（如java）中的构造函数，ES5中一般的函数声明都可以当做构造函数来使用，只是他们前面多了一个`new`关键字，似乎看起来更像传统意义上的构造函数罢了，似乎称之为“对一般函数的构造调用”似乎更加确切。

还是先上示例代码：
```
function foo(name){
  this.name = name;
};

foo.prototype.getName = function(){
  console.log(this.name);
};

var foo1 = new foo('foo1');
foo1.getName();

var foo2 = new foo('foo2');
foo2.getName();

```
运行结果：

![alt](http://mife.io/static/upload/20171005/pQlcoIMFM94W_JQYBmTXlAS5.png)

由此可以看到this被绑定到了构造出来的实例上。
