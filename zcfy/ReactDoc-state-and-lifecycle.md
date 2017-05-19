---
id: state-and-lifecycle
title: State and Lifecycle
permalink: docs/state-and-lifecycle.html
redirect_from: "docs/interactivity-and-dynamic-uis.html"
prev: components-and-props.html
next: handling-events.html
---

# State and Lifecycle (React 组件 生命周期)

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

The `componentDidMount()` hook runs after the component output has been rendered to the DOM. This is a good place to set up a timer:

```js{2-5}
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

Note how we save the timer ID right on `this`.

While `this.props` is set up by React itself and `this.state` has a special meaning, you are free to add additional fields to the class manually if you need to store something that is not used for the visual output.

If you don't use something in `render()`, it shouldn't be in the state.

We will tear down the timer in the `componentWillUnmount()` lifecycle hook:

```js{2}
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

Finally, we will implement the `tick()` method that runs every second.

It will use `this.setState()` to schedule updates to the component local state:

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

[Try it on CodePen.](http://codepen.io/gaearon/pen/amqdNA?editors=0010)

Now the clock ticks every second.

Let's quickly recap what's going on and the order in which the methods are called:

1) When `<Clock />` is passed to `ReactDOM.render()`, React calls the constructor of the `Clock` component. Since `Clock` needs to display the current time, it initializes `this.state` with an object including the current time. We will later update this state.

2) React then calls the `Clock` component's `render()` method. This is how React learns what should be displayed on the screen. React then updates the DOM to match the `Clock`'s render output.

3) When the `Clock` output is inserted in the DOM, React calls the `componentDidMount()` lifecycle hook. Inside it, the `Clock` component asks the browser to set up a timer to call `tick()` once a second.

4) Every second the browser calls the `tick()` method. Inside it, the `Clock` component schedules a UI update by calling `setState()` with an object containing the current time. Thanks to the `setState()` call, React knows the state has changed, and calls `render()` method again to learn what should be on the screen. This time, `this.state.date` in the `render()` method will be different, and so the render output will include the updated time. React updates the DOM accordingly.

5) If the `Clock` component is ever removed from the DOM, React calls the `componentWillUnmount()` lifecycle hook so the timer is stopped.

## Using State Correctly

There are three things you should know about `setState()`.

### Do Not Modify State Directly

For example, this will not re-render a component:

```js
// Wrong
this.state.comment = 'Hello';
```

Instead, use `setState()`:

```js
// Correct
this.setState({comment: 'Hello'});
```

The only place where you can assign `this.state` is the constructor.

### State Updates May Be Asynchronous

React may batch multiple `setState()` calls into a single update for performance.

Because `this.props` and `this.state` may be updated asynchronously, you should not rely on their values for calculating the next state.

For example, this code may fail to update the counter:

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

To fix it, use a second form of `setState()` that accepts a function rather than an object. That function will receive the previous state as the first argument, and the props at the time the update is applied as the second argument:

```js
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

We used an [arrow function](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) above, but it also works with regular functions:

```js
// Correct
this.setState(function(prevState, props) {
  return {
    counter: prevState.counter + props.increment
  };
});
```

### State Updates are Merged

When you call `setState()`, React merges the object you provide into the current state.

For example, your state may contain several independent variables:

```js{4,5}
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```

Then you can update them independently with separate `setState()` calls:

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

The merging is shallow, so `this.setState({comments})` leaves `this.state.posts` intact, but completely replaces `this.state.comments`.

## The Data Flows Down

Neither parent nor child components can know if a certain component is stateful or stateless, and they shouldn't care whether it is defined as a function or a class.

This is why state is often called local or encapsulated. It is not accessible to any component other than the one that owns and sets it.

A component may choose to pass its state down as props to its child components:

```
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```

This also works for user-defined components:

```
<FormattedDate date={this.state.date} />
```

The `FormattedDate` component would receive the `date` in its props and wouldn't know whether it came from the `Clock`'s state, from the `Clock`'s props, or was typed by hand:

```jsx
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

[Try it on CodePen.](http://codepen.io/gaearon/pen/zKRqNB?editors=0010)

This is commonly called a "top-down" or "unidirectional" data flow. Any state is always owned by some specific component, and any data or UI derived from that state can only affect components "below" them in the tree.

If you imagine a component tree as a waterfall of props, each component's state is like an additional water source that joins it at an arbitrary point but also flows down.

To show that all components are truly isolated, we can create an `App` component that renders three `<Clock>`'s:

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

[Try it on CodePen.](http://codepen.io/gaearon/pen/vXdGmd?editors=0010)

Each `Clock` sets up its own timer and updates independently.

In React apps, whether a component is stateful or stateless is considered an implementation detail of the component that may change over time. You can use stateless components inside stateful components, and vice versa.
