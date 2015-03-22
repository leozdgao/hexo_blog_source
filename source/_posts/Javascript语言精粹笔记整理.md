title: "Javascript语言精粹笔记整理"
date: 2015-03-21 19:36:13
tags: javascript
categories: Javascript基础
---

这两天读了一下《Javascript语言精粹》一书，没有想象中的惊艳，不过在薄薄的100多页中浓缩了许多Javascript的编程技巧，值得细细品味。于是做了该笔记，算是对Javascript知识体系的一个回顾和整理，但并没有把书中的代码直接copy&paste，而是对技巧的罗列，其中对继承那部分，有一些自己的理解。

<!-- more -->

## 一些零星的技巧和知识点

Javascript的简单数据类型（原始类型 Primitive）包括Number、String、Boolean、null、undefined，其他所有值都是对象。

js中的假值（falsy）：false,0,null,undefined,'',NaN

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
function isArray(arr) {
	return Object.prototype.toString.apply(arr) === '[object Array]';
}

function isNull(obj) {
	return obj === null; // 注意要三个等号，因为undefined == null返回true
}
```

另外删除数组中的元素应使用`splice`方法，如果用`delete`，仅会将原来的值置为undefined。

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

- 一个普通的函数声明，其上下文绑定为global对象（window），在对象字面量中声明的函数，其上下文绑定为该对象。

- 让一个本来无返回值（undefined）的函数返回this，可以实现函数的链式调用，让代码更加优雅，更具表现力。

## 原型和继承相关

原型连接尽在检索值时才会被用到，当该对象没有相应的属性值时，会试着从原型中获取。对象属性的改变不会涉及到原型对象。

`delete`运算符不会触及原型链中的任何对象。