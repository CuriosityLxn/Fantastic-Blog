## 移除 JSX 中的 import React 

React 16 及以前，编写组件时即便没有使用到 React 中的任何方法，也需要显式的 import React，否则会报错 `React not found`。
```js
import React from 'react';

export default function Hello() {
    return <div>Hello</div>;
}
```

因为 JSX 编译完是这样的：

```js
import React from 'react';

export default function Hello() {
    return React.createElement('div', null, 'Hello');
}
```

React 17 移除了 JSX 中对 React 的引用，增加了新的依赖 `react/jsx-runtime` 来处理 JSX 的编译，编译完：

```js
import {jsx as _jsx} from 'react/jsx-runtime';

export default function Hello() {
    return _jsx('div', { children: 'Hello' });
```

