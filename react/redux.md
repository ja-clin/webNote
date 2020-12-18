### atcion

在 Flux 中，传统的想法是将每个 action type 定义为 string 常量：

```javascript
const ADD_TODO = 'ADD_TODO';
const REMOVE_TODO = 'REMOVE_TODO';
const LOAD_ARTICLE = 'LOAD_ARTICLE';
```

这么做的优势是什么？**人们通常声称常量不是必要的。对于小项目也许正确。** 对于大的项目，将 action types 定义为常量有如下好处：

- 帮助维护命名一致性，因为所有的 action type 汇总在同一位置。
- 有时，在开发一个新功能之前你想看到所有现存的 actions 。而你的团队里可能已经有人添加了你所需要的action，而你并不知道。
- Action types 列表在 Pull Request 中能查到所有添加，删除，修改的记录。这能帮助团队中的所有人及时追踪新功能的范围与实现。
- 如果你在 import 一个 Action 常量的时候拼写错了，你会得到 `undefined` 。在 dispatch 这个 action 的时候，Redux 会立即抛出这个错误，你也会马上发现错误。

### reducers

**Reducers** 指定了应用状态的变化如何响应 actions 并发送到 store 的，记住 actions 只是描述了*有事情发生了*这一事实，并没有描述应用如何更新 state。

保持 reducer 纯净非常重要。**永远不要**在 reducer 里做这些操作：

- 修改传入参数；
- 执行有副作用的操作，如 API 请求和路由跳转；
- 调用非纯函数，如 `Date.now()` 或 `Math.random()`。

**只要传入参数相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。**

用Reducers改变 state 中的 `visibilityFilter`

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

注意:

1. **不要修改 state。** 使用 [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 新建了一个副本。不能这样使用 `Object.assign(state, { visibilityFilter: action.filter })`，因为它会改变第一个参数的值。你**必须**把第一个参数设置为空对象。你也可以开启对ES7提案[对象展开运算符](http://www.redux.org.cn/docs/recipes/UsingObjectSpreadOperator.html)的支持, 从而使用 `{ ...state, ...newState }` 达到相同的目的。
2. **在 default 情况下返回旧的 state。**遇到未知的 action 时，一定要返回旧的 `state`。

