###react创建组件的三种方式与区别

1. 函数式定义的`无状态组件`
2. es5原生方式`React.createClass`定义的组件
3. es6形式的`extends React.Component`定义的组件

#####无状态组件

它是为了创建纯展示组件，这种组件只负责根据传入的props来展示，不涉及到要state状态的操作。

无状态函数式组件形式上表现为一个只带有一个**render**方法的组件类，通过函数形式或者ES6 箭头 function的形式在创建，并且该组件是无state状态的。

```javascript
function HelloComponent(props, /* context */) {
  return <div>Hello {props.name}</div>
}
ReactDOM.render(<HelloComponent name="Sebastian" />, mountNode) 


cosnt HelloComponent = (props) => {
  
  return (<div>xxx</div>)
}
```

无状态组件的创建形式使代码的可读性更好，并且减少了大量冗余的代码，精简至只有一个render方法，大大的增强了编写一个组件的便利，除此之外无状态组件还有以下几个显著的特点：

1. **组件不会被实例化**，整体渲染性能得到提升
   因为组件被精简成一个render方法的函数来实现的，由于是无状态组件，所以无状态组件就不会在有组件实例化的过程，无实例化过程也就不需要分配多余的内存，从而性能得到一定的提升。
2. **组件不能访问this对象**
   无状态组件由于没有实例化过程，所以无法访问组件this中的对象，例如：this.ref、this.state等均不能访问。若想访问就不能使用这种形式来创建组件
3. **组件无法访问生命周期的方法**
   因为无状态组件是不需要组件生命周期管理和状态管理，所以底层实现这种形式的组件时是不会实现组件的生命周期方法。所以无状态组件是不能参与组件的各个生命周期管理的。
4. 无状态组件只能访问输入的props，同样的props会得到同样的渲染结果，不会有副作用

无状态组件被鼓励在大型项目中尽可能以简单的写法来分割原本庞大的组件，未来React也会这种面向无状态组件在譬如无意义的检查和内存分配领域进行一系列优化，所以只要有可能，尽量使用无状态组件。

#####React.createClass

`React.createClass`是react刚开始推荐的创建组件的方式，这是ES5的原生的JavaScript来实现的React组件，其形式如下：

```js
var InputControlES5 = React.createClass({
    propTypes: {//定义传入props中的属性各种类型
        initialValue: React.PropTypes.string
    },
    defaultProps: { //组件默认的props对象
        initialValue: ''
    },
    // 设置 initial state
    getInitialState: function() {//组件相关的状态对象
        return {
            text: this.props.initialValue || 'placeholder'
        };
    },
    handleChange: function(event) {
        this.setState({ //this represents react component instance
            text: event.target.value
        });
    },
    render: function() {
        return (
            <div>
                Type something:
                <input onChange={this.handleChange} value={this.state.text} />
            </div>
        );
    }
});
```

与无状态组件相比，`React.createClass`和后面要描述的`React.Component`都是创建有状态的组件，这些组件是要被实例化的，并且可以访问组件的生命周期方法。

- React.createClass会自绑定函数方法（不像React.Component只绑定需要关心的函数）导致不必要的性能开销，增加代码过时的可能性。
- React.createClass的mixins不够自然、直观；React.Component形式非常适合高阶组件（Higher Order Components--HOC）,它以更直观的形式展示了比mixins更强大的功能，并且HOC是纯净的JavaScript，不用担心他们会被废弃。

#####React.Component

`React.Component`是以ES6的形式来创建react的组件的，是React目前极为推荐的创建有状态组件的方式，最终会取代`React.createClass`形式；相对于 React.createClass可以更好实现代码复用。将上面React.createClass的形式改为React.Component形式如下：

```javascript
constructor(props) {
        super(props);

        // 设置 initial state
        this.state = {
            text: props.initialValue || 'placeholder'
        };

        // ES6 类中函数必须手动绑定
        this.handleChange = this.handleChange.bind(this);
    }

    handleChange(event) {
        this.setState({
            text: event.target.value
        });
    }

    render() {
        return (
            <div>
                Type something:
                <input onChange={this.handleChange}
               value={this.state.text} />
            </div>
        );
    }
}
```

##### 区别

**this绑定方式**

`React.createClass`创建的组件，其每一个成员函数的this都有React自动绑定，任何时候使用，直接使用this.method即可，函数中的this会被正确设置。

```javascript
const Contacts = React.createClass({  
  handleClick() {
    console.log(this); // React Component instance
  },
  render() {
    return (
      <div onClick={this.handleClick}></div>
    );
  }
});
```

`React.Component`创建的组件，其成员函数不会自动绑定this，需要开发者手动绑定，否则this不能获取当前组件实例对象。

```javascript
//1. 构造函数中绑定
constructor(props) {
   super(props);
   this.handleClick = this.handleClick.bind(this); 
}

//2. 使用bind来绑定
<div onClick={this.handleClick.bind(this)}></div> 

//3. 使用箭头函数绑定
<div onClick={()=>this.handleClick()}></div> 
```

 **组件属性类型propTypes及其默认props属性defaultProps配置不同**

`React.Component`在创建组件时配置这两个对应信息时，他们是作为组件类的属性，不是组件实例的属性，也就是所谓的类的静态属性来配置的。对应上面配置如下：

```javascript
class TodoItem extends React.Component {
    static propTypes = {//类的静态属性
        name: React.PropTypes.string
    };

    static defaultProps = {//类的静态属性
        name: ''
    };

    ...
}
```

`React.createClass`在创建组件时，有关组件props的属性类型及组件默认的属性会作为组件实例的属性来配置。其中defaultProps是使用getDefaultProps的方法来获取默认组件属性的

```javascript
const TodoItem = React.createClass({
    propTypes: { // as an object
        name: React.PropTypes.string
    },
    getDefaultProps(){   // return a object
        return {
            name: ''    
        }
    }
    render(){
        return <div></div>
    }
})
```

 **组件初始状态state的配置不同**

`React.createClass`创建的组件，其状态state是通过`getInitialState`方法来配置组件相关的状态；
`React.Component`创建的组件，其状态state是在`constructor`中像初始化组件属性一样声明的。

```javascript
const TodoItem = React.createClass({
    // return an object
    getInitialState(){ 
        return {
            isEditing: false
        }
    }
    render(){
        return <div></div>
    }
})


class TodoItem extends React.Component{
    constructor(props){
        super(props);
        this.state = { // define this.state in constructor
            isEditing: false
        } 
    }
    render(){
        return <div></div>
    }
}
```

#####Mixins的支持不同

[`Mixins`](https://facebook.github.io/react/docs/reusable-components-zh-CN.html#mixins)(混入)是面向对象编程OOP的一种实现，其作用是为了复用共有的代码，将共有的代码通过抽取为一个对象，然后通过`Mixins`进该对象来达到代码复用。具体可以参考[React Mixin的前世今生](http://www.w3ctech.com/topic/1599)。

`React.createClass`在创建组件时可以使用`mixins`属性，以数组的形式来混合类的集合。

######react组件实例

这里的实例特指React组件的实例。React 组件是一个函数或类，实际工作时，发挥作用的是React 组件的实例对象。只有组件实例化后，每一个组件实例才有了自己的**props和state**，才持有对它的DOM节点和子组件实例的引用。在传统的面向对象的开发方式中，实例化的工作是由开发者自己手动完成的，但在React中，组件的实例化工作是由React自动完成的，组件实例也是直接由React管理的。换句话说，开发者完全不必关心组件实例的创建、更新和销毁。