title: "利用React创建评论区（Comment Box）组件"
date: 2015-02-19 11:20:49
tags: react.js
categories: JS类库探索
---

本文是在阅读学习了官方的[React Tutorial](http://facebook.github.io/react/docs/tutorial.html)之后的整理，实例[链接](http://7sbm5t.com1.z0.glb.clouddn.com/react0_ex.html)。


##开始使用React

首先从官方获取React.js的最新版本（[v0.12.2](http://facebook.github.io/react/downloads.html)），或者下载官方的[Starter Kit](http://facebook.github.io/react/downloads/react-0.12.2.zip)，并在我们的html中引入它们：
```html
<head>
    <meta charset="UTF-8">
    <title>React Test Page</title>
    <script src="../build/react.js"></script>
    <script src="../build/JSXTransformer.js"></script>
</head>
```

##JSX语法

我们可以在React组件的代码中发现xml标签似乎直接写进了javascript里：

```
React.render(
    <CommentBox />,
    document.getElementById('content')
);
```

这种写法被称作JSX，是React的一个可选功能，将xml标签直接写在javascript中看上去比调用javascript方法要更加直观些。要正常使用这个功能，需要在你的页面中引入`JSXTransformer.js`文件，或者使用`npm`安装`react-tools`，将包含JSX语法的源文件编译成常规的javascript文件，比较推荐的是后者，因为使用后者让页面可以直接使用编译后的javascript文件而不需要在加载页面时进行JSX编译。

JSX中的类HTML标签并不是真正的HTML元素，也不是一段HTML字符串，而是实例化了的React组件，关于JSX语法的更多内容，可以看这篇文章。

<!-- more -->

##创建组件

React可以为我们创建模块化、可组合的组件，对于我们需要做的评论区，我们的组件结构如下：
```
- CommentBox
    - CommentList
        -Comment
    - CommentForm
```

通过`React.createClass()`可以一个React元素，我们可以像这样定义我们的CommentBox，并通过`React.render()`方法可以让我们在指定的容器中将React元素渲染为一个DOM组件：
```html
<body>
    <div id="content"></div>
    <script type="text/jsx">
        var CommentBox = React.createClass({
            render: function() {
                return (
                    <div className="contentBox">
                        <h1>Comments</h1>
                        <CommentList />
                        <CommentForm />
                    </div>
                );
            }
        });
 
        React.render(
            <CommentBox />,
            document.getElementById('content')
        );
    </script>
</body>
```

从这个例子也可以看出一个组件可以包含子组件，组件之间是可以组合的（Composing），并呈现一个树形结构，也可以说render方法中的的CommentBox代表的是组件树的根元素。那么接下来我们来创建CommentList和CommentForm这两个子组件。

首先是CommentList组件，这个组件是用来呈现评论列表的，根据开始我们设计的组件结构树，这个组件应该是包含许多Comment子组件的，那么，假设我们已经获取到评论数据了：

```
var comments = [
    {author: "Pete Hunt", text: "This is one comment"},
    {author: "Jordan Walke", text: "This is *another* comment"}
];
```

我们需要把数据传递给CommentList组件才能让它去呈现，那么如何传递呢？我们可以通过`this.props`来访问组件标签上的属性，比如我们在CommentBox组件的代码中做如下修改：

```
<CommentList data=comments  />
```

于是在CommentList组件中，我们可以通过访问`this.props.data`来获取到我们的评论数据。

```
var CommentList = React.createClass({
    render: function() {
        var commentNodes = this.props.data.map(function(comment) {
            return (
                <Comment author={comment.author}>
                    {comment.text}
                </Comment>
            );
        });
 
        return (
            <div className="commentList">
                {commentNodes}       
            </div>
        );
    }
});
```

接下来写Comment组件，这个组件用于呈现单个评论，我们希望它可以支持markdown语法，于是我们引入[showdown](https://github.com/showdownjs/showdown)这个库，在HTML中引入它之后，我们可以调用它让我们的评论支持Markdown语法。在这里我们需要`this.props.children`这个属性，它返回了该组件标签里的所有子元素。

```
var converter = new Showdown.converter();
var Comment = React.createClass({
    render: function() {
        return (
            <div className="comment">
                <h2 className="commentAuthor">
                    {this.props.author}
                </h2>
                {converter.makeHtml(this.props.children.toString())}
            </div>
        );
    }
});
```

我们看一下现在的效果：
![效果](http://7sbm5t.com1.z0.glb.clouddn.com/react0_pic0.png)

我们发现经过解析后html标签被直接呈现了上去，因为React默认是有XSS保护的，所有对呈现的内容进行了转义，但在现在的场景中，我们并不需要它的转义（如果取消React默认的XSS保护，那么就需要仰仗于我们引入的库具有XSS保护或者我们手动处理），这时我们可以这样：

```
var converter = new Showdown.converter();
var Comment = React.createClass({
    render: function() {
 
        // 通过this.props.children访问元素的子元素
        var rawHtml = converter.makeHtml(this.props.children.toString());
        return (
            // 通过this.props访问元素的属性
            // 不转义，直接插入纯HTML
            <div className="comment">
                <h2 className="commentAuthor">{this.props.author}</h2>
                <span dangerouslySetInnerHTML={{__html: rawHtml}} />
            </div>
        );
    }
});
```

好了，接下来我们的CommentList算是完成了，我们需要加上CommentForm组件让我们可以提交评论：

```
var CommentForm = React.createClass({
    handleSubmit: function(e) {
 
        e.preventDefault();

        var author = this.refs.author.getDOMNode().value.trim();
        var text = this.refs.text.getDOMNode().value.trim();
 
        if(!text || !author) return;
 
        // TODO 修改commentList
 
        // 获取原生DOM元素
        this.refs.author.getDOMNode().value = '';
        this.refs.text.getDOMNode().value = '';
    },
    render: function() {
        return (
            // 为元素添加submit事件处理程序
            // 用ref为子组件命名，并可以在this.refs中引用
            <form className="commentForm" onSubmit={this.handleSubmit}>
                <input type="text" placeholder="Your name" ref="author"/>
                <input type="text" placeholder="Say something..." ref="text"/>
                <input type="submit" value="Post"/>
            </form>
        );
    }
});
```

从以上的代码中我们可以发现，我们可以为我们的组件添加事件处理程序，比如在这里我们需要利用form的submit事件，于是直接在标签上添加`onSubmit`的属性即可。需要注意的是，事件属性需要满足驼峰命名规则，也就是说如果是要添加click事件，那就要添加`onClick`，以此类推。还有一点就是我们需要获取两个文本框中的内容，这里使用的方法是在`input`标签上添加`ref`属性，这样就可以认为这个`input`是它的一个子组件，然后就可以通过访问`this.refs`来访问到这个子组件了，通过调用`getDOMNode`方法可以获取原生的DOM对象进行相应的操作。

我们发现到现在为止，我们的页面是静态的，但我们希望可以在成功提交了评论后可以立刻在评论列表中看到自己的评论，并可以每隔一段时间获取最新的评论，也就是说我们希望我们的CommentBox可以动态地改变状态。

首先我们先让CommentBox组件可以通过AJAX请求（在这里我用setTimeout来模拟获取数据的延迟），从服务器端获取评论数据同时更新CommentList。React组件有一个私有的`this.state`属性用于保存组件可变状态的数据，但一开始我们需要的是一个初始的状态，初始状态可以通过设置组件的`getInitialState`方法，它的返回值即为状态初始值。这个时候我们不是从标签的属性上直接获取数据了，需要通过访问`this.state`来获取（这个`state`属性如果直接用javascript访问会返回`undefined`，但可以在JSX中可以像`this.state.data`这样使用）：

```
var CommentBox = React.createClass({
  getInitialState: function() {
    return {data: []};
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm />
      </div>
    );
  }
});
```

接下来我们需要获取评论数据，我们可以在组件的`componentDidMount`方法中实现，这个方法会在组件呈现在页面上之后会被立刻调用一次，我们就在这个方法中获取到数据后更新下组件的状态，要更新组件的状态需要调用组件的`this.setState`方法，于是我们就这样写：

```
var CommentBox = React.createClass({
    // 在组件的生命周期中仅执行一次，用于设置初始状态
    getInitialState: function() {
        return {data: []};
    },
    loadCommentsFromServer : function() {
 
        var self = this;
        setTimeout(function() {
            // 动态更新state
            self.setState({data: comments});
        }, 2000);
    },
    // 当组件render完成后自动被调用
    componentDidMount: function() {
 
        this.loadCommentsFromServer();
        setInterval(this.loadCommentsFromServer, this.props.pollInterval);
    },
    render: function() {
        return (
            <div className="commentBox">
                <h1>Comments</h1>
                <CommentList data={this.state.data} />
                <CommentForm />
            </div>
        );
    }
});
```

现在我们已经可以更新评论列表里的数据了，那么同样的我们在CommentForm中成功提交的评论也要可以在CommentList中呈现出来，在这里需要注意的是我们现在设置的初始状态是CommentBox这个组件的，修改状态也是修改的CommentBox的状态，那么如果要在CommentForm中改变CommentBox的状态，就需要在CommentBox组件中通过标签属性的方式传递一个方法给子组件CommentForm，让CommentForm组件中的`handleSubmit`可以调用这个方法（也就是上面TODO的位置），于是我们的代码就是这样的：

```
var CommentBox = React.createClass({
    // 在组件的生命周期中仅执行一次，用于设置初始状态
    getInitialState: function() {
        return {data: []};
    },
    onCommentSubmit: function(comment) {
        // 模拟提交数据
        comments.push(comment);
 
        var self = this;
        setTimeout(function() {
            // 动态更新state
            self.setState({data: comments});
        }, 500);
    },
    loadCommentsFromServer : function() {
 
        var self = this;
        setTimeout(function() {
            // 动态更新state
            self.setState({data: data});
 
        }, 2000);
    },
    // 当组件render完成后自动被调用
    componentDidMount: function() {
 
        this.loadCommentsFromServer();
        setInterval(this.loadCommentsFromServer, this.props.pollInterval);
    },
    render: function() {
        return (
            // 并非是真正的DOM元素，是React的div组件，默认具有XSS保护
            <div className="commentBox">
                <h1>Comments</h1>
                <CommentList data={this.state.data} />
                <CommentForm onCommentSubmit={this.onCommentSubmit} />
            </div>
        );
    }
});

var CommentForm = React.createClass({
    handleSubmit: function(e) {
 
        e.preventDefault();
        // e.returnValue = false;
        var author = this.refs.author.getDOMNode().value.trim();
        var text = this.refs.text.getDOMNode().value.trim();
 
        if(!text || !author) return;
 
        this.props.onCommentSubmit({author: author, text: text});
 
        // 获取原生DOM元素
        this.refs.author.getDOMNode().value = '';
        this.refs.text.getDOMNode().value = '';
    },
    render: function() {
        return (
            // 为元素添加submit事件处理程序
            // 用ref为子组件命名，并可以在this.refs中引用
            <form className="commentForm" onSubmit={this.handleSubmit}>
                <input type="text" placeholder="Your name" ref="author"/>
                <input type="text" placeholder="Say something..." ref="text"/>
                <input type="submit" value="Post"/>
            </form>
        );
    }
});
```

到此为止，我们的CommentBox组件就大功告成了，实例[链接](http://7sbm5t.com1.z0.glb.clouddn.com/react0_ex.html)。
