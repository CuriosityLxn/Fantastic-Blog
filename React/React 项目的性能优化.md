## 为什么需要性能优化

React 通过 diff 算法，生成新的 Virtual Dom 树，替换掉旧的 Virtual Dom 树，来实现视图更新。

任意节点的 state 发生改变时，就会以该节点为父节点、dfs 更新其子树的每一个叶子节点，触发子孙元素的 re-render。即便子孙节点的 props 和 state 没有发生变化。

这就会产生不必要的计算、渲染开销。

## 优化思路

- 代码结构：将「变的部分」和「不变的部分」分离。
- 良好代码结构的基础上：使用性能优化 API 或第三方库，确保引用类型的 state “Immutable”。


#### 「变」与「不变」分离

**变的部分：**
- props
- state
- context

props、context 都源于父组件的 state。

**优化方案：**
1. 让父组件 state 变化不影响到不使用该 state 的子组件。
2. 确保子组件自身的 state 稳定。

#### 性能优化 API

重点：如何判断 props 和 Hooks 的 deps 是否改变。

**React 默认使用全等比较（===）。**    
全等比较判断的是「引用相等」，可以准确判断基本类型的值是否相同，对于引用类型值，判断的是其堆内存地址的引用是否相同。

特殊：即便组件没有显式接受 props，在源码中默认 props 是空对象，`({}) === ({})` 为 `false`。

**性能优化 API 使用 React 的 `shallowEqual()` 方法进行浅比较。**

在 `shallowEqual()` 中，
- 先用 `Object.is()` 判断是否原值相等
- 若不等则用 `typeof` 判断是否是 `object` 类型
- 遍历对象的 key，再用 `Object.is()` 判断两个对象相同 key 值对应的 value 是否原值相等。

所以 React 浅比较会认为空数组和空对象也是相等的。

**性能优化 API：**

- `shouldComponentUpdate(props, state)`：类组件的生命周期函数，开发者可以手动比较新旧 props 和 state 来判断是否更新组件。
- `PureComponent`：类组件中的纯组件，自动在 `shouldComponentUpdate()` 做了 `shallowEqual()`。
- `React.memo()`：用于函数组件，也是自动做了 `shallowEqual()`。
- `useMemo(fn, deps)`：类似于 `React.memo()`，不过是 Hook，还可以用于缓存函数的执行结果。
- `useCallback(fn, deps)`：用于缓存函数组件的内置函数，避免随着 re-render 多次重新给函数声明划分堆内存，因为函数的功能通常都是相同的。

## React 项目性能优化具体步骤

1. 寻找项目中性能损耗严重的子树。
2. 在子树的根节点（父组件）运用「变」与「不变」分离原则，形成好的代码结构，命中性能优化策略。
3. 在子树（子组件）中使用性能优化 API，保持子组件的 state 稳定。
