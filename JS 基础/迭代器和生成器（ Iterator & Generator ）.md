## 迭代器（Iterator）

- 用途：用于遍历JS表示“集合”的数据结构，这些集合必须是可迭代对象，主要是 Array、Object、Map、Set。
- 本质：是一个**指针对象**。
- 要注意迭代器只是定义出一个序列形式的接口形式，这个序列可以理解为**单链表**，**和它要遍历的数据结构实际上是分开的**。    

每个节点的信息域，存放一定是一个对象，包含两个**必要属性**：
  - value：和 done 一起存在时，是该节点的值，可以是任何类型，包括假值，但字段必须有
  - done：bool 类型，如果序列在当前节点终止，为 true，否则为 false。  
每个节点的指针域指向下一个节点，调用迭代器对象的 next() 方法，显示迭代到下一个节点，消耗它并返回它的值。

![image](https://user-images.githubusercontent.com/31687804/122403195-c28ac480-cfb0-11eb-891b-a2232b248dd4.png)

PS. 既然是链表，就不难理解：  
在序列终止时可以返回一个值，也可以不返回。  
序列中若有 return，当即终止；否则，会在末尾终止。

## 生成器（Generator）

> Generator，又叫生成器对象，既是迭代器，也是可迭代对象。[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#%E7%94%9F%E6%88%90%E5%99%A8%E5%AF%B9%E8%B1%A1%E5%88%B0%E5%BA%95%E6%98%AF%E4%B8%80%E4%B8%AA%E8%BF%AD%E4%BB%A3%E5%99%A8%EF%BC%8C%E8%BF%98%E6%98%AF%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%BF%AD%E4%BB%A3%E5%AF%B9%E8%B1%A1%EF%BC%9F)
- 作用：用于自定义的迭代器，需要显式地维护其内部状态，使用 `function*` 语法编写。
- 生成了什么：一个包含自有迭代算法的函数（），同时它可以自动维护自己的状态。

调用过程：

1. 最初调用，不执行任何代码，返回 Generator 迭代器；
2. 调用生成器的 next 方法时，执行 Generator 函数，遇到下一个 yield 关键字之前的所有语句。
3. 每次调用生成一个新的 Generator，但每个 Generator 只能迭代一次。

