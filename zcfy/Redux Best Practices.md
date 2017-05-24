* * *

# Redux Best Practices (Redux最佳实践)

原文链接:[https://medium.com/@kylpo/redux-best-practices-eef55a20cc72](https://medium.com/@kylpo/redux-best-practices-eef55a20cc72)

[Redux](https://github.com/reactjs/redux)可以管理你的应用状态.他是一个很好用并且可预测的状态管理工具,但是相比[Mobx](https://github.com/mobxjs/mobx) ([Dan Abramov’s tweet](https://twitter.com/dan_abramov/status/733705049902329856))速度上可能会稍微慢点.

React通过取出store中的数据来自动渲染UI以实现响应式的UI编程(就像名字React那样).因此你不必告诉视图层'如何更新UI'.同样的,我认为Redux/Flux使你的数据层(model)变成自动响应式,通过处理actions,这样你不必告诉数据模型(model)如何更新自己数据模型就会自己更新.-[from @dtinth](https://github.com/reactjs/redux/issues/1171#issuecomment-167714850)

### 规则

你的 **reducers**必须是纯函数(**确定的**).

所有具有副作用(不确定)的逻辑(外部服务,异步代码)都应该写在action中(可以通过[redux-thunk](https://github.com/gaearon/redux-thunk) and/or [redux-saga](https://github.com/yelouafi/redux-saga)这样一些辅助插件实现)

*   想了解更多关于纯函数和不纯函数的内容可以看[这里Github Issue 回复](https://github.com/reactjs/redux/issues/1171#issuecomment-205888533)

容器可以通过selectors来读取store中的数据.selectors和reducers可以协同作用来处理api数据.

*  可以看这篇博客[So you’ve screwed up your Redux store — or, why Redux makes refactoring easy](https://blog.boldlisting.com/so-youve-screwed-up-your-redux-store-or-why-redux-makes-refactoring-easy-400e19606c71#.rho2ned2d)

*   [Computing Derived Data | Redux](http://redux.js.org/docs/recipes/ComputingDerivedData.html)

*   [视频] [Colocating Selectors with Reducers](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers)

**学着在所有地方使用selectors**. 即使是在一个细小的位置.

Redux应该储存尽可能少的state,而让Selectors去计算派生的数据.

如果需要缓存之前selectors的计算结果(某些派生数据),你可以使用[Reselect](https://github.com/reactjs/reselect).

Selectors可以和其他selectors组合

规范化的数据结构可以构造出更好的reducer组合

* 可以看这篇关于[规范化](https://github.com/paularmstrong/normalizr)示例的文章

### Reducer 组合

用Redux以多层次的方式组织你的数据模型和**集合**(一种标准化模型).相比于将其作为一个整体,这将会是一个更好的方式.

就像Selectors和Components一样,Reducers也是可组合的!

将你的Redux看成是reducers组成的树(就像React渲染DOM时的树形结构那样).他们可以被当做不同函数的组合.

就算应用中有许多许多的reducers也是完全没问题的.

下面是Dan的视频,介绍了相关概念:

*   [Redux: Reducer Composition with Arrays](https://egghead.io/lessons/javascript-redux-reducer-composition-with-arrays)

*   [Redux: Reducer Composition with Objects](https://egghead.io/lessons/javascript-redux-reducer-composition-with-objects?series=getting-started-with-redux)

*   [Redux: Reducer Composition with combineReducers()](https://egghead.io/lessons/javascript-redux-reducer-composition-with-combinereducers?series=getting-started-with-redux)

* combineReducers: 他可以用来将不同reducers合并成一个reducer(当然被合并的reducer也可以是由其他reducers合并而来的)

### 结构和命名

使用 [Ducks](https://github.com/erikras/ducks-modular-redux)

*   提示: Dan 并不鼓励使用ducks因为它可能会妨碍你将actions和reducers一一对应. [tweet](https://twitter.com/dan_abramov/status/738405796770353152)

使用[redux-actions](https://github.com/acdlite/redux-actions)

action 命名规则: \<名词\>**_**\<动词\>

action creator 命名规则: \<动词>\<名词>

selector 命名规则: **_get_**\<名词>

### Actions

* action(常量)命名为\<动词\>_\<名词>,动词用现在时

*   **为什么?** 用于避免命名冲突同时利于排序

```
TODO_ADD

```

* 用[redux-actions](https://github.com/acdlite/redux-actions)的 **_createAction()_** 方法来生成你的action creators

* **为什么?** 可以减少重复代码以符合[FSA](https://github.com/acdlite/flux-standard-action)标准

```
createAction( ‘TODO_ADD’ )

```

* action creator 命名为\<动词\>\<名词>

*   **为什么?** 可以明确区分不同方法

```
const addTodo = createAction( ‘TODO_ADD’ )

```

### Selectors

* selector 命名为 _get_ \<名词>

*   **为什么?** 可以区分不同方法类型

```
const getTodo = (state) => state

```

### Reducers

* 使用[redux-actions](https://github.com/acdlite/redux-actions)的 **_handleActions()_** 方法来生成reducers

```
export default handleActions({

  TODO_ADD: (state, action) => ([
    ...state,
    action.payload,
  ]),

  // Other reducers
  // ...
  //

}, initialState )

```

* **为什么**没有用文档中的switch语句?主要是因为他可以实现简洁的类似于switch语法的功能,同时每一分支都有自己作用域.这意味着你可以在各个分支用重复变量名.而在switch中,你的作用域在switch中,所以你必须避免命名冲突.

### 文件

* 组织Redux目录可以按照[Ducks](https://github.com/erikras/ducks-modular-redux)的范例来将其放在一个单独目录下(通常叫做 **_flux/_**, **_ducks/_**, or **_stores/_**).可以从这个[链接](https://medium.com/@scbarrus/the-ducks-file-structure-for-redux-d63c41b7035c#.iw6yey65h)需求帮助

*   reducer示例:

```
import { createAction, handleActions } from 'redux-actions';

```

```
//- Actions
export const addTodo = createAction( 'TODO_ADD' )

```

```
//- State
const initialState = []

```

```
//- Reducers
export default handleActions({

```

```
  TODO_ADD: (state, action) => ([
    ...state,
    action.payload,
  ]),

```

```
  // Other reducers
  // ...
  //

```

```
}, initialState )

```

```
//- Selectors
export const getTodos = (state) => state

```

### 文件格式

* 如果你的代码格式是Flow或者TypeScript,那么redux的格式也应该保持一致!

*   Flow: [在Flow格式下使用Redux(Using Redux with Flow)](http://frantic.im/using-redux-with-flow) 这是一个好的示例.

### 相关资料

*   [对action-create,reducers和selectors的最佳实践建议(Recommendations for best practices regarding action-creators, reducers, and selectors)](https://github.com/reactjs/redux/issues/1171) —  对于怎么统一较复杂的与较简单的actions这里提出了比较好的思路

*   [使用React和Redux对Neos CMS进行完整的UI重构(Neos CMS Goes for a Full UI Rewrite with React and Redux)](http://dimaip.github.io/2016/03/13/neos-react-redux-rewrite/)

*   [关于Redux中的reducers我是不是理解错了?(Redux reducers.. Am I missing something? : javascript)](https://www.reddit.com/r/javascript/comments/40n5u3/redux_reducers_am_i_missing_something/) — 这里有对如何组织reducer的讨论
