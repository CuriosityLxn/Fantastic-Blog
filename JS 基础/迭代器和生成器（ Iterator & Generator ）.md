这块建议看《深入理解 ES 6》，讲的简洁易懂。不用看规范，规范为了严谨写了很多重复内容，看规范很浪费时间。

## 迭代协议（帮助理解，不是非要了解，可直接跳过）

#### 迭代器结果协议

包含两个**必要属性**：
  - value：和 done 一起存在时，是该节点的值，可以是任何类型，包括假值。
  - done：bool 类型，如果序列在当前节点终止，为 true，否则为 false。  

![image](https://user-images.githubusercontent.com/31687804/123583358-0e8c0380-d812-11eb-9742-605802af80a3.png)

#### 可迭代协议

必须有 **@@iterator** 内部插槽，它是一个无参数函数，返回一个**迭代器对象**。  
@@iterator 函数可在迭代器的原型对象中被访问到，属性名是 **"[Symbol.iterator]"**。

![image](https://user-images.githubusercontent.com/31687804/123584268-a4745e00-d813-11eb-849c-f57572dc8aa1.png)

#### 迭代器协议

必须有 **next** 属性，next() 方法是一个函数，返回一个**迭代器结果对象**。  
next() 方法可以接受传参，参数的解析和有效性取决于迭代器如何处理及是否使用。

所有的迭代器对象除了可以通过 next() 获取节点的值，也可以通过 **return** 在迭代器末尾之前提前返回，或者通过 **throw** 方法抛出异常。这两个方法也可以接受传参。

#### 异步可迭代协议及异步迭代器协议

与上述同步协议类似，区别是异步可迭代协议返回的是**异步迭代器对象**，异步迭代器协议的 next、return、throw 方法返回的是一个符合**Promise 类型的迭代器结果对象 **。  
- **next 方法**：返回的 promise 如果完成了，则 promise 将以符合迭代器结果协议的对象完成。 value 属性值是迭代器结果对象，且不能是 Promise 类型或 thenable 的。
- **return 方法**：传参如果是一个被拒绝的 promise，则返回的是一个相同原因的 promise 拒绝；如果传参是完成的 promise，那么它的完成值会被作为返回的 Promise 迭代器结果对象的 value 属性的值。 
- **throw 方法**：返回一个**被拒绝的 promise**，这个 **Promise** 的拒绝是传参的值。


## 迭代器（Iterator）

- 用途：用于遍历JS表示“集合”的数据结构，这些集合必须是可迭代对象，主要是 Array、Object、Map、Set。
- 本质：是一个**特殊对象**。   

迭代器的每个节点由**一个迭代器结果对象**和**一个指针**组成，指针指向下一个节点，调用迭代器对象的 next() 方法，显示迭代到下一个节点，消耗它并返回它的值。    
> ⚠️ 迭代器只是定义出一个序列形式的接口形式，这个序列可以理解为**单链表**，**和它要遍历的数据结构（可迭代对象）一般是分开的**。 
> 迭代器和迭代语句（for-of、for-in）通过对可迭代对象的 Symbol.iterator 属性实现迭代。

### 内建迭代器
一些内建类型提供了内建迭代器，可以用三种大类型概括这些类型：集合对象、字符串、类数组。

- 集合对象：Array、Map、Set。
- 字符串：String
- 其他可迭代对象：主要指类数组，如 NodeList、arguments

##### 集合对象迭代器

内建类型**Array、Map、Set**的原型对象中实现了 @@iterator 方法，且都内建了以下三种迭代器：
- values()：返回一个迭代器，其值为集合的值 [value, value, value]。
- keys()：返回一个迭代器，其值为集合的键名 [key, key, key]。
- entries()：返回一个迭代器，其值为多个键值对 [[key, value], [key, value]] 。

Array、Set 默认使用的迭代器是 values() 方法，Map 默认使用的迭代器是 entries() 方法。

> 注意 **WeakMap 和 WeakSet 无法被迭代**，采用弱管理引用，不能确切的知道集合中的值。

##### 字符串迭代器

ES 5 规定可以用方括号访问字符串中的字符。方括号操作的是编码单元而非字符，所以无法正确访问双字节字符。  
ES 6 希望全面支持 Unicode。可以通过 for-of 循环操作字符来解决这个问题

```javascript
var message = 'A 汉 B';

// 方括号通过默认迭代器，访问字符串中的字符
for (let i = 0; i < message.length; i++) {
  console.log(message[i]);  // 现在输出正常的
}

// for-of 改变迭代器，访问字符串中的字符
for (let c of message) {
  console.log(c);
}
```

##### NodeList 的默认迭代器

NodeList 定义在 DOM 标准中，虽然有 length 属性，打印出来看上去也是数组形式，但其实和定义在 ECMAScript 标准中的 Array 是两回事。ES 6 支持默认迭代器后，HTML 标准也给 NodeList 增加了默认迭代器，行为与 Array 相同。  

### 展开运算符



## 生成器（Generator）

> Generator，又叫生成器对象，既是迭代器，也是可迭代对象。  --[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#%E7%94%9F%E6%88%90%E5%99%A8%E5%AF%B9%E8%B1%A1%E5%88%B0%E5%BA%95%E6%98%AF%E4%B8%80%E4%B8%AA%E8%BF%AD%E4%BB%A3%E5%99%A8%EF%BC%8C%E8%BF%98%E6%98%AF%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%BF%AD%E4%BB%A3%E5%AF%B9%E8%B1%A1%EF%BC%9F)
- 作用：用于自定义的迭代器，需要显式地维护其内部状态，使用 `function*` 语法编写。
- 生成了什么：一个包含自有迭代算法的函数，同时它可以自动维护自己的状态。

调用过程：

1. 最初调用，不执行任何代码，返回 Generator 迭代器；
2. 调用生成器的 next 方法时，执行 Generator 函数，遇到下一个 yield 关键字之前的所有语句。
3. 每次调用生成一个新的 Generator，但每个 Generator 只能迭代一次。

