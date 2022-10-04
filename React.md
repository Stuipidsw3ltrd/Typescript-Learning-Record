# React

## 重渲染(re-render)

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

- 一个React组件的Root层级直接被调用

  ```react
  export default function Search() {
      const foo = () => {
          [name, setName] = useState("") // 错误，Hook必须直接在组件Search下被调用，而不是在一个普通函数中被调用
      }
  }
  ```

- 在一个Custom Hook中被调用

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
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

```react
  // React does *not* create a new div. It renders the children into `domNode`.
  // `domNode` is any valid DOM node, regardless of its location in the DOM.
export default Father(props:IProps){
  const containerNode = document.getElementByID('modal-root')!
  return (
    <div>
      ReactDOM.createPortal(props.children, containerNode)
    </div>
  );
}
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

## 骚操作:快速转化String为Number

```typescript
// normal way
let x = parseInt(str)
// terrific way
let y = +str // 对负数也有效，并不会把负数变成正数。str必须是能转化为数字的字符串,不然报错
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

```typescript
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
export const ChildrenComponent = React.forwardRef((props, ref) => {
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
