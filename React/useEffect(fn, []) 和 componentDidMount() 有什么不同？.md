自从使用 Hooks 后，可以很方便的在函数组件中执行有副作用的操作了。  
`useEffect(fn, [])` 在执行时机上与类组件生命周期函数 `componentDidMount()` 类似，那么它们一个是生命周期函数，一个是 Hook，它们在执行时机上具体有什么不同呢？

这个问题可以从两方面解答：
1. useEffect 的第二个参数 [] 如何影响 fn 的执行时机？
2. fn 和 componentDidMount 的执行时机


#### 1. useEffect 的第二个参数 [] 如何影响 fn 的执行时机？
- useEffect(fn, []) 和 componentDidMount 在插入 DOM 节点元素时执行（标记了 Placement）。
- useEffect(fn) 在 「创建｜更新」 元素时执行，即每次 render 都会执行。
- useEffect(fn, [dep]) 在 「创建元素｜dep 发生变化」 时执行。

#### 2. fn 和 componentDidMount 的执行时机
- `useEffect` 在 commit 阶段完成后，被异步调用。
- `componentDidMount` 在 commit 阶段完成视图更新（mutation 阶段）后，在 layout 子阶段被同步调用。useLayoutEffect 也是在此子阶段同步调用。

**commit 阶段的三个子阶段：**

1. 渲染前：beforeMutation 阶段
2. 渲染中：mutation 阶段
3. 渲染后：layout 阶段    -->  componentDidMount useLayoutEffect 同步调用

commit 阶段完成后，异步调用 useEffect 的 fn()。  


**commit 阶段如何渲染数据变化:**

render 阶段会计算出每个节点的数据变化，节点间的数据变化以 Effect 链表的形式，传递到 commit 阶段。

## useEffect() 清除函数的作用及清除时机

清除时机： 

- 组件销毁
- 每次组件重新渲染时，先清除上一个 effect 副作用。


因为 React 会在 浏览器绘制后 执行 effect，执行过程：
- React 渲染（commit） { id: 2 } 的 UI
- 浏览器渲染 { id: 2 } 的 UI
- React 清除 { id: 1 } 的 effect
- React 执行 { id: 2 } 的 effect


作用： 
- 取消订阅，包括事件绑定、定时器 timer。
- 取消请求。
