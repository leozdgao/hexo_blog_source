title: "React JSX语法说明"
date: 2015-02-21 11:17:00
tags: react.js
categories: JS类库探索
---

## 什么是JSX？

在用React写组件的时候，通常会用到JSX语法，粗看上去，像是在Javascript代码里直接写起了XML标签，实质上这只是一个**语法糖**，每一个XML标签都会被JSX转换工具转换成纯Javascript代码，当然你想直接使用纯Javascript代码写也是可以的，只是利用JSX，组件的结构和组件之间的关系看上去更加清晰。

```
var MyComponent = React.createClass({/*...*/});
var myElement = <MyComponent someProperty={true} />;
React.render(myElement, document.body);
```

<!-- more -->

一个XML标签，比如`<MyComponent someProperty={true} />`会被JSX转换工具转换成什么呢？

比如：
```
var Nav = React.createClass({/*...*/});
var app = <Nav color="blue"><Profile>click</Profile></Nav>;
```

会被转化为：
```
var Nav = React.createClass({/*...*/});
var app = React.createElement(
    Nav,
    {color:"blue"},
    React.createElement(Profile, null, "click")
);
```

那么也就是说，我们写一个XML标签，实质上就是在调用`React.createElement`这个方法，并返回一个`ReactElement`对象。

```
ReactElement createElement(
    string/ReactClass type,
    [object props],
    [children ...]
)
```

这个方法的第一个参数可以是一个字符串，表示是一个HTML标准内的元素，或者是一个`ReactClass`类型的对象，表示我们之前封装好的自定义组件。第二个参数是一个对象，或者说是字典也可以，它保存了这个元素的所有固有属性（即传入后基本不会改变的值）。从第三个参数开始，之后的参数都被认作是元素的子元素。

## JSX转化器

要把带有JSX语法的代码转化为纯Javascript代码，有多种方式，对于内联与HTML中的代码或者是未经过转化的外部文件，在`script`标签中要加上`type="text/jsx"`，并引入`JSXTransformer.js`文件即可，不过这种方式并不建议在生产环境使用，建议的方法是在代码上线前就将代码转换好，可以使用`npm`**全局安装**`react-tools`：

```
npm install -g react-tools
```

并使用命令行工具转化即可（具体用法可以参考`jsx -h`）：

```
jsx src/ build/
```

如果使用自动化工具，比如`gulp`的话，可以使用相应插件`gulp-react`。


## 使用HTML标签

要创建一个HTML标准中存在的元素，直接像写HTML代码一样即可：

```
var myDivElement = <div className="foo" />;
React.render(myDivElement, document.body);
```

不过需要注意的是`class`和`for`这两个属性，JSX语法最终是要被转换为纯Javascript的，所以要和在Javascript DOM中一样，用`className`和`htmlFor`。

还有一点是，在创建HTML标准内的元素时，JSX转化器会丢弃那些非标准的属性，如果一定要添加自定义属性，那么需要在这些自定义属性之前添加`data-`前缀。

```
<div data-custom-attribute="foo" />
```

## 命名空间式组件

比如开发组件的时候，一个组件有多个子组件，你希望这些子组件可以作为其父组件的属性，那么可以像这样用：

```
var Form = MyFormComponent;
 
var App = (
  <Form>
    <Form.Row>
      <Form.Label />
      <Form.Input />
    </Form.Row>
  </Form>
);
```

这样你只需将子组件的`ReactClass`作为其父组件的属性：

```
var MyFormComponent = React.createClass({ ... });
 
MyFormComponent.Row = React.createClass({ ... });
MyFormComponent.Label = React.createClass({ ... });
MyFormComponent.Input = React.createClass({ ... });
```

而创建子元素可以直接交给JSX转化器：

```
var App = (
    React.createElement(Form, null,
        React.createElement(Form.Row, null,
            React.createElement(Form.Label, null),
            React.createElement(Form.Input, null)
        )
    )
);
```

*该功能需要0.11及以上版本*

## Javascript表达式

在JSX语法中写Javascript表达式只需要用`{}`即可，比如下面这个使用三目运算符的例子：

```
// Input (JSX):
var content = <Container>{window.isLoggedIn ? <Nav /> : <Login />}</Container>;
// Output (JS):
var content = React.createElement(
    Container,
    null,
    window.isLoggedIn ? React.createElement(Nav) : React.createElement(Login)
);
```

不过要注意的是，JSX语法只是语法糖，它的背后是调用`ReactElement`的构造方法`React.createElement`的，所以类似这样的写法是不可以的：

```
// This JSX:
<div id={if (condition) { 'msg' }}>Hello World!</div>
 
// Is transformed to this JS:
React.createElement("div", {id: if (condition) { 'msg' }}, "Hello World!");
```

可以从转化后的Javascript代码中看出明显的语法错误，所以要不用三目运算符，要不就这样写：

```
if (condition) <div id='msg'>Hello World!</div>
else <div>Hello World!</div>
```

## 传播属性（Spread Attributes）

在JSX中，可以使用`...`运算符，表示将一个对象的键值对与`ReactElement`的`props`属性合并，这个`...`运算符的实现类似于ES6 Array中的`...`运算符的特性。

```
var props = { foo: x, bar: y };
var component = <Component { ...props } />;
```

这样就相当于：

```
var component = <Component foo={x} bar={y} />
```

它也可以和普通的XML属性混合使用，需要同名属性，后者将覆盖前者：

```
var props = { foo: 'default' };
var component = <Component {...props} foo={'override'} />;
console.log(component.props.foo); // 'override'
```

## 参考资料（可能无法直接打开链接）
- [JSX in Depth](http://facebook.github.io/react/docs/jsx-in-depth.html)
- [JSX Spread Attributes](http://facebook.github.io/react/docs/jsx-spread.html)
- [If-Else in JSX](http://facebook.github.io/react/tips/if-else-in-JSX.html)
- [JSX Gotchas](http://facebook.github.io/react/docs/jsx-gotchas.html)








