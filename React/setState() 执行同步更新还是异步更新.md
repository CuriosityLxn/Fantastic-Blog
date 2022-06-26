## 结论前置

- Legacy 模式：**采用同步的递归更新**机制。
  - React 生命周期函数和合成事件，会命中 `batchedUpdates()`，`setState()` 异步执行更新
  - JS 原生事件和异步代码，不会命中 `batchedUpdates()`，`setState()` 同步执行更新
- Concurrent 模式：**采用异步的可中断更新**机制，`setState()` 遵循异步的执行更新。

## setState 的官方说明

**官方文档中对于 `setState` 的说明：**

> Think of `setState()` as a request rather than an immediate command to update the component. For better perceived performance, React may delay it, and then update several components in a single pass. React does not guarantee that the state changes are applied immediately.
>
> `setState()` does not always immediately update the component. It may batch or defer the update until later. This makes reading this.state right after calling `setState()` a potential pitfall. Instead, use `componentDidUpdate` or a `setState` callback (`setState(updater, callback)`), either of which are guaranteed to fire after the update has been applied. If you need to set the state based on the previous state, read about the updater argument below.

大意是 React 会有延迟的在一个传递中更新多个 `state`，在 `setState()` 后有可能不能立马取到更新后的 `state` ，想要保证能取到，要在 `componentDidUpdate` 或者 `setState` 回调函数中去取。   
这种现象是跟 React 的执行机制有关。


## React 不同模式采用不同的更新机制

React 提供了渐进迁移策略，来防止高破坏性的升级，于是产生了三种开发模式：

- Legacy 模式：`ReactDom.render(<App/>, rootNode)`，**采用同步的递归更新**机制。
- Blocking 模式：`ReactDom.createBlockingRoot(rootNode).render(<App/>)`
- Concurrent 模式：`ReactDom.createRoot(rootNode).render(<App/>)`，**采用异步的可中断更新**机制，将作为未来默认的开发模式。


## 1. Legacy 模式

同步更新不可被打断，但是递归生成整个 Virtual-Dom 树可能执行的时间会很长，造成丢帧卡顿，所以 React 使用**批处理**收集多个 `setState()` 在某一时段批量执行更新。

**setState() 源码中的注释：**

`setState()` 源码中的注释：
> * There is no guarantee that `this.state` will be immediately updated, so accessing `this.state` after calling this method may return the old value.
> * There is no guarantee that calls to `setState` will run synchronously,
> * as they may eventually be batched together.  You can provide an optional callback 
> * that will be executed when the call to setState is actually completed.
> * When a function is provided to setState, it will be called at some point in the future (not synchronously). It will be called with the up to date component arguments (state, props, context). These values can be different from this.* because your function may be called after receiveProps but before shouldComponentUpdate, and this new state, props, and context will not yet be assigned to this.

大意是：
- 不能保证 `this.state` 会立即更新，在调用完 `setState` 后立马取 `this.state` 有可能取到的是旧的值。
- 不能保证对 `setState` 的调用是同步运行的，它们最终可能会组合在一起被调用。您可以提供一个可选的回调函数，该函数将在对setState 的调用实际完成时执行。
- 当给 `setState()` 传函数作为第一个参数时，该函数不会被同步调用，而是在将来的某个时刻被调用。它被调用时将使用最新的组件参数（state， props，context）。这些参数可能与 this 取到的不同。因为这个函数可能在 `receiveProps` 和 `shouldComponentUpdate` 之间被调用，此时新的 state，props和 context 还未被分配给 this。

**setState() 的源码实现**

在 React 的 `setState()` 函数实现中，会根据一个变量 `isBatchingUpdates` 判断是直接更新 `this.state` 还是放到队列中回头再说。默认 `isBatchingUpdates: false`，也就表示默认情况下 `setState()` 会同步更新 `this.state`。  

但是，有一个函数 `batchedUpdates()`，会令 `isBatchingUpdates: true`，而当 React 在调用事件处理函数之前就会调用函数 `batchedUpdates()`，结果就是由 React 控制的事件处理过程 `setState()` 不会同步更新 `this.state`。  
![image](https://user-images.githubusercontent.com/31687804/175802894-7cfe4002-c29e-4c77-97af-27ff349ac3dd.png)

在 `batchedUpdates()` 函数中， JS 代码未执行完成时，`executionContext` 被标记为 `BatchedContext`；同步执行完 JS 代码后，`executionContext` 清除 `BatchedContext` 标记。


**生命周期函数和合成事件中：** 

- 无论调用多少次 `setState()`，都不会立即执行更新。而是将要更新的 state 存入 `'_pendingStateQuene'`，·将要更新的组件存入 `'dirtyComponent'`;
- 当根组件 didMount 后，批处理机制更新为 `false`。此时再取出 `'_pendingStateQuene'` 和 `'dirtyComponent'` 中的state和组件进行合并更新；

**原生事件和异步代码中：**

- 原生事件不会触发 React 的批处理机制，因而调用 `setState()` 会直接更新；
- 异步代码中调用 `setState()`，由于js的异步处理机制，异步代码会暂存，等待同步代码执行完毕再执行，此时 React 的批处理机制已经结束，因而直接更新。

## 2. Concurrent 模式

Concurrent 模式下的所有更新都是异步的，不管同步的调用还是在异步函数中调用 `setState()`，都是异步更新。
异步可中断更新机制的实现依赖于 React Fiber 在运行时的优化，这与浏览器的渲染过程相关，见文章 [《浏览器的渲染流程及 React Fiber 的运行时优化思路》](https://github.com/CuriosityLxn/Fantastic-Blog/blob/main/%E6%B5%8F%E8%A7%88%E5%99%A8/%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9A%84%E6%B8%B2%E6%9F%93%E6%B5%81%E7%A8%8B%E5%8F%8A%20React%20Fiber%20%E7%9A%84%E8%BF%90%E8%A1%8C%E6%97%B6%E4%BC%98%E5%8C%96%E6%80%9D%E8%B7%AF.md)。

