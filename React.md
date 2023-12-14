# React

## JSX的本质

JSX就是允许我们在JS代码中穿插一些HTML的组件的语法，如下所示：

```react
...
render() {
	return (
  <div> 
    <button id="lhy">
      Hit
    </button>
  </div>
) // 这就是jsx语法，它不能被原生JS正常编译，会产生语法错误
}
```

JSX本质其实就是React.createElement的语法糖，当我们写出上述的JSX语法代码之后，Babel会帮我们自动将其解析成为React的代码。

React.createElement 会接受3个参数，分别是type, config, children，其中：

- type 为 html元素的类型
- config 为传入组件中的 props
- children 为该组件的子组件对象

上述的JSX代码经过babel转换后会成为如下代码

```js
/*#__PURE__*/ React.createElement(
  "div",
  null,
  /*#__PURE__*/ React.createElement(
    "button",
    {
      id: "lhy"
    },
    "Hit"
  )
);

```



## 虚拟DOM

从上一节中可以看到 React.createElement 最终会返回一个 ReactElement 对象，这个对象有一个 children 属性，其值也是一个 ReactElement 对象。这样就组成了一个由 ReactElement 组成的对象树，而这个对象树就是“虚拟DOM“

```jsx
(
  <div>
    <h2>
      <div>Hello</div>
    </h2>
    <button id="cmq">
      Hit
    </button>
    <button id="lhy">
      Hit
    </button>
  </div>
)
```

```js
/*#__PURE__*/ React.createElement(
  "div",
  null,
  /*#__PURE__*/ React.createElement(
    "h2",
    null,
    /*#__PURE__*/ React.createElement("div", null, "Hello")
  ),
  /*#__PURE__*/ React.createElement(
    "button",
    {
      id: "cmq"
    },
    "Hit"
  ),
  /*#__PURE__*/ React.createElement(
    "button",
    {
      id: "lhy"
    },
    "Hit"
  )
);
// 这颗对象树就是我们的虚拟DOM
```

因此，我们整个JSX-真实DOM的流程就是：JSX代码经过babel编译，得到ReactElement对象树（虚拟DOM），再经过原生平台的渲染，成为真实DOM

虚拟DOM最关键的两个作用：

1. 提高性能：通过虚拟DOM，我们不用每次在页面更新的时候，都去更新整个真实DOM，而是可以在新虚拟DOM和老虚拟DOM之间进行一个快速Diff算法，来得出需要更新的部分，并且只更新这一部分，从而达到提升性能的目的
2. 多平台支持：虚拟DOM只是一个JS对象，这代表它具有更高的灵活性。在Web开发中，它可以被 `document.createElement` 这种HTML的语法渲染成HTML组件，同样的，React-Native可以将这个JS对象渲染成苹果，安卓等平台的原生组件

## 脚手架

脚手架是即用的工程项目模版，每个框架都有自己的脚手架。React的脚手架为`create-react-app`, 它使用node.js编写，打包工具则是基于webpack。

```cmd
# install scaffold
npm install create-react-app -g

# create your app
cd path/to/folder
create-react-app your-app-name

# run your app
cd your-app-name/
npm run start # we can see the command in package.json
```

脚手架目录结构介绍

![image-20231121215939091](./React.assets/image-20231121215939091.png)

## 组件的类型划分

React是基于组件式开发的思想，组件是构成整个React APP的基本单位。根据不同的准则，可以将React中的组件划分为：

1. 按照声明方式划分：类组件和函数式组件
2. 按照有无状态管理划分：有状态组件和无状态组件
3. 按照职责划分：容器组件和渲染组件（容器组件注重数据的逻辑计算，而渲染组件主要用于UI渲染）

## JSX组件能够返回的数据类型

当类组件的render函数或者函数组件的return语句被调用时，它会检查`this.props`和`this.state`的变化并返回以下类型之一：

1. React元素：即我们通过JSX语法创建的东西（<div>hahaha</div>），HTML的原生组件如div，会被渲染成DOM节点，而<MyComponent>这种则会被渲染成自定义组件
2. 字符串或数值类型：会在DOM中被渲染成文本节点
3. 布尔类型或者Null：什么都不渲染
4. 数组或者Fragments：使render方法能够返回多个元素，这些元素会被遍历并渲染
5. Portals：可以使该元素被渲染到不同的DOM子树当中去

## 组件的生命周期

组件的生命周期仅用于类组件，函数组件是没有明确的生命周期及其函数定义的。在函数组件当中，我们通常是使用各类hook函数来模拟生命周期。下面是简化版和详细版的生命周期示意图。

![React lifecycle methods diagram](./React.assets/ogimage.png)

![React Lifecycle Methods - A Deep Dive - Programming with Mosh](./React.assets/Screen-Shot-2018-10-31-at-1.44.28-PM.png)

1. Constructor

   当要渲染一个组件的时候，第一步就是先实例化该组件，得到一个JSX Element。类组件的构造函数通常只做两件事情：

   - 初始化内部的state
   - 为事件绑定this实例

   因此，如果不需要在类组件内进行state初始化或者this绑定，则不需要为类组件写构造函数

2. componentDidMount

   该函数会在组件挂载（插入DOM树中）后立即调用。

   该生命周期阶段通常适用于进行下列操作：

   - 依赖于DOM的操作
   - 发送网络请求
   - 添加一些订阅（取消订阅则在componentWillUnmount中进行）

3. componentDidUpdate

   该函数会在组件在DOM中被更新后被立即调用，首次渲染则不会执行此方法。

   该生命周期阶段通常适用于进行下列操作：

   - 当组件更新后，可以对DOM进行一些操作
   - 如果对更新前后的props进行了比较，也可以选择在此处进行网络请求（若props更新则发送网络请求）

4. componentWillUnmount

   该函数会在组件卸载及销毁之前直接调用。

   该生命周期阶段通常适用于进行下列操作：

   - 执行必要的清理操作，防止内存泄漏。例如清除timer、取消未完成的网络请求以及取消订阅等等

5. getDerivedStateFromProps（罕见，几乎不使用）

   state的值在任何时候都依赖于props的时候使用。该方法返回一个对象来更新state

6. shouldComponentUpdate

   该函数会在props和state改变后，再次调用render函数之前调用。

   该函数用于自定义组件重渲染规则。

7. getSnapshotBeforeUpdate

   该函数会在DOM更新之前执行，可以获取DOM更新前的一些信息（比如说，我们可以通过该函数返回DOM更新前的滚动位置，这个位置信息将在componentDidUpdate的参数列表中拿到，然后我们可以把新旧两个滚动位置进行对比，根据结果实现一些操作）

   ```jsx
   componentDidUpdate(prevProps, prevState, snapshot) {
     console.log(snapshot.scrollPosition) // 在这里可以拿到getSnapshotBeforeUpdate返回的值
   }
   
   getSnapshotBeforeUpdate() {
     return {
       scrollPosition: this.getScrollPosition()
     }
   }
   ```

   

## 组件间通信

### 参数类型验证和默认参数

如果使用的是Typescript，那么语言特性直接就能做到参数类型验证。

如果使用的是Javascript，那么需要使用到 `propTypes` 来进行参数类型验证

```javascript
import React from 'react'
import PropTypes from "prop-types"

export function MainBanner (props) {
  ...
}

// 参数类型验证
MainBanner.propTypes = {
  banners: PropTypes.string,
  title: PropTypes.string
}
  
// 默认参数
MainBanner.defaultProps = {
  banner: "defaultBanner",
  title: "defaultTitle"
}
```

### 子组件传递消息给父组件

子传父通常需要父组件将一个回调函数传递给子组件，这个回调函数中通常会对父组件的状态进行一些操作，当子组件调用该回调函数，则会对父组件的状态进行更新。

```jsx
//parent component
import React, { useState } from "react";
import AddCounter from "./AddCounter";

export default function App() {
  const [count, setCount] = useState(0);
  const updateCount = (num) => setCount(count + num)
  return (
    <>
      <h2>Current Counter: {count}</h2>
      <AddCounter setCountFn={updateCount}/> // pass the callback to son component
    </>
  );
}
```

```jsx
// son component
import React from "react";

export default function AddCounter(props) {
    const changeCount = num => props.setCountFn(num)
  return (
    <>
      <button onClick={() => changeCount(1)}>+1</button>
      <button onClick={() => changeCount(-1)}>-1</button>
    </>
  );
}
```

### 插槽 (slot)

插槽是Vue和小程序中的概念。当一个组件预留了一个插槽，则这个插槽可以插入任何组件。这通常适用于一个组件被应用于多个不同上下文的场景。比如说某个导航栏组件，它在商品页面的上下文时，可能由搜索框组成，而在订单页面的上下文时，它可能是由 ”已发货“、”已收货“ 等等tab所组成。这时我们往导航栏组件中预留一些插槽，在需要的时候，通过插槽的名字来往对应插槽中插入需要的组件（又称具名插槽）。

React中没有插槽这一概念，反之，得益于React的灵活性，我们完全可以在React中非常方便地手动实现插槽。

1. 通过children来实现插槽 （不推荐）

这种方式要通过数组索引的方式来取得children当中的组件，同时，如果children当中有多个元素的时候，children的类型会是一个由JSX元素组成的数组，而如果它只有一个元素的时候，它的类型就直接是一个JSX元素。因此，这种写法非常容易出错，通过数组索引的方式也非常不优雅，因此不推荐。

```jsx
import React from 'react'
import NavBar from './nav-bar/NavBar'

export default function App() {
  return (
    <NavBar>
        <div>left</div>
        <i>center</i>
        <button>right</button>
    </NavBar>
  )
}

```

```jsx
import React from 'react'
import './NavBar.css'

export default function NavBar(props) {
  const children = props.children
  return (
    <div className='nav-bar'>
        <div className='left'>{children[0]}</div>
        <div className='center'>{children[1]}</div>
        <div className='right'>{children[2]}</div>
    </div>
  )
}

```

2. 通过往props中传递JSX元素来实现插槽

这种方式更加接近具名插槽

```jsx
import React from 'react'
import NavBar from './nav-bar/NavBar'

export default function App() {
  return (
    <NavBar
        left={<div>left</div>}
        center={<i>center</i>}
        right={<button>right</button>}
    />
  )
}
```

```jsx
import React from 'react'
import './NavBar.css'

export default function NavBar(props) {
  return (
    <div className='nav-bar'>
        <div className='left'>{props.left}</div>
        <div className='center'>{props.center}</div>
        <div className='right'>{props.right}</div>
    </div>
  )
}

```

#### 作用域插槽

在React中，当我们把JSX元素以props或children的形式传给目标组件后，我们想让这些JSX元素使用目标组件中的数据或者状态，这就是作用域插槽的应用场景。作用域插槽在React中的解决方案就是子传父的组件通信，通过给目标组件传递一个**“参数为状态或数据，返回值为JSX元素”**的回调函数，从而达到作用域插槽的目标。

```jsx
import React from 'react'
import NavBar from './nav-bar/NavBar'

export default function App() {
  const leftContextSlot = text => <div>{text}</div>
  const centerContextSlot = text => <i>{text}</i>
  const rightContextSlot = text => <button>{text}</button>
  return (
    <NavBar
        left={leftContextSlot}
        center={centerContextSlot}
        right={rightContextSlot}
    />
  )
}

```

```jsx
import React from 'react'
import './NavBar.css'

export default function NavBar(props) {
  return (
    <div className='nav-bar'>
        <div className='left'>{props.left("myLeft")}</div>
        <div className='center'>{props.center("myCenter")}</div>
        <div className='right'>{props.right("myRight")}</div>
    </div>
  )
}

```

### 非父子组件间通信

#### context (详见[useContext](###useContext()))

#### 状态提升

状态提升就是让两个组件拥有同一个parent，然后把这两个兄弟组件直接通讯需要的状态都存在parent组件里面

```jsx
export const ParentComponent:React.FC<IParentProps> = (props) => {
    const [num, setNum] = setState(0) // 状态提升，Alpha和Beta通讯的状态被存储在共同的parent组件中
    const numIncrement = (n: number) => {
        setNum(n)
    }
    return (
    	<ChildComponentAlpha num={num}></ChildComponentAlpha>
        <ChildComponentBeta numIncrement={numIncrement}></ChildComponentBeta>
    )
}


export const ChildComponentAlpha: React.FC<IAlphaProps> = ({num}) => {
    return (<div>The current num is: {num} </div>) //ChildAlpha显示ChildBeta设置的随机数
}

export const ChildComponentBeta: React.FC<IBetaProps> = ({numIncrement}) => {
    const randomNumber = Math.floor(Math.random() * 100) // ChildBeta使用一个随机数与ChildAlpha通讯
    const clickHandler = () => {
        numIncrement(randomNumber);
    }
    return (<button onclick={clickHandler}>Click Me!</button>)
}
```

#### 事件总线

在JavaScript中，事件总线是一种设计模式，用于简化组件之间的通信。它允许不同组件在不直接耦合的情况下进行通信，通过在一个中心位置注册、触发和监听事件。这个中心位置被称为事件总线。

事件总线通常有两个主要部分：事件的触发者和事件的监听者。一个组件触发一个事件，而其他组件则监听该事件以执行相应的操作。这种模式有助于将代码解耦，提高组件的可维护性和可扩展性。

```jsx
import React from 'react'
import SiblingA from './SiblingA'
import SiblingB from "./SiblingB"


export default function App() {
  return (
    <>
      <h2>EventBusAPP</h2>
      <SiblingA></SiblingA>
      <SiblingB></SiblingB>
    </>
  )
}

```

```jsx
import React from "react";
import eventBus from "./utils/eventBus"; // 从第三方库引入事件总线对象

export default function SiblingA() {

  const increaseHandler = () => {
    eventBus.emit("increase") // 发送事件
  }

  const decreaseHandler = () => {
    eventBus.emit("decrease") // 发送事件
  }
  return (
    <>
      <button onClick={increaseHandler}>Sibling A: Let B + 1</button>
      <button onClick={decreaseHandler}>Sibling A: Let B - 1</button>
    </>
  );
}
```

```jsx
import React, { useEffect, useState } from "react";
import eventBus from "./utils/eventBus";

export default function SiblingB() {
  const [num, setNum] = useState(0);

  useEffect(() => {
    console.log("useEffect triggered!")
    const increaseHandler = () => setNum(prevNum => prevNum + 1);
    const decreaseHandler = () => setNum(prevNum => prevNum - 1);
    eventBus.on("increase", increaseHandler); // 监听事件
    eventBus.on("decrease", decreaseHandler); // 监听事件

    return () => eventBus.clear();
  }, []);
  return <div>SiblingB's current num: {num}</div>;
}
```

## setState的三种用法

1. 第一个参数传入一个对象

​	这种方式是最普遍的，当传入一个对象后，React会使用`Object.assign(prevState, newState)`的方式去合并新的状态对象和老的状态对象，从而完成状态的更新

2. 第一个参数传入一个函数

   这种方式适用于当你想在更新状态的时候根据前一个时间步的状态和props做一些计算来得到新状态的情况。

   ```jsx
   ...
   const clickHandler = () => {
     this.setState((prevState, prevProps) => {
       const newState = codeToCalculateNewState(prevState, prevProps)
       return newState
     })
   } 
   ```

   当然，我们也可以把这个计算的逻辑单独抽成一个函数，然后把计算结果通过第一种用法传给setState。但是如果直接写在setState内会让内聚性更好。

3. 第二个参数传入一个callback函数

   setState 是一个异步函数，当我们想要在这个异步函数执行完成之后做一些事情，就可以把这个逻辑抽成一个回调函数，传给setState的第二个参数。

   ```jsx
   this.setState(newState, () => {
     console.log('successfully set new state.')
   })
   ```

## setState为什么要设计成异步函数

1. 性能优化：减少重渲染次数

   不管是类组件的setState还是函数组件的状态管理函数，它们都是异步的。这样做的原因之一是为了减少重渲染次数。React设计了一种叫做“批处理”的机制来延迟状态的实际更新和重新渲染，以提高性能。比如说，当一个button的onClick函数中调用了多次setState函数时，这些调用都会加入本轮事件循环中的微任务队列，React会通过do while函数将这个队列里面的状态变化事件全部取出来，将其依次计算并合并最后的结果，因此最终实际只会发生一次重渲染。

   如果setState是同步函数，那每调用一次就会做一次重渲染，大大降低性能

2. setState函数如果是同步函数，它被执行之后，状态立马改变，但这个时候还没有执行render函数，如果父组件将state的值传给了子组件，那么子组件的props和父组件的state将不能保持同步，这种不一致性会在开发中导致很多问题

## React性能优化相关知识

### 虚拟DOM的Diff算法

当重渲染被触发时，React的render方法会被调用，从而创建出一个新的虚拟DOM。在新的虚拟DOM生成之后，React会将新老虚拟DOM进行一次Diff计算，以判断如何来更新真实DOM，这个Diff算法有几个特点

1. 复杂度为 $O(n)$
2. 仅仅会比较同层节点，不会跨层比较
3. 不同类型的节点会产生不同的树结构：父节点类型的变化会导致子树被完全重新生成，不会再做比较

### 通过 shouldComponentUpdate, PureComponent, memo 来避免不必要的重渲染

在React当中，如果父节点被重新渲染，那么默认情况下子组件也会被重渲染。但很多时候，子组件可能只依赖于父组件的部分状态或props，甚至根本就不依赖父组件的任何状态，这个时候如果导致父节点重渲染的状态变化并不会影响子组件的时候，重渲染该子组件就是一种性能浪费。因此React提出了一系列方式来优化这种性能浪费。

#### shouldComponentUpdate(SCU)

类组件中，我们可以在该生命周期函数中定义我们期望的重渲染条件，该函数会在render函数被调用之前执行。

#### PureComponent

类组件中，我们可以使组件继承 PureComponent 这个类，这样的话，只有该组件的props或者state发生变化时，该组件才会被重渲染。

PureComponent的实质是对新老state和props做一个浅层比较（对于复杂对象只比较第一层）:

```jsx
if(ctor.prototype && ctor.prototype.isPureReactComponent) {
  return (
  	!shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
  )
}
```

#### memo

函数组件中，memo的作用就类似于PureComponent，通过用memo包裹函数组件，可以只在props或者state更新的情况下重渲染组件

## 数据不可变

在React中更新状态时，一定要确保新状态对象和老状态对象指向的是不同的引用。原因很简单，如果我们直接在老状态对象上做修改，并且直接把修改后的老状态对象传给 setState 或者状态管理函数，那么在 PureComponent 或者函数组件做新老状态对比的时候，**新状态和老状态将指向同一引用**，这样的话组件将永远不会更新。

永远保证新状态对象指向新的引用，不要直接修改老状态，这就是React中的数据不可变。

对于不可变数据类型，创建新值的同时就已经让该值指向新引用了。

对于可变数据类型，如果我们只是使用PureComponent或者Memo，那么我们可以直接对其进行浅拷贝即可，否则，如果我们对状态有更严格的对比要求，且可变数据类型的子元素也包含可变数据类型，则需要进行深拷贝。这跟React的 PureComponent 中的SCU算法 `shallowEqual ` 有关系，该算法会对新老状态对象进行浅层对比，该算法伪代码如下：

```jsx
function shallowEqual(objA, objB) {
  // 1. 判断A和B是不是同一对象
  if(is(objA,objB)){
    return true;
  }
  
  // 2. 判断A和B是否为可变数据类型
  if(typeof objA !== 'object' || objA === null || typeof objB !== 'object' || objB == null) {
    return false;
  }
  
  // 3. 如果A和B均为可变数据类型
  
  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);
  
  // 3.1. 如果keys长度不同，直接返回false
  if (keysA.length !== keysB.length) {
    return false;
  }
  
  // 3.2. 否则，逐一比较每个B是否有A中的所有key，以及其对应value是否为同一对象
  for(let i = 0; i < keysA.length; i++) {
    const currentKey = keysA[i];
    if(!hasOwnProperty.call(objB, currentKey) || !is(objA[currentKey], objB[currentKey])) {
      return false; // 只要有一个键值不存在或者引用不同，则不相等
    }
  }
  
  return true;
}
```

## Ref

ref在React当中可以用来获取原生DOM或者类组件的实例，前者可以让我们做一些原生DOM操作，而后者可以让我们直接调用组件实例中的public方法。

```jsx
import React, { PureComponent, createRef } from 'react'
import HelloWorld from './HelloWorld'

export class App extends PureComponent {
  constructor() {
    super()
    this.h2Ref = createRef()
    this.hwRef = createRef()
  }

  componentDidMount() {
    console.log(`h2Ref: ${this.h2Ref.current}`)
    console.log(`hwRef: ${this.hwRef.current}`)
  }

  sayGoodbye() {
    this.hwRef.current.sayGoodbye()
  }
  render() {
    return (
      <>
      <h2 ref={this.h2Ref}>h2 element</h2>
      <HelloWorld ref={this.hwRef}></HelloWorld>
      <button onClick={this.sayGoodbye.bind(this)}>Say Goodbye</button>
      </>
    )
  }
}

export default App
```

```jsx
import React, { PureComponent } from 'react'

export class HelloWorld extends PureComponent {
  sayGoodbye () {
    console.log("goodbye world")
  }
  render() {
    return (
      <div>HelloWorld element</div>
    )
  }
}

export default HelloWorld
```

Console 输出：

```
h2Ref: [object HTMLHeadingElement]
hwRef: [object Object]
```

点击按钮后：

```
goodbye world
```

函数式组件由于没有实例，所以无法用上面的方法进行绑定，但是函数式组件可以进行ref转发。如果想将ref绑定给函数式组件，需要用 `React.forwardRef` 包裹该组件。通过这样的方式，我们可以进一步地将ref绑定到函数式组件的某一个子元素上面。

```jsx
import React, { PureComponent, createRef } from 'react'
import FunctionComponent from './FunctionComponent'

export class App extends PureComponent {
  constructor() {
    super()
    this.fcRef = createRef()
  }

  componentDidMount() {
    console.log(`fcRef: ${this,this.fcRef}`)
  }


  render() {
    return (
      <FunctionComponent ref={this.fcRef}></FunctionComponent>
    )
  }
}

export default App
```

```jsx
import React from "react";

const FunctionComponent = React.forwardRef(function FunctionComponent(
  props,
  ref
) {
  return (
    <>
      <h2 ref={ref}>FC H2</h2>
      <div>FC Div</div>
    </>
  );
});

export default FunctionComponent;
```

Console 输出：

```
fcRef: [object Object]
```

## 受控组件和非受控组件

当表单元素（`<input>, <textarea>, <select>` 等）的值属性被绑定给一个react的state时，必须为该组件指定 `onChange` handler，在该handler中，将组件接收到的最新值时，将这个值又更新给react的state（如果不这么做，那这个表单元素将会变成一个无法输入的只读元素）。这就是双向绑定，被双向绑定的组件叫做受控组件。

```jsx
import React from 'react'

export default function App() {
  const [text, setText] = React.useState("default text");
  return (
    {/* 受控组件 */}
    <input type='text' value={text} onChange={e => {setText(e.target.value)}}></input>
    {/* 非受控组件 */}
		<input type='text'></input>
  )
}
```

### 同一函数管理多个受控组件

在类组件中，由于state是一个对象，所以我们可以把多个受控组件的name元素以key的形式保存在state对象当中，当表单元素的value变化，需要更新的时候，这个表单元素的event.target就指向了这个表单元素，我们可以通过`event.target.name` 拿到name元素，从而更新state对象中对应的state。

```jsx
import React, { PureComponent } from 'react'

export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      username: "",
      password: ""
    }
  }

  onInputChanged = (event) => {
    this.setState({
      [event.target.name]: event.target.value // 按照event.target.name更新对应的状态
    })
  }

  onSubmit = (e) => {
    e.preventDefault(); // 防止以默认行为提交表单
    console.log(`Submitted! username: ${this.state.username}, password:${this.state.password}`)
  }
  render() {
    return (
      <form onSubmit={e => this.onSubmit(e)}>
        <label> {/* 将文本和input放在一个label当中，可以让用户点击文本的时候，也能够选中input，提高accessibility*/}
          username:
          <input type="text" name="username" value={this.state.username} onChange={e => this.onInputChanged(e)}/>
        </label>
        <label>
          password:
          <input type='password' name="password" value={this.state.password} onChange={e => this.onInputChanged(e)}/>
        </label>
        <button type='submit'>Submit</button>
      </form>
    )
  }
}

export default App
```

### 其他常见的受控组件

#### checkbox（单选，多选）以及 select 下拉框（单选，多选）

- Checkbox 单选：值字段为 checked 而不是 value
- Checkbox 多选：在单选基础上，需要用一个由对象组成的数组来管理状态
- Select 单选：值字段为value，子元素为option标签，option标签的value字段就是select的值字段中的候选值
- Select多选：在单选基础上，需要在select标签上加上 `multiple={true}`，其value字段必须是一个数组，内部存的依然是option标签中value字段里的值。

```jsx
import React, { PureComponent } from "react";

export class App extends PureComponent {
  constructor() {
    super();
    this.state = {
      username: "",
      password: "",
      complianceChecked: false,
      hobbies: [
        { name: "sing", id: 0, text: "sing", checked: false },
        { name: "dance", id: 1, text: "dance", checked: false },
        { name: "rap", id: 2, text: "rap", checked: false },
        { name: "basketball", id: 3, text: "basketball", checked: false },
      ],
      fruit: "orange",
      beverages: ["beer"],
    };
  }

  onInputChanged = (event) => {
    this.setState({
      [event.target.name]: event.target.value,
    });
  };

  onComplianceChanged = (e) => {
    this.setState({
      complianceChecked: e.target.checked,
    });
  };

  onHobbiesChanged = (e) => {
    const newHobbies = [...this.state.hobbies];
    newHobbies.map((hobby) => {
      if (hobby.name === e.target.name) {
        hobby.checked = e.target.checked;
      }
      return hobby;
    });
    this.setState({ hobbies: newHobbies });
  };

  onFruitChanged = (e) => {
    this.setState({ fruit: e.target.value });
  };

  onBeverageChanged = (e) => {
    this.setState({
      beverages: Array.from(e.target.selectedOptions, (item) => item.value),
    });
  };

  onSubmit = (e) => {
    e.preventDefault();
    console.log(
      `Submitted! username: ${this.state.username}, password:${
        this.state.password
      }, checkedHobbies: ${this.state.hobbies
        .filter((hobby) => hobby.checked === true)
        .map((item) => item.text)}, fruit: ${this.state.fruit}, beverages: ${this.state.beverages}`
    );
  };
  render() {
    return (
      <form onSubmit={(e) => this.onSubmit(e)}>
        <div>
          <label>
            username:
            <input
              type="text"
              name="username"
              value={this.state.username}
              onChange={(e) => this.onInputChanged(e)}
            />
          </label>
          <label>
            password:
            <input
              type="password"
              name="password"
              value={this.state.password}
              onChange={(e) => this.onInputChanged(e)}
            />
          </label>
        </div>
        <label>
          {/* checkbox 单选 */}
          <input
            type="checkbox"
            name="compliance"
            checked={this.state.complianceChecked}
            onChange={(e) => this.onComplianceChanged(e)}
          ></input>{" "}
          I've known all compliance rules
        </label>
        <div>
          {/* checkbox 多选 */}
          {this.state.hobbies.map((item) => (
            <>
              <label htmlFor={item.name} key={item.id}>
                <input
                  type="checkbox"
                  checked={item.checked}
                  id={item.name}
                  name={item.name}
                  onChange={(e) => this.onHobbiesChanged(e)}
                />
                {item.text}
              </label>
            </>
          ))}
        </div>
        {/* select 单选 */}
        <select
          value={this.state.fruit}
          onChange={(e) => this.onFruitChanged(e)}
        >
          <option value="orange" key="orange">
            Orange
          </option>
          <option value="banana" key="banana">
            Banana
          </option>
          <option value="apple" key="apple">
            Apple
          </option>
        </select>
        {/* select 多选 */}
        <select
          value={this.state.beverage}
          onChange={(e) => this.onBeverageChanged(e)}
          multiple={true}
        >
          <option value="beer" key="beer">
            Beer
          </option>
          <option value="wine" key="wine">
            Wine
          </option>
          <option value="cola" key="cola">
            Cola
          </option>
        </select>
        <button type="submit">Submit</button>
      </form>
    );
  }
}

export default App;

```

### 非受控组件

受控组件的数据是由React的状态来管理的。而非受控组件的数据则是由DOM节点来处理。在React中，大多数情况下我们都使用受控组件。

如果我们要使用非受控组件中的数据，那么我们需要使用ref来从DOM节点中获取表单数据。

```jsx
import React, { PureComponent } from "react";

export class App extends PureComponent {
  constructor() {
    super();
    this.inputRef = React.createRef();
  }
  submitHandler = (e) => {
    e.preventDefault();
    console.log(`current input text: ${this.inputRef.current.value} `);
  };
  render() {
    return (
      <>
        <form onSubmit={(e) => this.submitHandler(e)}>
          <label htmlFor="inputArea"> Input: </label>
          <input type="text" defaultValue={'...'} ref={this.inputRef} />
          <button type="submit">Submit</button>
        </form>
      </>
    );
  }
}

export default App;

```

## 高阶组件

### 高阶函数

至少满足以下两个条件之一的函数称为高阶函数：

1. 接受一个函数或多个函数作为输入
2. 返回一个函数

JS当中的 map, filter 等等都是高阶函数

### 高阶组件

高阶组件定义：接收一个组件作为参数，返回值为一个新组件的函数，这样的函数就叫做高阶组件。（注意，高阶组件实质上是一个函数，而不是组件）

高阶组件的意义是对传入组件的渲染进行拦截，从而在高阶组件函数内对传入的组件进行一些操作，这在需要对多个不同组件进行相同操作（如参数注入，context/gql注入或者登录鉴权）的时候非常有用。

下面举一个利用高阶组件进行context注入的例子：

```jsx
// ThemeContext.js
import React from 'react'

const ThemeContext = React.createContext({});

export default ThemeContext
```

```jsx
// WithThemeContext.jsx
import ThemeContext from "../ThemeContext";

const withThemeContext = (OriginalComponent) => {
  return (props) => (
    <ThemeContext.Consumer>
      {/* 利用高阶组件拦截传入组件的渲染，并为传入组件注入一个 Context Value */}
      {(value) => <OriginalComponent {...props} {...value} />}
    </ThemeContext.Consumer>
  );
};

export default withThemeContext;
```

```jsx
// App.jsx
import React, { PureComponent } from "react";
import withThemeContext from "./hoc/WithThemeContext";

export class App extends PureComponent {
  constructor(props) {
    super(props);
  }
  render() {
    return <h2>Theme:{this.props.theme}</h2>;
  }
}
// 直接导出高阶组件
export default withThemeContext(App);

```

## Portals

portal是React提供的一种跨层次的渲染方式，通常，当我们从组件返回一个元素时，它会作为最近父节点的子节点挂载到 DOM 的层次结构当中。

```react
export default Father(props:IProps){
  return (
    <div>
      {props.children} //该组件会直接作为Father的子组件挂载在DOM当中
    </div>
  );
}
```

但是，在某些情况下，我们需要把子组件直接渲染到DOM当中其他层次的位置当中去，比如说对话框、悬浮卡片、工具提示等需要突破原有容器的场景，这个时候我们就需要直接把子组件插入到其他层次的节点当中进行渲染，这时候就需要用到React中的Portal

```react
import ReactDOM from 'react-dom'
ReactDOM.createPortal(child, container)
```

其中，child就是直接可渲染的一个组件，container就是一个已存在于DOM的组件，child将被挂载到这个container下

```html
<!--index.html-->
<html>
  ...
  <body>
    ...
    <div id="root"></div>
    <div id="lhy"></div>
  </body>
</html>
```

```jsx
import React, { memo } from 'react'
import { createPortal } from 'react-dom'

const App = memo(() => {
  return (
    <div className='aaa'>
        <h2>H2</h2>
        {
            createPortal(<h3>H3</h3>, document.getElementById('lhy'))
        }
    </div>
  )
})

export default App
```

运行结果：

![image-20231213224941930](./React.assets/image-20231213224941930.png)



## 严格模式

`<StrictMode>` 是一个用来突出显示应用程序中存在的潜在问题的工具，它在React当中以组件的形式存在的：

1. 严格模式不会渲染额外的UI
2. 为后代元素触发额外的检查和警告
3. 仅在开发环境下有用，不会影响生产环境

严格模式主要检查：

1. 识别过时和不安全的函数（如过时的生命周期函数，Ref API等等）
2. 检查意外的副作用：严格模式下的**组件在初次挂载的时候会渲染两次**
   - 如果在挂载的时候添加了一些副作用，同时我们忘记在取消挂载的时候删除这些副作用，如果我们在这两个生命周期中有一些console输出，那渲染两次会让我们更清楚地在console中看到此类错误。

## 过渡动画

组件的过渡动画是指伴随组件状态变化过程的动画，这个状态变化可以是组件在消失/出现之间的转变，也可能是新/旧组件之间的更替。

React当中，我们可以使用官方社区维护的包 `react-transition-group` 来为组件添加过渡动画。

该库的官方文档地址：[React Transition Group](https://reactcommunity.org/react-transition-group/transition)

``` cmd
npm install react-transition-group
```

### CSSTransition

如果我们想使用CSS来控制过渡动画的行为，那么就需要使用 `CSSTransition`，它以组件形式存在于React当中，在我们只控制单个组件的消失/出现过渡动画的时候，可以单独使用这个过渡动画组件将我们的目标组件包裹起来。在其他时候，该过渡动画组件可以配合 `SwitchTransition` 和 `TransitionGroup` 实现更多的过渡动画效果。

CSSTransition的过渡动画执行过程中，有三个状态：

1. appear：这是组件在初次挂载时，从未显示到显示的状态
2. enter：这是组件从未显示到显示的状态
3. exit：这是组件从显示到未显示到状态

这三个状态，都分别包含三个过程，为这三个过程我们需要定义对应的CSS样式：

- 第一个过程 - 预备开始执行动画，对应的CSS类是：`-appear` , `-enter`, `-exit`
- 第二个过程 - 开始执行动画，对应的CSS类是：`-appear-active` , `-enter-active`, `-exit-active`
- 第三个过程 - 执行动画结束，对应的CSS类是：`-appear-done` , `-enter-done`, `-exit-done`

一般来说，第一个过程定义了动画初始状态，第二个过程定义了动画结束时的状态、变化需要的时间以及如何变化。第三个过程在动画结束后，为组件添加对应的额外样式。

CSSTransition 有几个关键的参数：

1.  `in` ：表明被包裹的组件是显示还是未显示

2. `timeout`：整个过渡动画最大时间，以毫秒为单位，这个值最好跟CSS中每个状态的过渡动画的执行时间一致

3. `classNames`：CSS类名，这个类名的设置比较特殊，如果我为其赋值为`lhy` ，那么在CSS文件中，我只需要像如下这样写就可以完成对三个状态的过渡动画执行过程的CSS样式的定义，在运行时，CSSTransition会根据组件的状态自行改变组件的CSS类

4. `nodeRef` ：目标组件的引用，如果不填写该参数，CSSTransition 会使用过时的API `findDomNode` 来寻找被目标组件，来改变它的CSS样式。填写该参数有助于规避该风险

   ```css
   .lhy-appear {
   	...
   }
   
   .lhy-appear-active {
     ...
   }
   
   .lhy-enter {
     ...
   }
   
   .lhy-enter-active {
     ...
   }
   
   .lhy-exit {
     ...
   }
   
   .lhy-exit-active {
     ...
   }
   ```

   

下面展示使用 CSSTransition 为一个文本标签实现 消失/出现 过渡动画的例子

```jsx
import React, { PureComponent, createRef } from 'react'
import { CSSTransition } from 'react-transition-group'
import "./style.css"

export class App extends PureComponent {
  constructor(props) {
    super(props)
    this.state = {
      isShow: true
    }
    this.h2Ref = createRef(null)
  }
  render() {
    return (
      <>
      <button onClick={() => this.setState({isShow: !this.state.isShow})}>Toggle</button>
      <CSSTransition in={this.state.isShow} classNames="lhy" timeout={1000} unmountOnExit={true} refNode={this.h2Ref}>
        <h2 ref={this.h2Ref}>Hahahahaha</h2>
      </CSSTransition>
      </>
    )
  }
}

export default App
```

```css
.lhy-appear {
  opacity: 0;
}

.lhy-appear-active {
  opacity: 1;
  transition: opacity 1s ease;
}

.lhy-enter {
  opacity: 0;
}

.lhy-enter-active {
  opacity: 1;
  transition: opacity 1s ease;
}

.lhy-exit {
  opacity: 1;
}

.lhy-exit-active {
  opacity: 0;
  transition: opacity 1s ease;
}
```

### SwitchTransition

当组件的状态变化时，我们想用过渡动画来展示两种不同状态下的组件变换的过程，这个时候就可以用SwitchTransition。

SwitchTransition的需要和CSSTransition配合起来使用。SwitchTransition需要包裹CSSTransition，而CSSTransition又包裹目标组件。和单独使用CSS时最大的不同点在于，CSSTransition不再使用 `in` 参数，而是变成了 `key` 参数，我们需要为每种状态下的组件绑定一个独一无二 key，同时也需要绑定独一无二的ref

```jsx
import React, { PureComponent, createRef } from "react";
import { SwitchTransition, CSSTransition } from "react-transition-group";
import "./style.css";

export class App extends PureComponent {
  constructor(props) {
    super(props);
    this.state = {
      isLogin: false,
    };
    this.exitRef = createRef(null);
    this.loginRef = createRef(null);
    this.nodeRef = this.state.isLogin ? this.loginRef : this.exitRef;
  }
  render() {
    return (
      <SwitchTransition>
        <CSSTransition
          key={this.state.isLogin ? "exit" : "login"}
          classNames="login"
          timeout={1000}
          nodeRef={this.nodeRef}
        >
          <button
            onClick={() => this.setState({ isLogin: !this.state.isLogin })}
            ref={this.nodeRef}
          >
            {this.state.isLogin ? "Logout" : "Login"}
          </button>
        </CSSTransition>
      </SwitchTransition>
    );
  }
}

export default App;

```

```css
.login-enter {
  transform: translateX(100px);
  opacity: 0;
}

.login-enter-active {
  transform: translateX(0);
  opacity: 1;
  transition: all 1s ease;
}

.login-exit {
  transform: translateX(0);
  opacity: 1;
}

.login-exit-active {
  transform: translateX(-100px);
  opacity: 0;
  transition: all 1s ease;
}

```

### TransitionGroup

当我们的数据以列表的形式展现，而我们想为列表中的每一条数据的增加/删除都添加过渡动画时，使用TransitionGroup是再好不过了。TransitionGroup的使用方法和SwitchTransition基本类似，需要和CSSTransition配合使用，同时需要为每个CSSTransition分配独一无二的key和ref。

TransitionGroup可以接受一个叫component的参数，这个参数可以指定列表元素的父组件类型，默认为div，但我们通常都会使用ul标签。
```jsx
import React, { PureComponent, createRef } from "react";
import { TransitionGroup, CSSTransition } from "react-transition-group";
import { v4 as uuid } from "uuid";
import "./style.css";

const FAKE_BOOK_NAME = "Book";
const FAKE_BOOK_PRICE = 100;
export class App extends PureComponent {
  constructor(props) {
    super(props);
    this.state = {
      books: [
        {
          bookName: FAKE_BOOK_NAME,
          price: FAKE_BOOK_PRICE,
          nodeRef: createRef(null),
          id: uuid(),
        },
        {
          bookName: FAKE_BOOK_NAME,
          price: FAKE_BOOK_PRICE,
          nodeRef: createRef(null),
          id: uuid(),
        },
      ],
    };
  }

  addBook = () => {
    const newBooks = [...this.state.books];
    newBooks.push({
      bookName: FAKE_BOOK_NAME,
      price: FAKE_BOOK_PRICE,
      nodeRef: createRef(null),
      id: uuid(),
    });
    this.setState({ books: newBooks });
  };

  deleteBook = (id) => {
    const newBooks = [...this.state.books].filter((book) => book.id !== id);
    this.setState({ books: newBooks });
  };
  render() {
    return (
      <>
        <h3>Books</h3>
        <TransitionGroup component="ul">
          {this.state.books.map((book) => (
            <CSSTransition
              key={book.id}
              nodeRef={book.nodeRef}
              timeout={1000}
              classNames="book"
            >
              <li ref={book.nodeRef} key={book.id}>
                <span>
                  {book.id} - {book.bookName} - {book.price}
                </span>
                <button
                  onClick={() => this.deleteBook(book.id)}
                  className="deleteButton"
                >
                  delete
                </button>
              </li>
            </CSSTransition>
          ))}
        </TransitionGroup>
        <button onClick={() => this.addBook()}>Add</button>
      </>
    );
  }
}

export default App;

```

```css
.book-enter {
  transform: translateX(100px);
  opacity: 0;
}

.book-enter-active {
  transform: translateX(0);
  opacity: 1;
  transition: all 1s ease;
}

.book-exit {
  transform: translateX(0);
  opacity: 1;
}

.book-exit-active {
  transform: translateX(-100px);
  opacity: 0;
  transition: all 1s ease;
}

.deleteButton {
  margin-left: 4px;
}
```



## 重渲染 (re-render)

重渲染前提：

1. A state change within the component (via the `useState` or `useReducer` hooks)
2. A prop change
3. A parent render (due to 1. 2. or 3.) if the component is not memoized or otherwise referentially the same

每次React中的组件重新渲染时，该组件内的代码都会被完整执行一遍，所有的变量和方法，都将被重新赋值并重新分配引用。但注意，组件函数以外的那些变量和方法，并不会被重新创建或重新分配内存

```react
import React, { useState, useEffect } from "react";

export default function ComponentWithUseEffect() {

    const [num, setNum] = useState(0)
    console.log(num) // 每点击按钮一次，控制台都会输出信息
  return (
    <div>
      <button onclick={() => {setNum(num + 1)}}>add</button>
    </div>
  );
}
```

这里说的重渲染，并不非要是这个组件返回的那些html中的内容发生了改变。任何state的变化，都可以导致重渲染，可以看到上面这个例子，UI界面并没有任何改变，一直只是一个按钮，只是点击这个按钮会出发state变化，从而导致重渲染，从而控制台打印方法被触发。

useEffect， useMemo等不会被重渲染所触发，它只对自己监听对象的变化负责。

如果一个父组件的state发生了变化，触发了重渲染，那么在DOM-Tree结构中，所有以该组件为祖先组件的组件都会被重渲染。所以说如果一个组件有很多后代，那么触发该组件的重渲染所需要的性能成本是庞大的。

```react
export default function Father() {
    const [mockState, setMockState] = useState(0)
    return (
    	<Children1></Children1>
        <Children2></Children2>
    ) // 如果Father组件被重渲染，那么所有Children组件以及Children的后代组件都会被重渲染
}
```

React当中的Context，如果其内的值发生变化，那么在Context Provider下的所有组件都会被重渲染

## Hook调用规则

React中的Hook必须在：

- 一个React组件的Root层级中被无条件调用（不能在条件分支或者二级函数中被调用）

  ```react
  export default function Search() {
      const foo = () => {
          [name, setName] = useState("") // 错误，Hook必须直接在组件Search下被调用，而不是在一个普通函数中被调用
      }
  }
  ```

- 在一个Custom Hook的Root层级中被无条件调用

## One Root Element

React only allows one root element returned.

React的组件当中，当返回JSX代码的时候，只允许有一个根组件，可以用React.Fragment来合并多个组件并返回

```react
export const foo: React.FC<Props> = ({a, b}) => {
    return (
    	<div>{a}</div>  // 报错，有多个根组件
        <div>{b}</div>
    )
}
```

```react
export const foo: React.FC<Props> = ({a, b}) => {
    return (
    	<>
    		<div>{a}</div>  // 使用React.Fragment来合并为一个根组件
        	<div>{b}</div>
        </>
    )
}
```

## React项目中的图片import

`import mealsImage from "../../assets/meals.jpg"`

## React中的props.children

children就是一个组件下面所包含的子组件

```react
<Father>
	<Children1></Children1>
    <Children2></Children2>
</Father>
	
```

那么我们如果想在Father组件中，对Children组件进行操作，在JS当中可以直接用props.children来获取到所有的子组件，但是typescript不行，因为在typescript中使用props必须要预先对props进行接口定义。

方便的方法是直接使用`PropsWithChildren` 来进行自定义类型增强，从而为自定义的props接口引入children这一属性

```react
type IProps = PropsWithChildren<{
    name: string
}>
/*
type React.PropsWithChildren<P> = P & { children?: React.ReactNode }
*/


const Father: React.FC<IProps> = ({name, children}) => {
    return (
        <>
            <div className='name'> {name} <div/>
            <div classeName= 'children'> {children} <div/>
        </>
  )
}
```

也可以自己直接把children定义在props的接口定义当中：

```typescript
type IProps = {
  text: string
  children?: ReactNode
}
```

## Portals



```html

```

```react

```

## Non-null assertion

当一个变量可能为undefined或者null的时候，在typescript中需要做额外的类型检查才能避免类型报错。

```typescript
const variableThatMayBeNull = getVar();
const result = variableThatMayBeNull ? FunctionThatNotLikeNull(variableThatMayBeNull) : null
```

但是如果你非常确定这个变量绝不可能是null或者undefined的话，你可以在这个变量赋值语句末尾加上感叹号来进行非空断言

```typescript
const variableThatMayBeNull = getVar()!;
const result = FunctionThatNotLikeNull(variableThatMayBeNull)
```

这个感叹号其实和optional标志(?)是相对应的，这个非空断言要谨慎使用，避免发生空值错误

## Props Drilling VS useContext

useContext的优点很显著，它优化了在state需要传递很多层才能到达目标组件的情况，通过Context Provider包裹一个父组件，该父组件下的所有子孙组件都能够使用context中提供的state。

但是有些时候，Props Drilling的方式也并非不可取，当你想要提升某个组件的重用性的时候，就应该使用Props Drilling而不是Context，看下面的例子：

```react
export const BackDrop: React.FC<IBackDropProps> = ({clickHandler}) => {
    return <div className="backDrop" onClick={clickHandler}></div>
}
```

对于上面这个backDrop组件，我们想在点击BackDrop的时候，做一些操作，这个操作可能是关闭当前聚焦的对话框，也有可能是其他操作，对于这种我们想要通过传不同的clickHandler来让其实现不同行为的组件，就需要避免使用useContext。因为一旦使用Context，通过Context传过来的state就定死了，直接将该组件限制到一个具体的行为，降低了它的重用性、

## Typescript: Cannot find namespace

如果一个文件要进行组件渲染（定义函数组件并渲染组件），那么这个文件的后缀名一定要是.tsx

```react
import React from "react";
import { IMealItemProps } from "../interfaces/MealsInterfaces";
import { IChildrenProps } from "../interfaces/UIInterfaces";
import {CartContext} from "./cart-context";  //如果这个文件的后缀是.ts，那么这个Context可以正常引入，但在下面使用JSX代码的时候，就会报无法找到命名空间的错误

const CartProvider = (props:IChildrenProps) => {
    const addItemToCartHandler = (item: IMealItemProps) => {}
    const removeItemFromCartHandler = (id: string) => {}
    const cartContext = {
        items: [],
        totalAmount: 0,
        addItem: addItemToCartHandler,
        removeItem: removeItemFromCartHandler
    }
    return <CartContext.Provider value={cartContext}>  //Error: Cannot find name space "CartContext"
        {props.children}
    </CartContext.Provider>
}
```

## 禁止Submit表单时页面的默认重新加载

```react
<button onclick={
        (event) => {
            event.preventDefault(); // 使用这个语句来禁止提交时的页面重加载
            buttonClickHandler();
        }
    }></button>
```

## 更新状态要避免直接修改老状态

React中在更新状态的时候要避免可变对象(Mutable Object)的产生，在更新状态的时候，不要直接对老的状态对象进行修改，而是应该直接返回一个新对象，然后让新对象成为新的状态对象

```react
...
const [itemList, setItemList] = useState([] as IItems[])
...
setItemList([...itemList, newItem]) // Correct
//-----------------
itemList.push(newItem)
setItemList(itemList) // 出错，页面并不会重渲染，因为itemList的引用并没有改变
```

数字、字符串、布尔值这些类型本身就是不可变类型，任何值的改变都会导致重新创建引用，因此可以直接在原对象的值上进行操作

```react
function App() {
  const [strList, setStrList] = useState([] as string[])
  const [testState, setTestState] = useState(0)
  let handleClick = () => {
    strList.push("new")
    setStrList(strList)
    console.log("strList current:",strList)
  }
  const latestStrList = [...strList] // 创建新引用，值会更新
  return (
    <div className="App">
      <h1>StrList:{strList}</h1> //永远不会重渲染，即便它的值已经更新。因为它始终指向同一对象
      <h1>LatestStrLists:{latestStrList}</h1> //当testState变化的时候才会更新值，因为触发重渲染会执行它的赋值语句，改变它的引用
      <h1>testState:{testState}</h1> // 会在点击按钮的时候实时更新，因为它的引用一直在变化
      <button onClick={() => {setTestState(testState + 1)}}>refresh page</button>
      <button onClick={handleClick} >append</button>
    </div>
  );
}
```

如果state是非常复杂的nested object，那么要确保被修改到的那些变量所涉及到的每一层都是被copy的，而不是直接mutate老状态，或者只是对顶层浅拷贝后就开始对底层的可变类型属性进行直接修改，这也是一种对老状态的直接mutate，因为底层那些可变类型属性仍然跟老状态中的属性指向同一引用

```typescript
function updateNestedState(state, action) {
  // Problem: this only does a shallow copy!
  let newState = { ...state }

  // ERROR: nestedState作为一个object是可变类型，在上层被浅拷贝后，它仍然指向和老状态的同一引用，所以这里的修改是完全错误的
  newState.nestedState.nestedField = action.data

  return newState
```

## React中提高性能的方法

### React.memo

只适用于函数组件（React.PureComponent 是专用于类组件的相同工具），如果一个函数组件接收相同props的情况下总是返回相同的渲染内容，那么可以用React.memo包裹该函数，则当传入的props不变时，不会重新渲染该组件。

```react
export const component: React.FC<IMockProps> = React.memo(()=>{
    return <div>What you want to hack</div>
})
```

注意，默认情况下React.memo只会对复杂的props对象做浅层对比，如果想要控制对比过程，可以往React.memo当中传入第二个参数，这个参数是一个函数：

```typescript
// React.memo(ReactComponent, CompareFunction)

function CompareFunction(prevProps: IMockProps, currentProps: IMockProps){
    if (prevProps.name === currentProps.name && prevProps.age === currentProps.age){
        return true;
    }else{
        return false;
    }
}
```

### useMemo Hook

详见[useMemo()](###useMemo())

### useCallback Hook

详见[useCallback()](###useCallback())

## Hooks

### useEffect()

当某些依赖发生改变，需要执行相应的操作（通常是副作用，比如网络请求、数据获取或者状态变化等等）时，就需要使用到useEffect

```react
useEffect(callback, dependency)
```

其中callback是回调函数。dependency是一个依赖数组，一旦这个数组中的元素发生改变，那么就将触发第一个参数callback回调函数。

dependency可以有多种写法，每种都会造成不一样的效果：

1. 直接不写或者写undefined，那么只要当前组件重渲染，就会触发callback
2. 填一个空数组 `[]` ，那么只有组件首次渲染的时候，会触发callback
3. 填入一个非空数组 `[a,b,c]` ，那么当依赖数组里面的a、b或者c发生变化的时候，就会触发callback

useEffect还可以在callback当中写一个return语句，这个语句就是useEffect的清洁工，它会在每次组件被销毁的时候执行。所谓组件被销毁，就是重渲染的时候，会销毁旧组件，创建新组件，这个时候，如果组件内还有一些计时器或者订阅之类的会持续浪费内存的操作，那么就可以在return语句中统一销毁掉。

> **每次重新渲染，都会导致原组件（包含子组件）的销毁，以及新组件（包含子组件）的诞生**。
>
> **结论**：
>
> 1、首次渲染，并不会执行useEffect中的 return
>
> 2、变量修改后，导致的重新render，会先执行 useEffect 中的 return，再执行useEffect内除了return部分代码。
>
> 3、return 内的回调，可以用来清理遗留垃圾，比如订阅或计时器 ID 等占用资源的东西。

```react
  useEffect(() => {
    if (cartCtx.items.length === 0){
      return
    }
    setBtnIsHighlighted(true)
    const timer = setTimeout(() => {
      setBtnIsHighlighted(false)
    }, 300)

    return () => {
      clearTimeout(timer)
    }
  }, [cartCtx.items])
```



### useContext()

参考链接：

[Hook API 索引 – useContext (reactjs.org)](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)

useContext可以给特定组件树下的所有组件都提供一个全局的上下文，这些组件可以用到这些上下文包含的那些属性

使用useContext：

1. 用createContext创建一个context

   ```react
   import React from 'react'
   import { ICartContext, ICartItem } from '../interfaces/CartInterfaces'
   
   export const CartContext = React.createContext({
       items: [],
       totalPrice: 0,
       addItem: (item: ICartItem) => {},
       removeItem: (id: string) => {}
   } as ICartContext) // 这里需要提供一个初始值
   ```

2. 用ContextProvider包裹要接收上下文的组件

   ```react
   import { CartContext } from "./cart-context"; // 先把创建的context引入过来
   // 通常可以把创建context和context provider的逻辑分开，因为在provider当中，我们需要定义一些逻辑和方法来对context当中的value进行修改，比如说下面的addItem和removeItem函数就是用来对context当中的items和totalPrice进行更新的
   ...
     const cartContext: ICartContext = {
       items: state.items,
       totalPrice: state.totalPrice,
       addItem: addItemToCartHandler,
       removeItem: removeItemFromCartHandler,
     };
     return (
       // 用CartContext包裹组件，那么被包裹的组件及其下属的组件树，都可以使用这个Context Provider提供的value
       <CartContext.Provider value={cartContext}> 
         {props.children}
       </CartContext.Provider>
     );
    };
   ```

3. 在被包裹的子组件中使用context中的属性

   ```react
   import { CartContext } from '../../store/cart-context'
   
   export default function HeaderCartButton(props:IHeaderButtonProps) {
     const cartCtx = useContext(CartContext);
     const numberOfCartItems = cartCtx.items.reduce((curNumber, item: ICartItem) => {return curNumber + item.amount}, 0)
     return (
       <button className={classes.button} onClick={props.showCartHandler}>
           <span className={classes.icon}>
               <CartIcon></CartIcon>
           </span>
           <span>Your Cart</span>
           <span className={classes.badge}>{numberOfCartItems}</span>
       </button>
     )
   }
   ```

### useReducer()

参考链接：

[Immutable Update Patterns | Redux](https://redux.js.org/usage/structuring-reducers/immutable-update-patterns#updating-nested-objects)

---

当一个组件中需要管理较多的state，或者说state的逻辑比较复杂的时候，可以使用useReducer来让state的更新逻辑更加清晰

使用useReducer：

1. 定义初始状态和reducer函数

   初始状态就是要管理的状态的初始值，通常在使用useReducer的时候，我们会把原本需要使用useState分开定义的state合并到一个object当中

   reducer函数是一个管理状态更新逻辑的函数，它接受两个参数：prevState和action，前者是上一个时间步（当前还没更新）的状态，后者包含要采取的状态更新操作和状态更新所用到的数据

   reducer函数返回更新后的状态：

   ```typescript
   type IState = {
       ... // your state
   }
       
   type IAction = {
       type: string,
       payload?: {item: IItem} // payload就是更新状态时所需要用到的数据，类型接口需要自己决定，是否为optional也可以自己决定
   }
   
   const reducerFunction = (prevState: IState, action: IAction) => {
       switch(action.type){
           case "AddItem":
               updatedItems = [...prevState.items, action.payload!.item] //这里请参看参考链接里关于如何在reducer函数中更新状态的说明
               return {...prevState, items: updatedItems}
           case "RemoveItem":
               ......
           default:
               return prevState
       }
   }
   ```

2. 使用useReducer钩子

   ```typescript
   const [state, dispatch] = useReducer(reducerFunction, initState);
   // state 就是现在的最新状态
   // dispatch 就是调用 reducerFuntion 的函数，其接口定义： const dispatch: (value: IAction) => void
   // 所以我们要使用dispatch，就需要往里面传一个IAction对象，也就是我们在上一个代码块中定义的那个，来决定状态更新的操作和数据
   const addItemOperation = (item: IItem) => {
       dispatch({type: "AddItem", payload: {item: item}})
   }
   // 这个时候，我们就能够调用 addItemOperation 来更新状态当中的items了
   ```

### useRef()

参考链接：

[你不知道的 useRef - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/105276393)

---

在React中，当组件被重渲染时，组件内的所有变量和方法都会被重新赋值和分配引用，如果当我们想保持住组件内一个值或者方法的引用保持不变，不随重渲染而被重新赋值和引用，那么就需要使用useRef。

useRef的作用是保持对某一个值的引用. 同样，当你想对原生的DOM中某一个元素进行直接引用，那么也可以使用useRef。

useRef所保持的元素或者值,在组件的整个生命周期内都返回同一个引用（一个可变的Ref对象）。

useRef的使用方法：

首先使用useRef来初始化一个ref，然后可以通过调用ref中的current来获取这个ref中最新的值，在整个组件的生命周期中，这个ref始终都是同一对象。

```react
import React, { useRef } from "react";

export const TestComponent: React.FC<IProps> = (props) => {
    const ref = useRef(0)
    console.log(ref.current) // 这个current根据我们的定义是一个数字。它可以容纳任何类型
    ...
}
```

如果想把ref传递给其他组件，可以使用React.forwardRef来包裹该组件，然后被包裹的这个组件就可以在props旁边再加一个ref属性，被传递ref的这个组件就可以直接使用这个ref了

useRef 可以用来直接获取原生DOM节点的引用

```tsx
import React, { useRef, useState } from "react";

export const TestComponent: React.FC<IProps> = (props) => {
    const ref = useRef<HTMLInputElement>(); // 这个ref现在就被定义成了一个HTMLInputElement对象
    const [isShow, setIsShow] = useState(false)
    return (
    	<TestInput ref={ref}></TestInput>
        <button onclick={()=>setIsShow(!isShow)}>toggle show input value</button>
        {isShow && `Input Value:${ref.current.value}`} //现在这个ref所指向的对象已经是下面代码块中的那个input节点了，因此可以用ref.current.value来实时获取它的值
    )
}
```

```react
export const TestInput = React.forwardRef((props, ref) => {
    return (
        <input ref={ref}></input>
    )
})
```

由于ref是一个始终指向同一引用的可变对象，**因此修改ref.current并不会导致页面的重渲染**

### useMemo()

useMemo是一个提高组件性能的钩子，它能够减少组件内开销很大的计算被不必要运行的次数。

```typescript
useMemo(fn: () => {}, dependency: any[])
// fn为要执行的代码块，在组件首次渲染的时候它会被执行，此后如果dependency中的变量不发生变化，fn不会被重新执行，而是返回之前缓存的结果
// denpendency 是一个数组，可以存放任何对象，一旦里面的内容发生变化，fn就会被重新执行并返回新结果
```

```typescript
const result = useMemo(() => {
	let addResult = 0;
	for (let i = state; i < 10000000000; i++){
		addResult += i
	}
	return addResult;
}, [state])
```

### useCallback()

参考链接：

[React useCallback Hook (w3schools.com)](https://www.w3schools.com/react/react_usecallback.asp)

useCallback跟useMemo类似，只不过它会返回一个函数，如果依赖数组中的对象或者变量保持不变，则这个函数不会被更新：

```typescript
const useInputWithUseCallback = () => {
  const [ value, setValue ] = useState('')
  const handleChange = useCallback(
    (e) => setValue(e.target.value),
    []
  )
  return [ value, handleChange ]
} // 使用useCallback实现自定义钩子
```

useCallback 经常与 React.memo连用

因为在React当中，每当组件被重渲染，组件中的函数都会被重新赋值和分配引用，因此新函数的引用跟被销毁组件中的函数根本不指向同一内存地址，两个对象只要引用不一样，那么默认情况下React会直接判定二者不是同一对象，从而导致React.memo判定达不到程序员想要的效果。

这时候需要把要传给子组件的函数用useCallback所包裹起来，这时候如果依赖不变，那么这个函数将始终指向同一引用，这时候被React.memo包裹的子组件就能够接收到相同的函数对象了。

### Custom Hooks

自定义Hook的最终目的：共享相同的代码逻辑，减少代码冗余。

自定义Hook就是一个函数，函数名以use开头，用use开头能让React将这个函数识别为一个hook，从而让其遵循[Hook规则](https://zh-hans.reactjs.org/docs/hooks-rules.html)

程序员可以自行定义Custom Hook函数的参数和返回值

```react
export function useBeer(beerType: string) {
    {data, loading, error} = useQuery(BEER_QUERY[beerType])
    [price, setPrice] = useState(0)
    if (data){
        setPrice(data.beerInfo.price)
    }
    const setCrazySalePrice = (factor:number, bias:number) => {
        this.setPrice(price * factor + bias)
    }
    return [price, setCrazySalePrice, loading]
}
```

这个自定义Hook接收一个啤酒类别的参数，然后获取啤酒的实时价格，然后返回啤酒的价格state以及一个可以改变价格的函数和loading状态

