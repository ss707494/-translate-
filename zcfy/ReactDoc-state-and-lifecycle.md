---
id: state-and-lifecycle
title: State and Lifecycle
permalink: docs/state-and-lifecycle.html
redirect_from: "docs/interactivity-and-dynamic-uis.html"
prev: components-and-props.html
next: handling-events.html
---

# State and Lifecycle (React 组件 生命周期)
原文链接: [https://github.com/facebook/react/blob/master/docs/docs/state-and-lifecycle.md](https://github.com/facebook/react/blob/master/docs/docs/state-and-lifecycle.md)
ps: 这是React官方文档中的文章,本人作为学习材料简单翻译下.

本文使用的是上一篇的[时钟示例](https://facebook.github.io/react/docs/rendering-elements.html#updating-the-rendered-element)

目前为止,我们仅仅只学了一种更新UI的方法.

我们调用`ReactDOM.render()`来进行渲染:

```js{8-11}
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[在CodePen上尝试](http://codepen.io/gaearon/pen/gwoJZk?editors=0010)

在这一部分,我们将学习封装一个真正可复用的`Clock`(时钟)组件,它将拥有自己的计时器同时自发地更新时间.

我们先看看之前封装好的组件:

```js{3-6,12}
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[在CodePen上尝试](http://codepen.io/gaearon/pen/dpdoYR?editors=0010)

但是,这个组件没满足一个关键的需求:事实上时钟组件每秒钟更新UI的功能应该是组建内部进行实现,这样才合理.

由组件自己更新UI才是合理的方案:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

为了实现这个,我们需要给`Clock`组件添加"state"

state与props相似,但是它是私有属性,同时完全由组件内部控制.

我们[之前提到过](https://facebook.github.io/react/docs/components-and-props.html#functional-and-class-components)以classes方式生成的组件拥有一些额外功能.state属性就是只有在classes中才有的功能.

## 将方法(function)定义的组件转换为类(class)定义

你可以花下面5步将一个像`Clock`通过方法定义的组件转换为通过类(class)定义的组件

1. 创建一个继承(extends)了`React.Component`的类(class)

2. 添加一个叫`render()`的方法

3. 将之前组件方法体中的代码添加至`render()`中

4. 在`render()`中将`props`替换为`this.props`

5. 删除原先的方法声明

```es6
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

[在CodePen上尝试](http://codepen.io/gaearon/pen/zKRGpo?editors=0010)

`Clock`现在被定义为一个类(class)而不是一个函数(function).

这使我们可以使用其他功能,如本地状态和生命周期方法.

## 向类中添加本地状态(Local State)

我们将通过3步来将`date`从props变成state

1) 将`render()`方法中的`this.props.date`替换为`this.state.date`

```js{6}
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

2) 添加一个[构造函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes#Constructor)用来初始化`this.state`:

```js{4}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

注意我们是如何将`props`传递给父类构造函数的:

```js{2}
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

Class组件应该用`props`作参数调用父类构造函数.

3) 删除`<Clock />`组件里的props `date`属性:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

我们稍后将给组件添加计时代码.

结果就像下面这样:

```js{2-5,11,18}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[在CodePen上尝试](http://codepen.io/gaearon/pen/KgQpJd?editors=0010)

接下来,我们将给`Clock`组件安装计时器以让它每秒钟更新一次UI.

## 向类中添加生命周期函数

在拥有许多组件的应用中,销毁组件时释放掉组件所占用的资源是非常重要的.

我们想要在组件第一次渲染进DOM时[添加计时器方法](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval).在React中这个过程被叫做"mounting"

我们想要在DOM中的组件被移除时执行[移除计时器方法](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval).在React中这个过程被叫做"unmounting"

为了在组件加载(mounts)及销毁(unmounts)时执行指定方法,我们需要在组件类中声明特别的函数:

```js{7-9,11-13}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {

  }

  componentWillUnmount() {

  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

这些方法被称为"生命周期函数(lifecycle hooks)".

`componentDidMount()`函数将在组件渲染到DOM后触发.我们可以在这里执行启动计时器的方法:


```js{2-5}
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

注意我们如何将计时器的ID赋值给`this`的.

虽然`this.props`是React内置的属性而且`this.state`有特别的含义,你仍然可以自定义的在组件实体类上添加不用于展示的其他属性.

如果这些属性你不准备用在`render()`方法里,那就不要把它写在state里面.

我们将会在`componentWillUnmount()`这个函数中关闭计时器:

```js{2}
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

最后,我们将实现`tick()`方法,他将会在每秒钟执行一次.

它将调用`this.setState()`方法来根据日期时间来修改local state:

```js{18-22}
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[在CodePen上尝试](http://codepen.io/gaearon/pen/amqdNA?editors=0010)

现在时钟将每一秒更新一次.

让我们快速回顾一下发生了什么以及调用方法的顺序:

1. 当`<Clock />`组件被传递到`ReactDOM.render()`方法中,React框架调用了`Clock`组件的构造函数.`Clock`需要显示当前时间,所以他将用储存当前时间信息的对象来初始化`this.state`.

2. React调用`Clock`组件的`render()`方法.React通过这个方法将组件渲染到屏幕上.react将用render函数的输出更新DOM.

3. 当`Clock`被插入到DOM中时,React调用`componentDidMount()`生命周期函数.其中,组件将在浏览器中安装计时器用来每秒执行一次`tick()`方法.

4. 每秒钟浏览器执行`tick()`方法.`Clock`组件通过使用当前时时间作参数执行`setState()`方法来更新UI.因为执行了`setState()`方法,React知道state已经改变,然后再次执行`render()`方法重新渲染页面.这一次,`render()`方法中的`this.state.date`将会不同,所以渲染的是最新的时间.React相应地更新DOM界面.

5. 一旦`Clock`组件从DOM中被移除,React将调用`componentWillUnmount()`生命周期函数,这样计时器将停止.

## 正确使用state

关于`setState()`你应该知道3点.

### 不要直接修改state

这将不会更新组件:

```js
// Wrong
this.state.comment = 'Hello';
```

相应的,你应该使用`setState()`:

```js
// Correct
this.setState({comment: 'Hello'});
```

你可以直接赋值给`this.state`的唯一地方就是构造函数(constructor)中.

### 异步修改state 

React可能会在一次操作中多次调用`setState()`方法.

因为`this.props`和`this.state`可能会异步更新,你不应该依赖他们之前的值来计算下一个值.

例如,下面代码可能无法更新计数:

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

为了解决这个问题,我们使用第二种方式调用`setState()`,传一个方法而不是对象做参数.该方法将接受之前的state值作为第一个参数,应用的props作为第二个参数:

```js
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

在上面我们用的[箭头函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions),你也可以使用普通函数的写法:

```js
// Correct
this.setState(function(prevState, props) {
  return {
    counter: prevState.counter + props.increment
  };
});
```

### State 通过合并的方式更新

当你调用`setState()`方法时,React将新的State合并到之前的state中.

例如,你的state包含几个独立变量:

```js{4,5}
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```

你可以分别调用`setState()`方法来单独更新数值:

```js{4,10}
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

这里是浅合并,所以`this.setState({comments})`将会完整的保留`this.state.posts`,同时完全替换掉`this.state.comments`.

## 单向数据流

不论是父组件还是子组件都不知道某一组件是有状态还是无状态,同时他们也不关心他是一个方法还是一个类.

所以state通常被称作本地化的和被封装的.任何其他组件都不可以修改该组件的state值.

一个组建可以将state作为子组件的props传递给子组件:

```
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```

这也适用于用户自定义组件:

```
<FormattedDate date={this.state.date} />
```

`FormattedDate`组件将会接受`date`数值但是不知道它是来自`Clock`组件的state或者props,还是来自人工输入:

```jsx
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

[在CodePen上尝试](http://codepen.io/gaearon/pen/zKRqNB?editors=0010)

这通常被称作"自上而下"或"单向"数据流.任何一个state都是属于特定的组件,该state的数据和UI只能影响该组件树下面的组件.

可以这么想,一个组件树就像一个props的瀑布,每个组建的state就像是链接某一点的额外的水源,同时该水源也是从上往下流.

为了表明所有组件是隔离的,我们可以创建一个`App`组件,它包含三个`<Clock>`组件:

```js{4-6}
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[在CodePen上尝试](http://codepen.io/gaearon/pen/vXdGmd?editors=0010)

每一个`Clock`组件都有自己的计时器同时单独更新.

在React应用中,组件是有状态或者无状态被认为是一个会随时改变的实现细节.你可以在有状态的组件中使用无状态组件,反之亦可.
