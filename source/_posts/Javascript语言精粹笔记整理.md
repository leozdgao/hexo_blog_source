title: "Javascript语言精粹笔记整理"
date: 2015-03-21 19:36:13
tags: javascript
categories: Javascript基础
---

这两天读了一下《Javascript语言精粹》一书，没有想象中的惊艳，不过在薄薄的100多页中浓缩了许多Javascript的编程技巧，值得细细品味。于是做了该笔记，算是对Javascript知识体系的一个回顾和整理，但并没有把书中的代码直接copy&paste，而是对技巧的罗列，其中对继承那部分，有一些自己的理解。

<!-- more -->

## 一些零星的技巧和知识点

Javascript的简单数据类型（原始类型 Primitive）包括Number、String、Boolean、null、undefined，其他所有值都是对象。

在Javascript内部，string以UTF-16存储，每个字符固定2个字节。

当一个Number以0开头时，会试着将其解释为八进制数，如果这个Number包含小数点，则会报出语法错误。

js中的假值（falsy）：false,0,null,undefined,'',NaN

```
012 // 10
089 // 89
012.345 // SyntaxError: Unexpected number
```

`ParseInt`方法会试着解析一个整形数字

```
parseInt(012) // 10
parseInt(012, 10) // 10
parseInt(012, 8) // 8
parseInt("123Test") // 123
parseInt({}) // NaN
```

将数字以0开头并试着将它认为是8进制数不是个好的做法，ES6中有八进制数字面量表示法：`0o12`

下面是NaN表现出的一些奇怪行为，要检测NaN应该使用方法`isNaN()`。

```
typeof NaN === 'number' // true
NaN !== NaN // true
NaN === NaN // false

isNaN(NaN) // true
isNaN("Hello") // true
```

`for...in`会遍历到原型链中的元素，如果要避免遍历到原型链中的元素，可以在循环中使用`object.hasOwnProperty(variable)`来判断其是否属于原型链。

`typeof`运算符结果的可能值有'number'、'string'、'boolean'、'undefined'、'function'和'object'。在用typeof判断数组或`null`时，将返回'object'。
```
// 判断是否为数组
function isArray(arr) {
	return Object.prototype.toString.apply(arr) === '[object Array]';
}

function isNull(obj) {
	return obj === null; // 注意要三个等号，因为undefined == null返回true
}
```

`||`与`&&`的妙用：

- 对于`||`，如果第一个表达式为false，则返回第二个表达式的值，否则返回第一个表达式的值。
- 对于`&&`，如果第一个表达式为true，则返回第二个表达式的值，否则返回第一个表达式的值。

```
var a = a || "default"; // 广泛被用于设置默认值
var b = obj && obj.pro; // 保证访问属性时因对象为falsy指而报错
```

## 函数相关

Javascript没有块级作用域，只有函数级作用域。

**闭包**：通过函数字面量创建的函数对象包含一个连到外部上下文的连接，即该函数可以访问它被创建时所处的上下文环境。

**模块**：一个提供接口却隐藏状态与实现的函数或对象。

函数在Javascript中有4种调用模式：

**方法调用模式**
```
myobject.method(args); // 此时函数绑定上下文为myobject对象
```
**函数调用模式**
```
var sum = add(3, 4); // 此时函数绑定上下文为global对象(window)
```
**构造器调用模式**
```
var animal = new Animal(); // 如果构造器函数返回值不是一个对象，则返回this
```
其中`new`运算符将创建一个继承于Animal原型的对象，并在调用Animal时将该对象绑定给this。

**apply调用模式**
```
Object.prototype.toString.apply(obj, args);
```

一个普通的函数声明，其上下文绑定为global对象（window），在对象字面量中声明的函数，其上下文绑定为该对象。

## 原型和继承相关

原型连接尽在检索值时才会被用到，当该对象没有相应的属性值时，会试着从原型中获取。对象属性的改变不会涉及到原型对象。

`delete`运算符不会触及原型链中的任何对象。

**伪类继承**：给Function.prototype赋一个对象，可以把这个对象看作是'基类'
```
function Animal() {
    this.name = "animal";
}
function Cat() {}
Cat.prototype = new Animal();
Cat.prototype.sayHi = function() {
    console.log("Miao");
};

var cat = new Cat();
cat.sayHi();
```

这种继承方式还是挺常见的，不过给prototype赋的对象是有讲究的，如果是new一个对象，则会执行构造器，也就是说上例中我们创建的Cat对象可以从原型链上检索到name这个属性。这里我们来对比一下node.js中的util模块里的inherits方法的实现吧：
```
exports.inherits = function(ctor, superCtor) {
  ctor.super_ = superCtor;
  ctor.prototype = Object.create(superCtor.prototype, {
    constructor: {
      value: ctor,
      enumerable: false,
      writable: true,
      configurable: true
    }
  });
};
```

在node.js的实现中，ctor.prototype的值是以superCtor的原型为原型创建的一个新对象，和上面一种的区别就在于没有执行构造器方法，也就是说如果使用这种方式的继承，Cat对象是无法从原型链上检索到name属性，只能检索到从原型链上继承来的方法。

**函数化模式**：该模式并没有像伪类继承那样模仿类继承的概念，而是直接舍弃了new操作符，通过普通函数调用的方式实现继承。

```
function Animal() {
    var that = {};
    that.name = "animal";
    that.sayHi = function() { console.log(Hi); }

    return that;
}

function Cat(name) {
    var that = Animal();
    that.name = name || "Cat";    

    return that;
}

var cat = Cat("kitty");
```

这种方法不需要使用new操作符，也不涉及到原型，还可以通过各种闭包来保存私有的状态。

我个人还是会更倾向于写伪类继承的模式，可能是后端出身的关系，觉得其更符合我的思维方式。不过函数化模式更多是用于保存私用的状态，主要不太会用它实现继承。

看到过如下的一种做法：

```
var SkyScroll = function(opts) { return new SkyScroll.prototype.init(opts); }
SkyScroll.prototype = {
  init: function(opts) {},
  // other methods
};
SkyScroll.prototype.init.prototype = SkyScroll.prototype;
window.SkyScroll = SkyScroll;
```

这种方法说不上和继承有关系，不过有一定特点，就是不论你使用`var a = SkyScroll({})`还是用new操作符调用构造器`var b = new SkyScroll({})`返回的结果都是相同的，不会因为使用函数调用模式调用构造器而污染全局环境，使用构造器模式调用返回的为一个对象，这个对象可以被正常返回，所以这两种方式都可以成功构造一个SkyScroll对象。
