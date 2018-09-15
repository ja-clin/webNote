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