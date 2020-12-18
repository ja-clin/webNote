jsx-> js对象-> dom    prevState    defaultProps   



React可以作为MVVM中第二个V，也就是View，但是并不是MVVM框架。

React的组件有自己的state，可以代替MVVM中M（Model）的功能，但是和MVVM完全两码事。

没有双向绑定就不是 MVVM



无状态

定义：当一个组件只有一个`render()`函数的时候（即UI组件），可将当前class组件改造为函数组件，即无状态组件。性能会比class组件高，因为它就是一个函数。函数的执行比class执行的更快。

this.props.children[0]



组件的私有方法都用 `_` 开头，所有事件监听的方法都用 `handle` 开头。把事件监听方法传给组件的时候，属性名用 `on` 开头。例如：

```javascript
<CommentInput
  onSubmit={this.handleSubmitComment.bind(this)} />
```

这样统一规范处理事件命名会给我们带来语义化组件的好处，监听（`on`）`CommentInput` 的 `Submit` 事件，并且交给 `this` 去处理（`handle`）。这种规范在多人协作的时候也会非常方便。

另外，组件的内容编写顺序如下：

1. static 开头的类属性，如 `defaultProps`、`propTypes`。
2. 构造函数，`constructor`。
3. getter/setter（还不了解的同学可以暂时忽略）。
4. 组件生命周期。
5. `_` 开头的私有方法。
6. 事件监听方法，`handle*`。
7. `render*`开头的方法，有时候 `render()` 方法里面的内容会分开到不同函数里面进行，这些函数都以 `render*` 开头。
8. `render()` 方法。



生命周期

我们把 *React.js 将组件渲染，并且构造 DOM 元素然后塞入页面的过程称为组件的挂载*（这个定义请好好记住）。

实例化 -> 渲染 -> 构建真实dom-> dom插入到页面

```javascript
-> constructor()
-> componentWillMount()
-> render()
// 然后构造 DOM 元素插入页面
-> componentDidMount()
// ...
// 即将从页面中删除
-> componentWillUnmount()
// 从页面中删除
```



##### keys

​	Keys可以在DOM中的某些元素被增加或删除的时候帮助React识别哪些元素发生了变化 。一个元素的key最好是这个元素在列表中拥有的一个独一无二的字符串。通常，我们使用来自数据的id作为元素的key。

​	如果列表可以重新排序，我们不建议使用索引来进行排序，因为这会导致渲染变得很慢。万不得已，你可以传递他们在数组中的索引作为key。若元素没有重排，该方法效果不错，但重排会使得其变慢。 当索引用作key时，组件状态在重新排序时也会有问题。组件实例基于key进行更新和重用。如果key是索引，则item的顺序变化会改变key值。这将导致受控组件的状态可能会以意想不到的方式混淆和更新。 



