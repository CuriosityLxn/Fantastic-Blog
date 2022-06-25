## 浏览器渲染进程

### 渲染帧

一般浏览器刷新频率是 60 HZ，也就是 1s 刷新 60 帧。

渲染一帧的时长约：16.67ms。

丢帧：某一帧渲染时长超过了 16.67 ms，用户视觉上会产生卡顿，即丢帧。


### 渲染帧的生命周期

**主线程（Main Thread）处理：** 

1. Event Handlers：处理 UI 交互回调。
2. animation：由 requestAnimationFrame() 调度，确保下一帧开始时动画立即进行。
3. parse HTML：构建 DOM tree。当 JS 操作 DOM 时，会重新触发该流程。
4. parse CSS：构建 CSS tree。
5. Render Tree：DOM tree + CSS tree = Render Tree。
6. Layout：所有元素的 position、size 等信息。
7. Paint：分层（渲染层、合成层），像素填充，eg：颜色、文字、边框等，为可视元素生成详细的绘制信息，将信息传递给合成线程。
10. requestIdleCallback：在刷新这一帧完成之前，若有空余时间则执行该回调。

**合成线程（Composite Thread）处理：**

8. Raster：将信息分块，将每块信息发给光栅线程。

**光栅线程处理：**

9. 创建位图，通知 GPU 进程刷新这一帧。


**需要注意的是，**
- Render Tree 不是等完整的 DOM tree 和 CSS tree 都构建完成才开始构建，而是 DOM tree 和 CSS tree 构建一部分，随之构建一部分的 Render Tree，交替进行。
- Paint 不是指像素渲染在屏幕上，而是计算出可视元素的具体渲染信息。
- requestIdleCallback() 可以做一些**不重要、不紧急**的事情，比如数据上报。
  - **不要操作已经渲染在视图上的 DOM，但是可以做懒加载**，因为前面的步骤已经完成了 DOM 构建。
  - **不要执行 Promise**：执行 Promise 无法保证在当前帧结束前可以拿到数据，造成丢帧。

## 为什么会产生丢帧现象

渲染之前会执行：

1. Event Handlers：用户 UI 交互太过频繁，执行时间过长。
2. Animation：动画过大。
3. 大量 JS 运算：浏览器 JS 进程和渲染进程互斥，解析 JS 占用渲染时间，浏览器不能及时处理下一帧。


## 递归更新的缺点

React 15 采用的 Reconciler 是通过递归的方式同步执行更新，一旦更新开始，无法被中断，必须执行完。递归层级很深时，就容易掉帧。

当组件 mount 或 update 时，会调用 mountComponent、updateComponent 方法，递归更新子组件。


## Fiber 解决丢帧

将一个大任务分割成多个小任务，在每一帧有空余时分别执行（利用 `requestIdleCallback()` ）。**实现了可中断的异步更新**。

这就是 Fiber 的基本思想。但 Fiber 并没有采用 `requestIdleCallback()` 来实现分片执行，原因是

- 浏览器兼容
- 触发频率不稳定。比如当切换浏览器 tab 后，原 tab 注册的 `requestIdleCallback()` 触发频率会降低。

所以 React 自行实现了 `requestIdleCallback()` 的 polyfill，即 **Scheduler**。

> **Scheduler** 是基于 React 实现的库。

至此，React 16的架构变为：Scheduler + Reconciler + Renderor

## Fiber 节点 <=> DOM 节点

> 双缓存：在内存中构建并直接替换的技术。

React 使用“双缓存”来完成 `Fiber Tree` 的构建与替换 —— 对应 `DOM Tree` 的创建与更新。

`current Fiber Tree` 上的 每个 Fiber 节点通过其 `alternate 属性` 与 `workInprogress Fiber Tree` 上对应的节点连接。

![image](https://user-images.githubusercontent.com/31687804/175778872-5dbc7965-7af1-4a0f-8520-f30d70c43eba.png)

### 更新 Fiber Tree

1. `ReactDOM.render` 构建 `fiberRoot` 节点和 `rootFiber`节点。

> `fiberRoot` 节点：整个应用的根节点，一个应用只会有一个。
> `rootFiber`节点：<App/> 组件树的根节点，多次调用 `ReactDOM.render` 就有多个 `rootFiber`。

2. `fiberRoot` 节点的 `current` 指向当前渲染的 `Fiber Tree`，即 `Current Tree`。
3. 进入 `render` 阶段，根据组件 JSX 依次创建 `Fiber` 节点，构建出的 `Fiber Tree` 被称为 `workInProgress Fiber Tree`。构建时会尝试复用 `current Fiber Tree` 中已有的 `Fiber` 节点内的属性。
4. `workInprogress Fiber Tree` 构建完成后，进入 `commit` 阶段渲染到页面上。渲染完毕后，`workInProgress Fiber Tree` 变为 `current Fiber Tree`：







