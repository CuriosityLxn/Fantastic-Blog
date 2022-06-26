## React

- 添加 `<React.Profiler>` API，以可编程的方式收集性能估算。

## React Dom
- **废弃 `UNSAFE_*` 生命周期方法的旧名称**：旧名称在这个版本仍然可以用，但会抛出警告。可用下面的命令做全局命名修改：

```shell
cd your_project
npx react-codemod rename-unsafe-lifecycles
```

- **废弃 `javascript: URLs`**:`javascript: URLs` 很容易包含不安全的代码在 `<a href>` 之类的标签中输出。这个版本中仍可以使用，但会抛出警告。在下一个版本中，React 将抛出错误。如果非要用，用 React 事件处理。

```html
const userProfile = {
  website: "javascript: alert('you got hacked')",
};
// This will now warn:
<a href={userProfile.website}>Profile</a>
```

- **废弃“工厂模式”组件**


## React Test

- **异步的 `act()` 测试方法**

```js
await act(async () => {
  // ...
});
```
- 支持从不同的 renderer 
