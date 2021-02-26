

# React Hook

## 概览

* `useState`
* `useEffect`
* `useContext`
* `useReducer`
* `useRef`
* `useImperativeHandle`
* `useCallback`&`useMemo`
* `useLayoutEffect`

## 为什么要用 Hook

1. 代码复用变的更简单
2. 不用记很多生命周期方法
3. 更干净的代码
4. 学习成本低

## useState

### 初识

```react
const [count, setCount] = useState(0)
```
* `调用 useState` 方法做了什么？

    定义一个“state变量”。

* `useState` 需要什么参数？

    其接收一个参数，作为变量初始化的值。

* `useState` 方法的返回值是什么？

    返回当前 state 以及更新 state 的函数。

### 使用useState

> React 会确保 setState 函数的标识是稳定的，并且不会在组件重新渲染时发生变化。这就是为什么可以安全地从 useEffect 或 useCallback 的依赖列表中省略 setState。

首先我们先通过 `useState` 方法定义三个变量（包含基本类型和引用类型的数据）分别为：`count`、`studentInfo`、`subjectList`，然后对它们的值进行修改。

```react
const [count, setCount] = useState(0)

const [studentInfo, setStudentInfo] = useState({name: '小文', age: 18, gender: '女'})

const [subjectList, setSubjectList] = useState([
  { id: 0, project_name: '语文' },
  { id: 1, project_name: '数学' }
])
```
* 修改 count 值为1

```react
setCount(1)
```
* 修改 studentInfo 对象的 age 属性，值为 20；并添加 weight 属性，值为 90

```react
setStudentInfo({
  ...studentInfo,
  age: 20,
  weight: 90
})
```
* 修改 subjectList 数组的第二项的 project_name 属性，值为体育；并添加第三项 `{ id: 2, project_name: '音乐' }`

```react
// 深拷贝（immutable.js、immer.js、loadsh）
let temp_subjectList = JSON.parse(JSON.stringify(subjectList))
temp_subjectList[1].project_name = '体育'
setSubjectList(temp_subjectList)
```
### 函数式更新

`setState` 接收一个函数。

如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 setState。该函数将接收先前的 state，并返回一个更新后的值。

使用上面定义的变量：

* 点击按钮累加 count 

```react
<button onClick={() => setCount(prevCount => ++prevCount)}>+ 累加</button>
```
* 修改 studentInfo 对象的 age 属性，值为 20

```react
setStudentInfo(prevState => {
  return {...prevState, age: 20}
})
```
### 惰性初始 state
如果初始 state 需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的 state，此函数只在初始渲染时被调用：

```react
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props)
  return initialState
})
```
### 实际应用

我们在实际应用中，经常会遇到一些结构比较复杂的数据，如果每个地方都使用 useState 去定义他们，会是一件比较痛苦的事情。

分享一个插件： `use-immer`。

1. 安装

```bash
npm install immer use-immer
```
2. 引用

```react
import { useImmer } from 'use-immer'
```

3. 重新声明上面用到的 `subjectList` 

```react
const [subjectList, setSubjectList] = useImmer([
  { id: 0, project_name: '语文' },
  { id: 1, project_name: '数学' }
])
```
4. 修改 subjectList 数组的第二项的 project_name 属性，值为体育；并添加第三项 `{ id: 2, project_name: '音乐' }`

```react
setSubjectList(draft => {
  draft[1].project_name = '体育'
  draft[2] = { id: 2, project_name: '音乐' }
})
```
需要注意的是，这里的 `setSubjectList` 方法接收的是一个函数，该函数接收一个参数 `draft`，可以理解为是变量 `subjectList` 的副本。更详细的可以深入了解一下`immutable`、`immer`、`use-immer`。

## useEffect

`useEffect` 会在浏览器绘制后延迟执行，但会保证在任何新的渲染前执行。React 将在组件更新前刷新上一轮渲染的 effect。


### 每一次渲染

**重点**：关于每一次渲染（rendering），组件都会拥有自己的：

1. Props and State
2. 事件处理函数
3. Effects

#### Props and State

写一个计数器组件 `Counter`：

```react
function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  )
}
```
`Counter`组件第一次渲染的时候，从 `useState()` 拿到 `count` 的初始值为 0。当我们调用 `setCount(1)`时，React 会再次渲染该组件，此时 `count` 的值为 1，以此类推，每一次渲染都是独立的。

```react
// 第一次
function Counter() {
  const count = 0
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// 第二次
function Counter() {
  const count = 1
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// 第三次
function Counter() {
  const count = 2
  // ...
  <p>You clicked {count} times</p>
  // ...
}
```
**每一次渲染都能拿到独立的`count` 状态，这个状态值是函数中的一个常量**，这个常量由 React 提供。当调用 `setCount` 的时候，React 会带着一个不同的 `count` 值再次调用组件。然后，React会更新DOM以保持和渲染输出一致。

**最关键** 的就是：任意一次渲染中的 `count` 常量都不会随着时间改变。渲染输出会变是因为 `Counter` 组件被调用，而在每一次调用引起的渲染中，它包含的 `count` 常量都是独立的。即，props 和 state 都是独立的。

#### 事件处理函数

修改一下计数器组件 `Counter` 的例子。

组件内容：有两个按钮，一个按钮用来修改 `count` 的值，另一个按钮在 3s 延迟后展示弹窗。

```react
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count)
    }, 3000)
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  )
}
```
* 点击按钮修改 `count` 的值为 3。
* 点击另一个按钮打开弹窗。
* 在弹窗弹出前，点击按钮修改 `count` 的值为 5。

此时，弹窗中的展示的 `count` 值为 3。

分析：

首先整个过程进行了 6 次渲染。

1. 初始化渲染：render0；
2. 修改 `count` 值为 3，进行 3 次渲染：render1 -> render2 -> render3；
3. 点击按钮打开弹窗，此时组件是 “render3 状态”；
4. 修改 `count` 值为 5，进行 2 次渲染：render3 -> render4 -> render5;

```react
// 组件状态：render0 -> render1 -> render2 -> render3
function Counter() {
  const count = 3
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count)
    }, 3000)
  }
  // ...
}

// 组件状态： render3
// 触发事件处理函数 handleAlertClick，此时该函数捕获 count 值为 3，并将在 3 秒后打开弹窗。
function Counter() {
  const count = 3
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 3)
    }, 3000)
  }
  // ...
}

// 组件状态：render3 -> render5 -> render5
function Counter() {
  const count = 5
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count)
    }, 3000)
  }
  // ...
}
```
#### Effects

修改 `Counter` 组件，点击 3 次按钮：
```react
function Counter() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`)
    }, 3000)
  })

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  )
}
```
分析：
整个过程组件进行了四次渲染：

1. 初始化，render0：打印 `You clicked 0 times`；
2. 修改 `count` 值为1，render1：打印 `You clicked 1 times`；
3. 修改 `count` 值为2，render2：打印 `You clicked 2 times`；
4. 修改 `count` 值为3，render3：打印 `You clicked 3 times`；

通过整个例子我们可以知道，在每次渲染中，Effects 也是独立的。

> 并不是 count的值在“不变”的 effect 中发生了改变，而是 effect 函数本身在每一次渲染中都不相同。

**思考：**

这对我们的实际应用有什么影响呢？

### 清除 effect

当我们在 `useEffect` 中使用了定时器或者添加了某些订阅，可以通过 `useEffect` 返回一个函数，进行清除定时器或者取消订阅等操作。但我们需要知道的是，清除是 “滞后” 的。

看一下例子，在 `useEffect` 中打印点击的次数：

```react
function Example() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    console.log(`You clicked ${count} times`)
    return() => {
      console.log('销毁')
    }
  })

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  )
}
```
点击按钮 3 次，控制台中打印的结果如下：

1. `You clicked 0 times`
2. 销毁
3. `You clicked 1 times`
4. 销毁
4. `You clicked 2 times`
5. 销毁
5. `You clicked 3 times`

从打印结果我们可以很容易看出，上一次的 effect 是在重新渲染时被清除的。

补充：那么组件的整个重新渲染的过程是怎么样的呢？

假设现在有 render0 和 render1 两次渲染：

1. React 渲染 render1 的UI;
2. 浏览器绘制，并呈现 render1 的UI；
3. React 清除 render0 的 effect；
4. React 运行 render1 的 effect；

> React 只会在浏览器绘制后运行 effects。这使得你的应用更流畅因为大多数effects并不会阻塞屏幕的更新。

通过下面这个例子，来印证一下这个结论吧~

```react
function Example() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    setCount(99)
    console.log(count)
    return() => {
      console.log('销毁')
    }
  })

  console.log('React Hook！')

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  )
}
```
运行代码，在控制台我们可以看到以下输出（此时组件状态为初始化：render0）：

1. React Hook！
2. 0
3. React Hook！
4. 销毁
5. 99
6. React Hook！

根据打印结果，我们可以分析出：

1. React 渲染初始化的UI（render0）；
2. 执行 `useEffect`；
3. 调用 `setCount` 方法修改 `count` 值为 99，组件重新渲染（render1）；
4. 根据执行顺序，继续执行 render0 状态下的 `useEffect`，打印 `count` 的值为 0。（render0 下 `count` 值为0）
5. React 重新渲染UI（render1）；
6. 执行 render0 的 `useEffect` 的清除函数；
7. 执行 render1 的 `useEffect`；
8. 调用 `setCount` 方法修改 `count` 值为 99（由于传入的值没有改变，所以组件没有重新渲染）；
9. 打印 `count` 的值为 99；

其中 `React Hook！`，这个打印我理解的是：组件被调用了，React 判断是否需要渲染。

### 设置依赖

实际应用中，我们不需要在每次组件更新时，都去执行某些 effects，这个时候我们可以给 `useEffect` 设置依赖，告诉 React 什么时候去执行 `useEffect`。

看下面这个例子，只有在 `name` 发生改变时，才会执行这个 `useEffect`。如果将依赖设置为空数组，那么这个 `useEffect` 只会执行一次。
```react
useEffect(() => {
  document.title = 'Hello, ' + name
}, [name])
```

#### 正确地设置依赖

引出问题：首先需求很简单，通过定时器，每过一秒就将 `count` 的值累加 1。

```react
const [count, setCount] = useState(0)

useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1)
}, 1000)
  return () => clearInterval(id)
}, [])
```

我们只希望设置一次 `setInterval` 定时器，所以将依赖设置为了 `[]`，但是由于组件每次渲染拥有独立的 state 和 effects，所以上面代码中的 `count` 值，一直是 0，当一次执行完 `setCount`后，后续的`setCount`操作都是无效的。

解决方案：使用函数式更新。

```react
const [count, setCount] = useState(0)

useEffect(() => {
  const id = setInterval(() => {
    setCount(preCount => preCount + 1)
	}, 1000)
  return () => clearInterval(id)
}, [])
```
但是在实际应用中，这种方式还远远不能满足我们的需求。比如在依赖多个数据的时候：

```react
function Counter() {
  const [count, setCount] = useState(0)
  const [step, setStep] = useState(1)

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step)
    }, 1000);
    return () => clearInterval(id)
  }, [step])

  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => setStep(Number(e.target.value))} />
    </>
  )
}
```
当我们在修改 `step` 变量时，会重新设置定时器。这是我们不愿意看到的，那应该怎么去优化呢？这个时候我们就需要用到`useReducer`了。

> 当我们想更新一个状态，并且这个状态更新依赖于另一个状态的值时，我们可能需要使用`useReducer`去替换它们。

#### 关于函数

在 `useEffect` 中调用了定义在外部的函数时，我们可能会遗漏依赖。所以我们可以将函数的定义放到`useEffect`中。

但是当我们有一些可复用的函数定义在外部，此时应该怎么处理呢？

1. 如果这个函数没有使用组件内的任何值，我们可以将它放到组件外部定义。

```react
function getFetchUrl(query) {
  return 'xxx?query=' + query
}

function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react')
  }, [])

  useEffect(() => {
    const url = getFetchUrl('redux')
  }, [])
}
```
2. 使用 `useCallback` 包装。
```react
function SearchResults() {
  const [query, setQuery] = useState('react')

  const getFetchUrl = useCallback(() => {
    return 'xxx?query=' + query
  }, [query])

  useEffect(() => {
    const url = getFetchUrl()
  }, [getFetchUrl])
}
```

如果 `query` 保持不变，`getFetchUrl`也会保持不变，我们的 effect 也不会重新运行。但是如果 `query` 修改了，`getFetchUrl` 也会随之改变，因此会重新请求数据。

> useCallback本质上是添加了一层依赖检查。它以另一种方式解决了问题 - 我们使函数本身只在需要的时候才改变，而不是去掉对函数的依赖。

模拟一个实际应用场景：列表的分页、增删改查

## useContext

useContext的使用场景。

> 在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但这种做法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。

直接看一个例子吧~

1. 创建顶层组件 `Container`。

```react
import React, { useState, createContext } from 'react'
import Child1 from './Child1'
import Child2 from './Child2'

// 创建一个 Context 对象
export const ContainerContext = createContext({})

function Container() {
  const [state, setState] = useState({child1Color: 'pink', child2Color: 'skyblue'})
  const changeChild1Color = () => {
    setState({
      ...state,
      child1Color: 'lightgreen'
    })
  }

  return (
    <>
      <ContainerContext.Provider value={state}>
        <Child1></Child1>
        <Child2></Child2>
      </ContainerContext.Provider>
      <button onClick={changeChild1Color}>修改child1颜色</button>
    </>
  )
}

export default Container
```

2. 创建子组件`Child01`。

```react
import React, {useContext} from 'react'
import { ContainerContext } from './Container'

function Child01() {
  const value = useContext(ContainerContext)
  
  return <h1 style={{color: value.child1Color}}>我是Child01组件</h1>
}

export default Child01
```

3. 创建子组件`Child02`。

```react
import React, {useContext} from 'react'
import { ContainerContext } from './Container'

function Child02() {
  const value = useContext(ContainerContext)
  
  return <h1 style={{color: value.child2Color}}>我是Child02组件</h1>
}

export default Child02
```

我们可以通过这个简单的 demo 来了解一下 `useContext` 相关的基础知识。

基本使用方法分析：

1. 通过 React 提供的 `createContext` 方法创建一个 `Context` 对象。

   通过`export const ContainerContext = createContext({})`创建了一个 `Context` 对象，并设置了默认值为 `{}`。

   **注意**：默认值只有在组件所处的树中没有匹配到 Provider 时，默认值才会生效。

   稍微修改一下 `Container` 中返回的组件，这个时候 `Child1` 和 `Child2` 读取的 context 就是创建 `Context` 对象时的默认值了。

   ```
   <>
     <Child1></Child1>
     <Child2></Child2>
     <button onClick={changeChild1Color}>修改child1颜色</button>
   </>
   ```

2. 每个 `Context` 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

   `Container` 组件中的 `Context` 对象返回 `ContainerContext.Provider` 组件，它接收一个value 属性，传递给消费组件。同时包裹在其内部的消费组件（`Child1`和`Child2`）可以订阅 context 的变化。

   一个 Provider React 组件可以和多个消费组件有对应关系。多个 Provider React 组件 也可以嵌套使用，内层的会覆盖外层的数据。

3. 在消费组件中，使用 `useContext` 订阅 context。

   **注意**：`useContext` 的参数必须是 context 对象本身。

值得注意的是：

1. 订阅了 context 的组件，总会在 context 值变化时重新渲染。如果重渲染组件的开销较大，可以通过使用 [memoization](https://github.com/facebook/react/issues/15156) 来优化。
2. 使用 Context 一定程度上会使组件的复用性降低，我们需要合理的取舍。

## useReducer

`useReducer` 的用法和 `Redux` 很像，可以用来创建一个局部的状态管理。

```react
const [state, dispatch] = useReducer(reducer, initialArg, init)
```

`useReducer` 是 `useState` 的替代方案，它接收一个形如 `(state, action) => newState` 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。

`useReducer` 的第三个参数可以省略，当我们需要惰性初始化的时候可以设置第三个参数。

### 使用场景

1. `state` 结构复杂，数据之间关联性强，一个操作需要修改多个 `state`；
2. 在深层子组件里面去修改父组件的状态；

我们先来看第一个场景。

### 复杂state的使用

`Counter` 组件中 `useReducer` 接收了两个参数：`countReducer` 和 `initialState`（其中 `initialState` 的值包含 `count`、`times`、`otherState`； `countReducer` 是一个纯函数，用来计算并返回最新的 state）。

`useReducer` 返回初始化的 `state` 和 `dispatch` 方法。点击按钮，通过 `dispatch` 发起一个 `action`，执行 `countReducer` 方法。

```react
import React, {useReducer} from 'react'

const initialState = {
  count: 0,
  times: 0,
  other: ''
}

function countReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {...state, count: state.count + 1, times: state.times + 1}
    case 'decrement':
      return {...state, count: state.count - 1, times: state.times + 1}
    default:
      return state
  }
}

function Counter() {
  const [state, dispatch] = useReducer(countReducer, initialState)

  return (
    <>
      Times: {state.times}
      <br/>
      Count: {state.count}
      <br/>
      <button onClick={() => dispatch({type: 'decrement'})}>减</button>
      <button onClick={() => dispatch({type: 'increment'})}>加</button>
    </>
  )
}

export default Counter

```

需要注意的是：React 在比较 `oldState` 和 `newState` 的时候是使用 `Object.is` 函数，如果是同一个对象则组件不会重新渲染。所以我们在使用 reducer 时不能直接修改参数中 `state` 对象，而是需要创建一个新的对象 `newState`，在 `newState` 上修改数据并返回。

例子中我们是通过 ES6 的解构赋值创建的新的对象，但是实际应用中我们的数据结构可能会更为复杂，state 可能是多层嵌套的，对于这种复杂的 state 的应用场景，我们可以通过 [immer](https://github.com/immerjs/immer) 等库来解决。

```react
import produce from 'immer'
// ...
const countReducer = produce((draft, action) => {
  switch (action.type) {
    case 'increment':
      draft.count++
      draft.times++
      break
    case 'decrement':
      draft.count--
      draft.times++
      break
    default:
      return draft
  }
})
// ...
```

### 在深层子组件里的使用

当我们在深层组件中，需要修改父组件中的 state 状态时，就可以通过 `useContext` + `useReducer` 的方式来实现。避免了逐层传递 props。

使用方法如下。

新建 `Parent` 组件：

```react
import React, {useReducer, createContext} from 'react'
import produce from 'immer'
import Child from './Child'

export const ParentContext = createContext({})

const initState = {
  form: {
    name: '小红'
  }
}
const reducer = produce((draft, action) => {
  switch (action.type) {
    case 'updateName':
      draft.form.name = action.payload.name
      break
    default:
      return draft
  }
})

function Parent() {
  const [state, dispatch] = useReducer(reducer, initState)

  return (
    <>
      <ParentContext.Provider value={dispatch}>
        姓名：{state.form.name}
        <Child></Child>
      </ParentContext.Provider>
    </>
  )
}

export default Parent
```

新建子组件 `Child`：

```react
import React, {useContext, memo} from 'react'
import {ParentContext} from './Parent'

function Child() {
  const dispatch = useContext(ParentContext)

  return (
    <div>
      <Button onClick={() => dispatch({type: 'updateName', payload: {name: '测试'}})}>
        修改姓名
      </Button>
    </div>
  )
}

export default memo(Child)
```

## useRef

```react
const refContainer = useRef(initialValue)
```

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

如果你将 `ref` 对象以 `<div ref={myRef} />` 形式传入组件，则无论该节点如何改变，React 都会将 `ref` 对象的 .current 属性设置为相应的 DOM 节点。

而且 `useRef` 可以很方便地保存任何可变值，且它会在每次渲染时返回同一个 `ref` 对象。 

**应用1：**

```react
function SelfUseRef() {
  const inputEl = useRef(null)

  const focusInput = () => {
    // `current`指向 已挂载到 Input 元素
    inputEl.current.focus()
  }

  return (
    <div>
      <Input ref={inputEl} placeholder="请输入内容" />
      <Button onClick={focusInput}>Focus the input</Button>
    </div>
  )
}

```

**应用2：**

分析以下代码：

```react
function Example ({ fn, propA, propB, propC }) {
  useEffect(() => {
    fn(propA, propB, propC)
  }, [])

  return (
    <div></div>
  )
}
```

`react`会警告我们，`useEffect`中缺少依赖。

解决方案有两种：

1. 取消校验；

```react
useEffect(() => {
    const { fn, propA, propB, propC } = initProps.current
    fn(propA, propB, propC)
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [])
```

1. 通过`useRef`包裹参数；

```react
function Example ({ fn, propA, propB, propC }) {
  const initProps = useRef({
    fn,
    propA,
    propB,
    propC,
  })

  useEffect(() => {
    const { fn, propA, propB, propC } = initProps.current
    fn(propA, propB, propC)
  }, [])

  return (
    <div></div>
  )
}
```

## `useImperativeHandle`

```react
useImperativeHandle(ref, createHandle, [deps])
```

第一个参数，接收一个 `ref` 对象；

第二个参数是一个回调函数，函数返回一个对象，对象里面存储需要暴露给父组件的属性或方法；

例子：

新建弹窗组件 `Popup`：

```react
import React, {useRef} from 'react'
import Form from './Form'

function Popup() {
  const formRef = useRef(null)
  const submitForm = () => {
    console.log(formRef)
    formRef.current.submit()
  }

  return (
    <div>
      <Form ref={formRef}></Form>
      <button onClick={submitForm}>确定</button>
    </div>
  )
}

export default Popup
```

新建表单组件 `Form`：

```react
import React, {memo, useImperativeHandle, forwardRef} from 'react'

function Form (props, ref) {
  useImperativeHandle(ref, () => ({
    submit: () => {
      console.log('提交表单')
    },
    params: { a: 1 }
  }))

  return (
    <div>
      <input type="temp_text" placeholder="请输入姓名"/>
      <input type="temp_password" placeholder="请输入密码"/>
    </div>
  )
}

export default memo(forwardRef(Form))
```

分析：

在父组件 `Popup` 中，使用 `useRef` 创建一个 `ref` 对象，并将其传递给子组件 `Form`。

在子组件 `Form` 中 ，`forwardRef` 接收组件（`Form`）作为参数，并创建一个React组件。这个组件能够将其接收的 `ref` 属性转发到其组件树下的另一个组件中。再将传入的 `ref` 通过 `useImperativeHandle` 进行绑定，指定该子组件对外暴露的方法或属性（`submit` 和 `params`）。

此时，在父组件中就可以调用子组件的方法了。

## useCallback 和 useMemo

### useCallback

```react
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  }, [a, b])
```

把内联回调函数及依赖项数组作为参数传入 `useCallback`，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染的子组件时，它将非常有用。

新建 `Parent` 组件：

```react
import React, {useState, useEffect} from 'react'
import Child from './Child'

function Parent() {
  const [parentTxt, setParentTXT] = useState('parent 初始化')
  const [childTxt, setChildTxt] = useState('child 初始化')

  useEffect(() => {
    console.log('Parent')
  })

  const updateParentTxt = () => {
    setParentTXT('parentText-' + new Date().getTime())
  }

  const updateChildTxt = () => {
    setChildTxt('childTxt-' + new Date().getTime())
  }

  return (
    <div>
      <p>Parent组件：{parentTxt}</p>
      <button onClick={updateParentTxt}>修改 parentTxt</button>
      <br/>
      <Child childTxt={childTxt} updateChildTxt={updateChildTxt}></Child>
    </div>
  )
}

export default Parent
```

新建 `Child` 组件：

```react
import React, {useEffect, memo} from 'react'

function Child(props) {
  const { childTxt, updateChildTxt } = props

  useEffect(() => {
    console.log('Child')
  })

  return (
    <div>
      <p>Child组件：{childTxt}</p>
      <button onClick={updateChildTxt}>修改 childTxt</button>
    </div>
  )
}

export default memo(Child)
```

Tips：打印顺序、关于 `shouldComponentUpdate`、`React.PureComponent`和`React.memo`。

当点击按钮 “修改 parentTxt” 的时候，控制台会再次输出一遍 “Child” 和 “Parent”，父组件 `Parent` 重新渲染，而子组件 `Child` 也进行了重新渲染。

原因是当 `Parent` 组件向 `Child` 组件传递 `updateChildTxt` 方法时，`Child` 组件接收的是这个方法的引用。当 `Parent` 组件重新渲染后，产生了另一个独立的 `updateChildTxt`（每一次渲染都有其独立的 state、 props、事件处理函数和effects），而 `Child` 组件接收的引用地址发生了改变，所以 React 认为 `Child` 组件也需要重新渲染。

使用 `useCallback` 包裹 `updateChildTxt` 方法：

```react
import React, {useState, useEffect, useCallback} from 'react'
...
const updateChildTxt = useCallback(() => {
  setChildTxt('childTxt-' + new Date().getTime())
}, [])
...
```

## useLayoutEffect

其函数签名与 `useEffect` 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，`useLayoutEffect` 内部的更新计划将被同步刷新。

例子：

```react
import React, { useState, useEffect } from 'react'
import { Button } from 'antd'

function SelfUseLayoutEffect() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    if (count === 0) {
      setCount(new Date().getTime())
    }
  }, [count])

  return (
    <div>
      <div>time: {count}</div>
      <div className="mr-t10"></div>
      <Button onClick={() => setCount(0)}>修改 count</Button>
    </div>
  )
}

export default SelfUseLayoutEffect
```

它们的主要区别在于 `useLayoutEffect` 会阻塞渲染，而 `useEffect` 不会。