title: "ECMAScript 6新增功能——块级作用域与解构赋值"
date: 2015-03-23 22:15:44
tags: [javascript, ES6]
categories: Javascript基础
---

根据[ECMAScript 6入门](http://es6.ruanyifeng.com/)学习了ES6的一些新特性，在io.js v1.61版本进行代码测试，部分代码需要打开了`--harmony`，`--use_strict`等flag。

这个系列将将会有如下内容
- let , const以及块级作用域
- 变量的解构赋值
- 新增的方法
- Set和Map数据结构
- Generator函数
- Promise对象
- Class和Module

<!-- more -->

## let , const以及块级作用域

*注：以下代码需要io.js --harmony --use_strict*

使用let声明的变量仅在所处的代码块中有效，即该变量处于块级作用域。

```
{
    let a = 6;
    var b = 6;
}

console.log(a);  // undefined
console.log(b);  // 6
```

用let声明的变量不存在变量提升，也不允许在代码块中出现相同的声明，即使在let声明之前。

```
function() {
    console.log(a);  // ReferenceError 'a' is not defined.
    let a = 6;
}

if(true) {
    var a = 6;  // Identifier 'a' is already defined.
    let a = 3;
    console.log(a);
}
```

使用let声明的变量将绑定其块级作用域，（这边不是特别清晰）可以避免在循环中声明函数时出现的隐患：
```
for(let i = 0; i < 10; i++) {
    setTimeout(function() {
        console.log(i);  // 输出为0-9
    });
}
```
如果上面的例子中改为var声明，则会输出10个10。

const拥有和let同样的特点，并且它用于声明常量，尝试改变常量的值不会导致异常，只会默默失败。
```
const a = 6;
a = 3;
console.log(a);  // 6
```

用const声明的对象仅是让其对象的引用地址保持不变，无法保证其属性不发生改变，要让对象不被改变，可以在使用const声明的同时，调用方法`Object.freeze`：
```
const a = Object.freeze({ msg: "Hello world!" });
```

在ES6中，函数声明时的作用域在其所处的块级作用域内。
```
function f() { console.log('I am outside!'); }
(function () {
  if(false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }
 
  f();  // ES5中，由于函数声明提升，输出为'I am inside!'，但在ES6中，结果为'I am outside!'
}());
```

最后，在ES6中规定，在全局使用var或是function声明的变量都会作为全局对象的属性，但用let , const或是class声明的变量，将不会成为全局对象的属性。（这边补充下，在node.js/io.js环境下，一个模块中全局声明的变量是不会成为全局对象global的属性的，要成为其属性，需要显式地赋值）
```
var a = 6;
console.log(window.a);  // 6
let b = 3;
consoel.log(window.b);  // undefined
```

## 变量的解构赋值（Destructure）

*注：当前版本的io.js不支持destructure*

有两种解构赋值方式：数组解构和对象结构
```
var [foo, bar] = [1, 3];  // foo = 1, bar = 3
var [foo, [bar]] = [1, [3]];  // foo = 1, bar = 3
var [,,third] = ["foo","bar","baz"];  // third = "baz"
var [head, ...tail] = [1,2,3,4]; // head = 1, tail = [2, 3, 4] (array spread特性)
var [a,b] = [1]; // a = 1, b = undefined
```

简单地说就是等号两边的模式匹配，如果匹配不成功，则变量值为`undefined`。对于数组解构，要求的是结构和顺序的匹配，而对于对象解构，要求的是结构和属性名的匹配：
```
var {foo, bar} = { foo: 'aaa', bzz: 'zzz' } // foo = 'aaa', bzz = undefined
```

如果变量名和属性名不一样也需要解构赋值，可以这样写：
```
var { foo: baz } = { foo: 'aaa' };  // baz = 'aaa'
```

两种解构赋值的方式都可以使用默认值：
```
var [a, b=3] = [6];  // a = 6, b = 3
var {foo,bar="dog"} = {};  // foo = undefined, bar = "dog"
```

**用途：**（这部分照抄了原文，觉得很有道理，不过鉴于io.js暂时不支持该特性，并未实践过如下用途）

**交换变量的值**
```
[x, y] = [y, x]
```

**函数返回多个值**
```
// 返回一个数组
function example() {
    return [1, 2, 3];
}
var [a, b, c] = example();

// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
var { foo, bar } = example();
```

**函数参数的定义及设置默认值**（合并了原文中的3和4）
```
jQuery.ajax = function (url, {
    async = true,
    beforeSend = function () {},
    cache = true,
    complete = function () {},
    crossDomain = false,
    global = true
    }) {
        // ... do stuff
    };
```

**遍历Map结构**
```
  var map = new Map();
  map.set('first', 'hello');
  map.set('second', 'world');
  
  for (let [key, value] of map) {
     console.log(key + " is " + value);
  }
  
  // 获取键名
  for (let [key] of map) {
    // ...
  } 
  // 获取键值
  for (let [,value] of map) {
    // ...
  }
```


