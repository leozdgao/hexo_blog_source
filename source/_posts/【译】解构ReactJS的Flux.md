title: "【译】解构ReactJS的Flux"
date: 2015-06-08 20:29:34
tags: react.js
categories: JS类库探索
---

## 用ReactJS时不要使用MVC

我将通过列出一些单向数据流的例子来将ReactJS官方实现的[Flux](https://github.com/facebook/flux)和我写的库[Reflux](https://github.com/spoike/refluxjs)作比较。

Facebook的ReactJS开发小组似乎并不待见MVC框架。将MVC模式和ReactJS结合使用了一段时间后，我似乎发现了争议从何而来了。你会遇到一个问题：你应该如何处理数据？ReactJS并不在乎太多关于数据是如何传入的或者贯穿整个Web应用去处理数据。这个几乎是一个架构层面的问题，并不是ReactJS所能涵盖的。于是Facebook中的优秀开发者提出了一个函数式的方法，他们称其为：**Flux**。

**Flux**的基本思想是可以在Web应用中拥有一个更加函数式的方法来处理数据。**Flux**介绍了Actions和Data Stores的概念来处理整个应用的事件和数据。数据流大致是这个样子的：

```
Action -> Data Store -> Component
```

数据的突变必须是在调用Actions时发生的，Data Stores需要监听actions并且改变store中的数据。这让数据结构保持扁平，并让数据的改变操作始终发生在Stores中，这防止了让Components自己处理数据所带来的副作用。

通过使用单向数据流，跟踪数据的改变将更加容易，因为它完全依赖于actions是如何发布的，继而影响整个应用。Components自身仅通过执行调用action来改变应用数据，这样避免了维护上的麻烦。

<!-- more -->

## Todo Example vs Reflux

这里有一个供参考的例子，是官方的[Todo-mvc](https://github.com/facebook/flux/tree/master/examples/flux-todomvc/)。我将基于这个例子来做我的解构。

Facebook喜欢将**Flux**说成是通过函数式编程来创建应用的一种方式，不过我发现了一些过去命令式编程的趋势并且可以它可以被改进得更加简单并且更加具有函数式的实现。

#### Dispatcher的古怪

TODO的实现涉及到了一个Dispatcher，它打包了所有的actions。接下来，Data Stores需要这样：

- 注册自己去监听action的事件，**所有的actions**
- 为了区分actions之间的区别，Stores需要比较它们要监听的action的名字（静态字符串）

后面那一点让我感觉困惑，因为它有一点破坏了JavaScript可以做到的函数式编程之美。我很抵触去比较类型，不论是通过字符串还是用`instanceof`，因为它摒弃了多态性，仿佛是在维护一堆蠕虫。

在**Reflux**中，我决定将Dispatcher合并进Actions中，去掉了其单例的实现。所以当你在使用actions的时候，你的应用仅需要做两做事：

- 创建actions
- 通过回调函数来监听action的调用

Actions是通过`Reflux.createAction`来创建的，传递一个回调函数给action的`listen`方法，于是它就可以被任何Data Store监听。

```
// Creating action
var toggleGem = Reflux.createAction();
 
// Listening to action
var isGemActivated = false;
 
toggleGem.listen(function() { 
  isGemActivated = !isGemActivated;
  var strActivated = isGemActivated ?
    'activated' :
    'deactivated';
  console.log('Gem is ' + strActivated);
});
```

在**Reflux**中的Actions是带有event emitters的函数。如果你想了解它的一个简单的实现，可以参考[这里](https://gist.github.com/spoike/ba561727a3f133b942dc)。

在你的应用中修改数据只需要调用action：

```
toggleGem();
 
// The callback that listens to the action will output
// "Gem is activated"
 
toggleGem();
 
// Will output "Gem is deactivated"
```

让actions成为一个函数，而不是维护一堆静态字符串：

```
var Actions = {};
 
Actions.toggleGem = createAction(); 
Actions.polishGem = createAction(); 
// and so on...
```

Data Stores也可以这样类似的实现，Reflux提供了一个方便的`createStore`函数。那些涉足ReactJS组件开发的开发者会把它认为是和`React.createClass`一样可以工作：

```
// Creates a DataStore
var gemStore = Reflux.createStore({
 
    // Initial setup
    init: function() {
        this.isGemActivated = false;
 
        // Register statusUpdate action
        this.listenTo(toggleGem, this.handleToggleGem);
    },
 
    // Callback
    handleToggleGem: function() {
        this.isGemActivated = !this.isGemActivated;
 
        // Pass on to listeners through
        // the DataStore.trigger function
        this.trigger(this.isGemActivated);
    }
 
});
```

在一个store实例上有两个方便的方法：`listenTo`用于action的注册，`trigger`用于触发DataStore的change事件。

ReactJS组件可以通过监听这些Data Stores的方式来使用它们：

```
var Gem = React.createClass({ 
    componentDidMount: function() {
        // the listen function returns a
        // unsubscription convenience functor
        this.unsubscribe =
            gemStore.listen(this.onGemChange);
    },
 
    componentWillUnmount: function() {
        this.unsubscribe();
    },
 
    // The listening callback
    onGemChange: function(gemStatus) {
        this.setState({gemStatus: gemStatus});
    },
 
    render: function() {
        var gemStatusStr = this.state.gemStatus ?
            "activated" :
            "deactivated";   
        return (React.DOM.h1(null,
            'Gem is ' + gemStatusStr));
    }
});
```

#### waitFor到底在等什么？

另一个让我困惑的是TodoList例子的代码中包含了一个`waitFor`，函数式的特性被公然破坏了。情况是这样的：一个Data Store需要等待其他Data Store在特定的action被执行后完成它们的数据处理，这似乎违背了单向数据流的原则。

不如让Data Store同样是可以被监听的。在**Reflux**中，你可以让一个Data Store直接监听另一个Data Store的change事件来处理上述情况。

## 结论

在你的ReactJS项目中试试用**Reflux**，它使得设置Flux架构更加简单。bower（`bower install reflux`）或者npm（`npm install reflux`）来都可以来获取到这个库。





