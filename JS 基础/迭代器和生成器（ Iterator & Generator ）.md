这块建议看《深入理解 ES 6》，讲的简洁易懂。不用看规范，规范为了严谨写了很多重复内容，看规范很浪费时间。

## 迭代协议（帮助理解，不是非要了解，可直接跳过）

#### 迭代器结果协议

包含两个**必要属性**：
| 属性 | 值 | 要求 |
| --- | --- | --- |
| done | bool 类型 | 迭代器 **next** 方法调用的结果状态。如果被迭代的序列在当前节点终止，为 true，否则为 false。 |
| value | 任意类型，包括假值 |和 done 一起存在时，是该节点的值。若迭代器没有返回值，则 **value** 为 **undefined**，该属性可能不存在。 |

如果 **done** 属性（不管是自有的还是继承来的）不存在，会被当作 **done** 为 **false**。

![image](https://user-images.githubusercontent.com/31687804/123943978-3d9ba400-d9cf-11eb-8fa1-f5c3b6ad2474.png)

#### 可迭代协议

| 属性 | 值 |
| --- | --- |
| @@iterator 内部插槽 | 无参数函数，返回一个**迭代器对象** |

@@iterator 函数可在迭代器的原型对象中被访问到，属性名是 **"[Symbol.iterator]"**。

#### 迭代器协议

| 属性 | 值 | 要求 |
| --- | --- | --- |
| next | 函数，返回一个**迭代器结果对象** | **done** 为 **true** 的节点之后可以继续调用该方法，所返回的迭代器结果对象的**done** 属性都为 **true**。 ｜
| return | 函数，返回一个**迭代器结果对象** | 可在在迭代器末尾之前提前结束，且通知迭代器不会继续调用 next() 方法。返回的迭代器结果对象的 **done** 属性为 **true**，**value** 的值是传给 **return** 方法的实参。 ｜
| throw | 函数，返回一个**迭代器结果对象** | 迭代器继续执行的同时抛出错误。若迭代器不进行 **throw** 操作，则返回的迭代器结果对象的 **done** 属性为 **true**。｜

这三个方法可以接受传参，参数的解析和有效性取决于迭代器如何处理及是否使用。

![image](https://user-images.githubusercontent.com/31687804/123948757-5d819680-d9d4-11eb-90b3-65f6fbe628e0.png)

所有的迭代器对象除了可以通过 next() 获取节点的值，也可以通过 **return()** 方法，或者通过 **throw()** 方法抛出异常。这两个方法也可以接受传参。

#### 异步可迭代协议及异步迭代器协议

与上述同步协议类似，区别是异步可迭代协议返回的是**异步迭代器对象**，异步迭代器协议的 next、return、throw 方法返回的是一个符合**Promise 类型的迭代器结果对象 **。  
- **next 方法**：返回的 promise 如果完成了，则 promise 将以符合迭代器结果协议的对象完成。 value 属性值是迭代器结果对象，且不能是 Promise 类型或 thenable 的。
- **return 方法**：传参如果是一个被拒绝的 promise，则返回的是一个相同原因的 promise 拒绝；如果传参是完成的 promise，那么它的完成值会被作为返回的 Promise 迭代器结果对象的 value 属性的值。 
- **throw 方法**：返回一个**被拒绝的 promise**，这个 **Promise** 的拒绝是传参的值。

## 迭代器（Iterator）

- 用途：用于遍历JS表示“集合”的数据结构，这些集合必须是可迭代对象，主要是 Array、Object、Map、Set。
- 本质：是一个**指针对象**。   

迭代器的每个节点由**一个迭代器结果对象**和**一个指针**组成，指针指向下一个节点，调用迭代器对象的 next() 方法，显示迭代到下一个节点，消耗它并返回它的值。    
> ⚠️ 迭代器只是定义出一个序列形式的接口形式，这个序列可以理解为**单链表**，**和它要遍历的数据结构（可迭代对象）一般是分开的**。 
> 迭代器和迭代语句（for-of、for-in）通过对可迭代对象的 Symbol.iterator 属性实现迭代。

![image](https://user-images.githubusercontent.com/31687804/123948487-07145800-d9d4-11eb-977e-b834858ad026.png)

## 可迭代对象（Iterable Object）

根据可迭代协议，凡事具有 Symbol.iterator 属性的对象就是可迭代对象，Symbol.iterator 属性是一个返回作用于当前对象的迭代器的函数。  
ES 6 内置可迭代对象：所有的集合对象和字符串，都有默认迭代器。  


### 内建迭代器
内建可迭代对象提供了内建迭代器，可以用三种大类型概括这些类型：集合对象、字符串、类数组。

- 集合对象：Array、Map、Set、TypedArray。
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

## 访问默认迭代器

```javascript
let values = [5,2,1];
let iterator = values[Symbol.iterator]();
console.log(iterator);
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
```
![image](https://user-images.githubusercontent.com/31687804/123607797-c92afe80-d830-11eb-9daa-a2a6fc3c5248.png)


## 用于操作可迭代对象的语句

- 迭代语句：for-of、for-in、for-await-of、while、do-while
- yield 语句
- 展开运算符及解构赋值

这些语句仅可用于可迭代对象，用于非可迭代对象时会抛出异常。

##### for-of 循环相较于一般 for 循环的优势
使用一般 for 循环遍历可迭代对象时，通常需要一个索引来记录当前遍历到的位置，索引出错则会导致遍历结果出错。  
用 **for-of 循环 + 迭代器** 遍历可以杜绝这个问题，因为索引被内置了，外界无法访问、改动。  

##### 执行原理：
通过 Symbol.iterator 方法返回迭代器，每次执行都会调用一次 next() 方法，将返回的 value 赋值给一个变量。当迭代器返回的 done 为 true 时，终止循环，不再赋值。

## 生成器（Generator）

规范中对于生成器（Generator）有两种描述：Generator 函数对象（GeneratorFunction Objects）和 Generator 对象（Generator Objects）。当提到生成器的时候，要先明确指的是 Generator 函数还是 Generator 对象。

> 《ES 2022 27.3 GeneratorFunction Objects》中对 Generator 函数对象的描述：  
> GeneratorFunction objects are functions that are usually created by evaluating GeneratorDeclarations, GeneratorExpressions, and GeneratorMethods. They may also be created by calling the %GeneratorFunction% intrinsic.  

意译：通常情况下，Generator函数 对象是在执行`生成器声明、生成器表达式和生成器方法`时，被创建出的函数，也可以被生成器函数自身创建出。

> 《ES 2022 27.5 Generator Objects》中对 Generator 对象的描述：  
> A Generator object is an instance of a generator function and conforms to both the Iterator and Iterable interfaces.  
>
> Generator instances directly inherit properties from the object that is the initial value of the "prototype" property of the Generator function that created the instance. Generator instances indirectly inherit properties from the Generator Prototype intrinsic, %GeneratorFunction.prototype.prototype%.     

意译：Generator 对象是 Generator 函数的实例，且同时遵守迭代器协议和可迭代协议。      
生成器实例直接继承一个对象中的属性，这个对象是创建该实例的生成器函数的 **"prototype"** 属性的初始值。Generator 实例间接继承了 Generator 原型的固有属性，即 `GeneratorFunction.prototype.prototype`。

概括一下两者关系，**Generator 函数返回一个 Generator 对象，Generator 对象既是迭代器又是可迭代对象**。    

Generator 方法语法（ES 5，不常用）：   
`* class 元素名(唯一形参) { 生成器体 }`    
  

#### Generator 函数   

语法：    

生成器函数声明：    
- `function * 绑定标识符(形参) { 生成器体 }`       
- `function * (形参) { 生成器体 }`

生成器函数表达式：   
`function * 绑定标识符 opt (形参) { 生成器体 }`
生成器体：即`函数体`   

优点：自定义迭代器需要显式地手写 next() 函数体，并显式地写出 next() 返回的 {done, value} 结构，Generator 函数允许定义一个包含自有迭代算法，同时它可以自动维护自己的状态。

调用过程：

1. 最初调用，不执行任何代码，返回 Generator 迭代器；
2. 调用生成器的 next 方法时，执行 Generator 函数，遇到下一个 yield 关键字之前的所有语句。
3. 每次调用生成一个新的 Generator 迭代器，但每个 Generator 只能迭代一次。

#### yield 语句

语法：

Yield 表达式：     
- `yield`    
- `yield 表达式`    
- `yield * 表达式` 

Generator 函数的执行在遇到 `yield` 语句时会暂停执行，`yield` 返回一个迭代器结果对象，迭代器结果对象 **value** 属性的值是 `yield` 后面表达式的求值结果，**done** 属性标明 Generator 函数是否执行到末尾（若有 return，则 return 处为末尾）。      
Generator 迭代器再次调用 next() 方法时，恢复 Generator 函数的执行。     


## 创建可迭代对象

默认创建的对象是一般对象，通过给 Symbol.iterator 属性添加一个生成器的方式，可将其变为可迭代对象。
```javascript
let collection = {
  items:[],
  *[Symbol.iterator]() {
    for (let item of this.items) {
      yield item;
    }
  }
};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

console.log(collection.items); // output: [1, 2, 3]
```
## 迭代器进阶用法：、异步

### 给迭代器传值

通过上文已知，迭代器通过调用自身的 next() 方法返回的 value 属性向外传值，且可以通过给 next()、return()、throw() 方法传参的方式，实现了向迭代器内部传值。   
- **next(param)**：param 会替换掉上一个迭代器的返回值，参与执行本段代码（指到下一个 yield 语句之前的所有代码）。
- **return(param)**：为当前迭代器制定返回值为 param，并将迭代器状态设置为 true，即返回 `{value: param, done: true}`，并提前退出 Generator 函数执行。
- **throw(param)**：通常情况下 param 是个 Error 对象，根据迭代器协议，调用 throw 之后的代码是否继续执行，取决于迭代器是否进行 throw 操作。

> P.S. 结束函数执行的两种方法：返回（return）或抛出错误（throw）

**举例**    
```javascript
// 给 next 传值，改变 Generator 函数执行时的值

function *createIterator() {
  let first = yield 1;
  let second = yield first + 2;
  yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next());  // {value: 1, done: false}
console.log(iterator.next(4)); // {value: 6, done: false}
console.log(iterator.next(5)); // {value: 8, done: false}
console.log(iterator.next());  // {value: undefined, done: true}

// 给 return 传值，改变 Generator 函数结束节点及返回值
// 只是 return，不返回任何值
function *createIterator() {
  yield 1;
  return;
  yield  2;
  yield 3;
}

let iterator = createIterator();

console.log(iterator.next());  // {value: 1, done: false}
console.log(iterator.next());  // {value: undefined, done: true}

// return 一个值，会将这个值赋给 return 返回的迭代器结果对象的 value 属性
function *createIterator() {
  yield 1;
  return 42;
}

let iterator = createIterator();

console.log(iterator.next());  // {value: 1, done: false}
console.log(iterator.next());  // {value: 42, done: false}
console.log(iterator.next());  // {value: undefined, done: true}

// 给 throw 传值，迭代器错误处理
// 不处理异常
function *createIterator() {
  let first = yield 1;
  let second = yield first + 2;
  yield second + 3;
  yield second + 6;
}

let iterator = createIterator();

console.log(iterator.next());  // {value: 1, done: false}
console.log(iterator.next(4)); // {value: 6, done: false}
console.log(iterator.throw(new Error('This is an Error!')));  // 抛出错误：Uncaught Error: This is an Error!
console.log(iterator.next());  // {value: undefined, done: true}
console.log(iterator.next());  // {value: undefined, done: true}

// 捕获并处理异常
function *createIterator() {
  let first = yield 1;
  let second;

  try{
    second = yield first + 2;
  } catch (err) {
    second = 6;
  }
  yield second + 3;
  yield second + 6;
}

let iterator = createIterator();

console.log(iterator.next());  // {value: 1, done: false}
console.log(iterator.next(4)); // {value: 6, done: false}
console.log(iterator.throw(new Error('This is an Error!'))); // {value: 9, done: false}
console.log(iterator.next()); // {value: 12, done: false}
console.log(iterator.next());  // {value: undefined, done: true}

```

### Generator 函数委托

yield 语句中有一种`yield * 表达式`，这个表达式可以是一个方法或一个 Generator 函数，表示正在执行的 Generator 函数，将运行时执行上下文，委托给另一个方法或其他迭代器。这意味着不仅可以向迭代器内传入简单的值，也可以传入将函数的返回结果，或者索性将生成数据的过程委托给其他迭代器。
```javascript
// 委托给其他生成器函数
function* createNumberIterator() {
  yield 1;
  yield 2;
  return 3;
}

function* createForIterator(count) {
  for (let i = 0; i < count; i++) {
    yield `repeat ${i}`;
  }
}

function* createCombinedIterator() {
  let res = yield* createNumberIterator();
  yield* createForIterator(res);
}

let iterator = createCombinedIterator();

console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: "repeat 0", done: false}
console.log(iterator.next()); // {value: "repeat 1", done: false}
console.log(iterator.next()); // {value: "repeat 2", done: false}
console.log(iterator.next()); // {value: undefined, done: true}

// 委托给字符串的默认迭代器
function* createStrIterator() {
  yield * 'hua';
}

let iterator = createStrIterator();
iterator.next(); // {value: "h", done: false}
iterator.next(); // {value: "u", done: false}
iterator.next(); // {value: "a", done: false}
```

### 实现异步任务执行



## 总结

- 满足 `有 Symbol.iterator 属性且Symbol.iterator 属性是返回迭代器对象的无参数函数` 的对象就是可迭代对象。
- 从含义上讲，Generator 函数是生成迭代器的函数。
- 就像 Array 是 Object 一样，Generator 对象是 迭代器对象。
- 自定义迭代器对象需要显示维护其状态，Generator 迭代器自维护其状态。
- 简记：
  -  可迭代对象：Object + [Symbol.iterator]: () {return 迭代器对象;}
  -  迭代器对象：Object + next() {return 迭代器结果对象;}
  -  Generator 对象：Object + [Symbol.iterator]: () {return 迭代器对象;} + next() {return 迭代器结果对象;}


#### 参考资料
- [ECMA262 规范-27.1 常用迭代协议](https://tc39.es/ecma262/#sec-common-iteration-interfaces)
- [ECMA262 规范-27.3 生成器函数对象](https://tc39.es/ecma262/#sec-generatorfunction-objects)
- [ECMA262 规范-27.5 生成器对象](https://tc39.es/ecma262/#sec-generator-objects)
- [ECMA262 规范-14.7.5.10 for-in 迭代器对象](https://tc39.es/ecma262/#sec-for-in-iterator-objects)
- [ECMA262 规范-14.7.5.9 EnumerateObjectProperties ( O ) 可列举的对象属性](https://tc39.es/ecma262/#sec-enumerate-object-properties)
- [MDN-迭代协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)
- [MDN-迭代器和生成器](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators)
- [MDN-yield](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield)
- 《深入理解 ES 6-迭代器和生成器》
- [《ECMAScript 6 入门-17.Iterator 和 for...of 循环
》阮一峰](https://es6.ruanyifeng.com/#docs/iterator)
- [《ECMAScript 6 入门-18.Generator 函数的语法》阮一峰](https://es6.ruanyifeng.com/#docs/generator)
- [《ECMAScript 6 入门-19.Generator 函数的异步应用》阮一峰](https://es6.ruanyifeng.com/#docs/generator-async)



