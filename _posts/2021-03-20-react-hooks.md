---
title: react-hooks
author: jyy
date: 2021-03-20 11:10:00 +0800
# categories: [Blogging, Tutorial]
tags: [react]
pin: true
---

## 解决了什么问题

- 函数组件内部没状态的问题
- 
## 注意事项

- 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
- 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用
    - 每个hook的调用都会对应一个全局的index索引，通过这个索引去当前组件上的hookState数组里对应的索引找数据。每次渲染完成后都会将index清零。

```js
// 当前正在运行的组件
let currentComponent
 
// 当前 hook 的全局索引
let currentIndex
 
// 第一次调用 currentIndex 为 0
if (Math.random() > 0.5) {
  useState('first')
}
 
// 第二次调用 currentIndex 为 1
useState('second')
//第一次渲染的时候随机值为0.6，那第一个hook会执行，对应的下标是0.第二次渲染的时候随机值为0.1，那么第二个hook会执行，对应的下标是0他会去hookState数组里下标是0的索引里找数据，这样就错乱了，他本应该找的下标是1
```
## useState

- 每次的setState也是异步的，和类组件的原理类似。
- 每次渲染都会得到一个新的useState，都是一个独立的闭包。每个useState函数都有那次特定的state.
## memo

- 是react的一个性能优化，主要是解决父组件给子组件传递属性时，即使属性值没有变，子组件也会重新渲染的问题。他是一个高阶组件，内部其实就是在shouldComponentUpdate这个钩子里做了一层浅比较。
- 注意浅比较带来的问题：
    - 对于引用类型的属性，只会对最外层做比较，如果对象里面嵌套的属性值有改变的话则是监测不到改变的，这样会导致子组件不会重新渲染。
    - 尽量保持数值类型的属性传递。

```js
class PureComponent extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    if(nextProps == this.props){
      return false;
    }
    if(Object.keys(this.props).length !== Object.keys(nextProps).length){
      return true;
    }
    for(let key in this.props){
      if(this.props[key] !== nextProps[key]){
        return true
      }
    }
    return false
  }
}
function memo(OldComponent) {
  return class extends PureComponent {
    render() {
      return (
        <OldComponent {...this.props} />
      )
    }
  }
}
```

## useCallback

- 是为了解决给子组件传递函数时，重新执行父组件会将传递给子组件一个新的函数，但是其实这个函数并没有做任何改变。只是引用地址改变了。导致子组件重复渲染。
- 只有当依赖的值发生改变时，才会重新创建一个新函数。

```js
let hookStates = [] //存放组件所有的hooks的数据
let hookIndex = 0
function useCallback(callback, deps) {
  if (hookStates[hookIndex]) {
    let [lastCallback, lastCallbackDeps] = hookStates[hookIndex]
    let same = deps.every((item, index) => item === lastCallbackDeps[index])
    if (same) {
      return lastCallback
    } else {
      hookStates[hookIndex++] = [callback, deps]
      return callback
    }
  } else {
    hookStates[hookIndex++] = [callback, deps]
    return callback
  }
}
```

## useMemo
- 是为了解决给子组件传递对象时，重新执行父组件会将传递给子组件一个新的对象，但是其实这个对象并没有做任何改变。只是引用地址改变了。导致子组件重复渲染。
- 只有当依赖的值发生改变时，才会重新创建一个新对象。

```js

function useMemo(factory, deps) {
  if (hookStates[hookIndex]) {
    let [lastMemo, lastMemoDeps] = hookStates[hookIndex]
    let same = deps.every((item, index) => item === lastMemoDeps[index])
    if (same) {
      return lastMemo
    } else {
      let memo = factory()
      hookStates[hookIndex++] = [memo, deps]
      return memo
    }
  } else {
    let memo = factory()
    hookStates[hookIndex++] = [memo, deps]
    return memo
  }
}
```

## useEffect

- 给函数组件增加了操作副作用的能力，传给useEffect的第一个参数会在组件渲染之后执行，如果这个函数有返回值，那这个函数函数会在下次执行useEffect先执行，目的是要清除副作用。它跟 class 组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途。
- 场景： 改变 DOM、添加订阅、设置定时器、记录日志等等。
- 同样每次使用useEffect也是一个独立的闭包。第二个参数也是依赖项。只有在依赖项的值有变化时或者是空数组才会再次执行useEffect。

- 想在effect的回调函数里读取最新的值而不是捕获的值。最简单的实现方法是使用refs
```js
function Parent() {
  const [query, setQuery] = useState("react");

  // ✅ Preserves identity until query changes
  const fetchData = useCallback(() => {
    const url = "https://hn.algolia.com/api/v1/search?query=" + query;
    // ... Fetch data and return it ...
  }, [query]); // ✅ Callback deps are OK

  return <Child fetchData={fetchData} />;
}

function Child({ fetchData }) {
  let [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ Effect deps are OK

  // ...
}

```
## useRef
- 每次都返回同一个对象和React.createRef()不同

## useLayoutEffect

- useEffect会在浏览器渲染结束后执行,是异步执行的；useLayoutEffect 则是在 DOM 更新完成后,浏览器绘制之前执行,会阻塞浏览器的渲染，是同步执行
- 会影响到渲染的操作尽量放到 useLayoutEffect中去，避免出现闪烁问题
