## 概述 - React 是什么

```javascript
// 总的来说
const UI = fn(state);

// 展开一下
const state = reconcile(update);
const commit = fn;
const UI = commit(state);
```
React 就是 fn，根据数据状态来渲染 UI。
- update：数据请求、用户交互等。
- reconcile 算法（diff 算法）计算状态变化，改变 state。
- renderer 将 state 变化渲染到视图中。

浏览器中的 renderer 是 ReactDom，APP 中的 renderer 是 React Native。

## React 15 架构缺点

在 Reconciler 中，`mount` 的组件会调用 [mountComponent](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L498) `update` 的组件会调用 [updateComponent](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L877)，这两个方法会递归更新子组件。
递归执行的缺点是**一旦开始，无法中断**，当层级很深、渲染时长超过 16ms（浏览器刷新频率为60Hz，约为 16.6ms）会掉帧卡顿。

## React 16 架构更新

1. 除 Reconciler、Renderer 外，新增
- Scheduler 调度器，负责调度任务的优先级，

优先级高的任务先进入 Reconciler。

2. <= React 15 之前使用 stack reconciler，React 16 及以后默认使用 fiber reconciler，优势：
    1. **stack 形式的是同步执行**，**fiber 形式的可中断的任务切片处理，异步执行**；
    2. fiber 形式的可以重置并复用任务，也可以在 render() 中返回多个元素

## state 更新 ==> 视图更新

// 图片上传失败，等翻墙OK了再传。

## Virtual Dom Tree 的创建与更新

首屏渲染，

1. 调用 ReactDom.render，
2. 进入 render 阶段，
3. 深度优先遍历递归创建 fiber 树（虚拟 DOM 树）；
4. 进入 commit 阶段，ReactDom 将 fiber 树渲染到视图，完成后从子向父执行生命周期函数 componentDidMount。


状态改变时，

1. 调用 `this.setState`，传递状态变化；
2. 进入 render 阶段，
3. 深度优先遍历递归创建**新的** fiber 树（虚拟 DOM 树），diff 算法计算并标记产生变化的节点；
4. 更新视图中产生变化的节点；
5. 新的 fiber 树替换掉旧的。

⚠️注意：
1. 每次调用 `this.setState` 都会创建新的 fiber 树。
2. 没有发生变化的节点不会重新调用生命周期函数。
3. 改变状态的方法还有 `this.forceUpdate`、`ReactDOM.render`等。
4. 虚拟 DOM 由 `React.createElement()` 方法返回，是一个描述真实 DOM 的 JS 对象。



