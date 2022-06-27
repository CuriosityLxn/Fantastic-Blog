[ES2020 原文](https://tc39.es/ecma262/#sec-fundamental-objects)

基本对象包括：
- Object 对象
- Function 对象
- Boolean 对象
- Symbol 对象
- Error 对象


### 20.1.1 Object 构造函数

#### MDN 总结
[原文](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/Object)
![MDN 总结.png](https://s1.ax1x.com/2022/06/27/jV0uHP.png)

#### 规范原文及注解
- 是一个 Object。
- 是全局对象的 “Object” 属性的初始值。
- 当作为构造函数调用时，会创建一个新的普通对象。
- 当作为普通函数而不是构造函数调用时，会执行类型转换。
- 可以用作 class 定义中的 extends 语句的值。

##### 20.1.1.1 Object ( [ value ] )

当 Object() 作为普通函数调用时，与 new Object() 表现一致，有可选参数 value：
![[ value ].png](https://s1.ax1x.com/2022/06/27/jV0nBt.png)

> 1.  NewTarget 不是 **undefined** 且不是当前活动函数，

意味着 value 是一个已经存在的值。
> 则返回  OrdinaryCreateFromConstructor(NewTarget, "%Object.prototype%")

就返回一个新对象，这个对象是根据 NewTarget 指向的值的构造函数创建出来的。
```javascript
function fn() {
  console.log('new target:', new.target);
}
var obj1 = new fn();
/*
* new target: ƒ fn() {
*  console.log('new target:', new.target);
* }
*/
var obj2 = Object(obj1);
console.log(obj2 === obj1); // true
console.log(obj1.__proto__ === fn.prototype); 
 // true
console.log(obj2.__proto__ === fn.prototype); // true
```
> 2. 若 value 是 **undefined** 或 **null**，返回 
OrdinaryObjectCreate(%Object.prototype%).

OrdinaryObjectCreate() 方法会返回一个新对象，这个对象是根据挂载在全局对象上的 Object.prototype 创建出的空对象。
![OrdinaryObjectCreate.png](https://s1.ax1x.com/2022/06/27/jV0ZjA.png)

> 3. 返回 ToObject(value)。

将 value 转换成 Object 类型。
![ToObject1.png](https://s1.ax1x.com/2022/06/27/jV0MAf.png)
![ToObject2.png](https://s1.ax1x.com/2022/06/27/jV0mnI.png)


### 20.1.2 Object 构造函数的属性
- 有 [[Prototype]] 属性，值为 Function.prototype。
- 有 **“length”** 属性，值为 1。
- 有静态方法：

#### 对象复制 Object.assign ( target, ...sources )
将一个或多个源对象（sources）中的所有自有可枚举属性，复制到目标对象（target）中。函数返回值是目标对象。  
![assign.png](https://s1.ax1x.com/2022/06/27/jV0QN8.png)

- 若没有参数 sources，将 target 转成相应对象类型并返回。
布尔值转成 Boolean {}，数字转成 Number {3}。
- 若有参数 sources，则对于多个源对象 sources 中的每个源对象 nextSource，做如下操作：
    - 若 nextSource 是 **undefined** 或者 **null**，忽略该源对象。
    - 否则
        - i. 将源对象 nextSource 转成相应对象类型，将结果赋给变量 from；
        - ii. 获取 from 的全部自有属性的 key 值，赋值给变量 keys；
        - iii. 对于 keys 中的每个元素 nextKey，执行 
```javascript
let propValue = from.[[Get]](nextKey, from);
to.[[Set]](nextKey, propValue, to);
```
assign 函数的 length 为 2。

⚠️：对于访问器属性的拷贝，并不是简单的 `to.nextKey = from.nextKey`，而是执行了对应的 getter、setter 函数。所以目标对象拷贝到的访问器属性值是源对象属性 getter 函数的执行结果。

#### 对象创建 Object.create ( O, Properties )
使用指定的原型对象和属性创建新对象。函数返回值是创建出的新对象。

![create.png](https://s1.ax1x.com/2022/06/27/jV0j58.png)

- 参数 O 是要操作的目标对象.
- 参数 Properties 是要添加或修改的**属性描述符对象**列表。

> 1. 如果 O 不是 Object 或者 Null 类型，抛出 **TypeError
TypeError** 异常。
> 2. 根据原型对象 O 创建新对象 obj。

![image.png](https://s1.ax1x.com/2022/06/27/jVBCKs.png)
关键方法 OrdinaryObjectCreate(proto) 创建一个新对象，并将其原型指针指向参数 proto。所以步骤2相当于
```javascript
let obj = {};
obj.__proto__ = O;
```
> 3. 如果 Properties 不是 **undefined**，则给 obj 创建指定属性 Properties，并返回 obj。
> 4. 未指定 Properties 时直接返回 obj。

#### 定义、获取对象的自有属性
**定义对象自有属性**，包括添加、更新属性的特性及值。
- 方法 Object.defineProperties ( O, Properties )，批量定义自有属性。
- 方法 Object.defineProperty ( O, P, Attributes )，单独定义自有属性 P。

**获取自有属性**的操作，可以「获取属性名｜获取属性描述符对象」。
- 方法 Object.getOwnPropertyDescriptor ( O, P )
- 方法 Object.getOwnPropertyDescriptors ( O )
- 方法 Object.getOwnPropertyNames ( O )
- 方法 Object.getOwnPropertySymbols ( O )

##### Object.defineProperties ( O, Properties )
用于给目标对象 O 添加自有属性，或修改已有自有属性的特性字段。函数返回值是目标对象 O。

- 参数 O 是要操作的目标对象.
- 参数 Properties 是要添加或修改的**属性描述符对象**列表。

> 1. O 不是 Object 类型，抛出 **TypeError
TypeError** 异常。
> 2. 返回 ObjectDefineProperties(O, Properties)。

##### ObjectDefineProperties ( O, Properties )
![ObjectDefineProperties.png](https://s1.ax1x.com/2022/06/27/jVBE5T.png)

> 1. 断言：O 是 Object 类型。

因为该函数是操作对象属性的，所以必须保证被操作的 O 是个 Object 类型。
> 2. 将 Properties 转成 Object 类型，赋值给变量 props。
> 3. 获取 props 的全部自有 key 值，赋值给变量 keys。
> 4. let descriptors = [];
> 5. 对于 keys 中的每个元素 nextKey，执行
    > a. 从 props 中获取自有属性 nextKey，将结果赋值给 propDesc；
    > b. propDesc 存在，设置属性 propDesc 的 [[Enumerable]] 插槽为 **true**，然后执行
        > i. let descObj = props.nextKey;
        > ii. 将 descObj 转成属性描述器类型，并赋值给变量 desc；
        > descriptors.push([nextKey, desc]);
> 6. 对于 descriptors 中的每个元素 pair，执行
    > a. let p = pair[0];
    > b. let desc = pair[1];
    > c. 执行 DefinePropertyOrThrow(O, P, desc);
> 7. 返回 O。

步骤 2-7 是根据 Properties 创建每条属性的属性描述符对象。常见属性描述符有两种：数据描述符和访问器描述符，它们包含特性字段如下：

| 描述符类型 | 包含特性字段 | 值类型 | 默认值 
| --- | --- | --- | --- |
| 数据描述符 | [[Value]] | Any |  undefined
| 数据描述符 | [[Writable]] | Boolean |  false
| 访问器描述符 | [[Get]] | Function Object / Undefined |  undefined 
| 访问器描述符 | [[Set]] | Function Object / Undefined |  undefined 
| 共有特性字段 | [[Enumerable]] | Boolean |  false
| 共有特性字段 | [[Configurable]] | Boolean |  false

用代码举例比较清晰：
```javascript
var obj = {};
Object.defineProperties(obj, {
   // 数据描述符对象
  'property1': {
    value: true,
    writable: true
  },
  // 访问器描述符对象
  'property2': {
    get: () => 3
  }
});
console.log(obj);
console.log(obj.property2); // output: 3
```
![console.png](https://s1.ax1x.com/2022/06/27/jVBkV0.png)

##### Object.defineProperty ( O, P, Attributes )

用于给目标对象 O 添加一个自有属性 P，或单独修改已有自有属性 P 的特性字段。
Attributes 是 P 的属性描述符特性字段列表。
函数返回值是目标对象 O。
![defineProperty.png](https://s1.ax1x.com/2022/06/27/jV0xPS.png)

> 1. 若 O 不是 Object 类型，抛出 **TypeError** 异常。
> 2. 将 P 转换成属性 key，并将结果赋值给变量 key。
> 3. 根据 Attributes 创建属性构造器对象，并赋值给变量 desc。
> 4. 执行 DefinePropertyOrThrow(O, key, desc)，判断是否能根据 desc 构造出 O 的自有属性 P。若能则返回 true；否则抛出 **TypeError** 异常。
> 5. 返回 O。

##### Object.getOwnPropertyDescriptor ( O, P )

作用：拿到 O.P 属性的属性描述符。
返回值：属性描述符对象。

##### Object.getOwnPropertyDescriptors ( O )
作用：拿到 O 中所有属性的属性描述符。
返回值：一个新对象，形如：`{
    key1: key1 的属性描述符对象,
    key2: key2 的属性描述符对象
}`。
##### 配合 Object.create() 实现对象浅拷贝
Q：和 Object.assign() 拷贝有什么区别？
A：Object.assign() 拷贝时的限制：
- 只能拷贝源对象的可枚举的自身属性，无法拷贝属性的特性。
- 拷贝时会将访问器属性转换成数据属性。
- 无法拷贝对象的原型对象。
```javascript
var target = { a: 1, b: 2 };
var source = Object.create(
  Object.prototype, {
    b: { value: 4, writable: true, configurable: true, enumerable: true },
    c: { value: 5, writable: true, configurable: true },
    d: {
      enumerable: true,
      get: () => 999
    }
  });
  
console.log(source); // output: {b: 4, c: 5}
console.log(source.d); // output: 999

// Object.assign() 拷贝结果
var returnedObj = Object.assign(target, source);
console.log(returned); // output: {a: 1, b: 4, d: 999}
// Object.create() + Object.getOwnPropertyDescriptors() 拷贝结果
var newObj = Object.create(
  Object.getPrototypeOf(source),
  Object.getOwnPropertyDescriptors(source)
);
console.log(newObj); // output: {b: 4, c: 5}
console.log(newObj.d); // output: 999
```
##### Object.getOwnPropertyNames ( O )、 Object.getOwnPropertySymbols ( O )
 作用：分别获取 string 、symbol 类型值作为名称的所有自有属性的属性名。
 返回值：属性名组成的数组。
##### 关键方法：GetOwnPropertyKeys ( O, type )
接受两个参数：
- O：从 O 中获取属性 key 值
- type：属性名类型枚举 string，symbol
![GetOwnPropertyKeys.png](https://s1.ax1x.com/2022/06/27/jV0z8g.png)

> 1. 将 O 转成相应类型的对象。
> 2. 获取对象中所有自有属性的 key，**不论是否可枚举**。

![img.png](https://s1.ax1x.com/2022/06/27/jVBpvj.png)

> 3. 将符合类型的属性名放进数组中。
> 4. 返回这个数组

##### 和 Object.keys() 的区别

Object.keys() 仅获取可枚举属性名，
GetOwnPropertyKeys() 获取可枚举和不可枚举属性名。

#### 迭代方法：获取可枚举属性
属性顺序与实际顺序一致。
- 方法 Object.entries ( O ) 
- 方法 Object.keys ( O ) 
- 方法 Object.values ( O )

![entries.png](https://i.postimg.cc/TY0z30W4/8-D387312-33-AF-4120-AA65-EA7-EC1567-F29.png)


他们的核心方法都是是 EnumerableOwnPropertyNames()，用于创建可枚举的自有属性名。

![EnumerableOwnPropertyNames.png](https://i.postimg.cc/ZKmmR0sC/7-E25-A349-5399-46-BA-A422-F71-E6-EA9-C4-EC.png)

接受两个参数：
- O：一个对象。
- kind：key 或 value 或 key + value。
> 1. 断言 O 是 Object 类型。
> 2. 获取 O 的自有属性的 keys，并赋值给变量 ownKeys。
> 3. 令变量 properties 是一个新的空列表。
> 4. 对于 ownKeys 中的每个元素 key，执行

当 key 是 String 类型，且 O.key **存在且可枚举**，则
- 情况 1：若 kind 是 key，properties.push(key)；
- 情况 2：若 kind 是 value，properties.push(O.key)；
- 情况 3：若 kind 是 key + value，
```javascript
entry = [key, value]; 
properties.push(entry);
```        
> 5. 返回 properties。

很明显，Object.keys() 命中情况 1，Object.values() 命中情况 2，Object.entries() 命中情况 3。

##### Object.entries() 的逆操作 - Object.fromEntries ( iterable )
作用：将键值对列表转化成一个对象。
返回值：一个新对象。

![fromEntries.png](https://i.postimg.cc/Pqp9KkVg/63732-A3-B-06-F0-4-A37-A606-75-BD978-A1749.png)

> 1. RequireObjectCoercible ( iterable ) 函数校验传参能不能被转成相应对象类型，能则返回原参数 iterable，不能则抛出 TypeError 异常。
> ![RequireObjectCoercible.png](https://i.postimg.cc/251VCT2Q/new.png)
> 2. let obj = {};
> 3. obj 是个可扩展的、没有自有属性的普通对象。
> 4. 根据键值对创建属性的 key 值、value 值。
> **关键方法 AddEntriesFromIterable**( target, iterable, adder ) 通过获取参数 iterable 的 @@iterator 方法，生成一个包含两个元素的类数组对象，第一个元素会被用作 Map 的 key 值，第二个元素被用作 key 关联的 value 值。该函数返回值是 target。
> 5. 返回 AddEntriesFromIterable 函数迭代处理过的 obj。

```javascript
// 借用 MDN 的例子
const entries = new Map([
  ['foo', 'bar'],
  ['baz', 42]
]);

const obj = Object.fromEntries(entries);

console.log(obj);
// output: Object { foo: "bar", baz: 42 }
```

**生成 key 的方法：ToPropertyKey ( argument )**
![ToPropertyKey.png](https://i.postimg.cc/x1dWJmvT/BB324855-BCC1-497-F-8914-1-F7-A995054-A4.png)
- key 为 Object 类型时，ToPrimitive(key, string) 函数调用 key 的 @@toPrimitive 方法，通过 OrdinaryToPrimitive(key, string) 函数，将指定转成 string 类型。调用优先级：对象的 toString 方法 > 对象的 valueOf 方法。
![ToPrimitive.png](https://i.postimg.cc/P5F0mqyb/53-BECFCF-F3-B1-44-FD-A7-AA-B53-E3-BF1-F67-D.png)

- key 为 Symbol 类型时，返回 key。
- 否则将 key 转成 string 类型并返回。

#### 改变对象状态
- 方法 Object.freeze ( O )，冻结对象 O。
- 方法 Object.seal ( O )，封闭对象 O。

**共同点：**
- 返回值都是：传入的对象 O。
- 使对象 O 不能增删属性，不能增删属性。

**区别：**
- Object.freeze ( O )：不能修改已有属性的特性字段（可枚举性、可配置性、可写性）和值。
- Object.seal ( O )：原先可写的属性，依旧可以改变其值。

```javascript
var obj = {a: 2, b: 3};
Object.getOwnPropertyDescriptors(obj);
// output:
// {
//   a: {value: 2, writable: true, configurable: true, enumerable: true},
//   b: {value: 3, writable: true, configurable: true, enumerable: true}
// }
Object.seal(obj);
obj.a = 5;
console.log(obj); // output: {a: 5, b: 3}
delete obj.a; // output: false

Object.freeze(obj);
obj.b = 6;
console.log(obj); // output: {a: 5, b: 3}
delete obj.a; // output: false
```

##### 公用关键函数 SetIntegrityLevel(O, level)
用于固定对象的自有属性集合。接受两个参数：
- 参数 O：是个 Object 类型值。
- 参数 level：值为 sealed，frozen 的枚举。
![SetIntegrityLevel.png](https://s1.ax1x.com/2022/06/27/jVaQNF.png)

**主要逻辑步骤：**
1. 若对象 O 不能被扩展，返回 false。
2. 获取 O 的自有属性集合 keys。
    - 若 level 值为 sealed，给 keys 中的每个元素 k 设置 k.[[Configurable]] = false;
    - 若 level 值为 frozen，对 keys 中的每个元素 k ，校验若 O.k 存在，
        - 若 O.k 是访问描述符，设置 k.[[Configurable]] = false;
        - 若 O.k 是数据描述符，设置 k.[[Configurable]] = false; k.[[Writable]] = false;
3. 属性的可枚举性、可配置性、可写性设置完毕，返回 true。

![freeze.png](https://s1.ax1x.com/2022/06/27/jVaZXq.png)
![seal.png](https://s1.ax1x.com/2022/06/27/jVaVcn.png)

##### Object.preventExtensions ( O )

作用：让对象 O 不可扩展，即不能添加自有属性。但不影响其原型对象添加属性。
返回值：不可扩展的源对象 O。

#### 校验对象状态 
- 方法 Object.isExtensible ( O ) 
- 方法 Object.isFrozen ( O ) 
- 方法 Object.isSealed ( O )

当 O 不是 Object 时，
- Object.isExtensible() 返回 false。
- Object.isFrozen(O) 返回 true。
- Object.isSealed(O) 返回 true。

若 `O.[[IsExtensible]]()` 为 true，
- Object.isExtensible() 返回 true。
- Object.isFrozen(O) 返回 false。
- Object.isSealed(O) 返回 false。

若 `O.[[IsExtensible]]()` 为 false，
- Object.isExtensible() 返回 false。
- Object.isFrozen(O) 和 Object.isSealed(O) 会获取 O 的每条自有属性的 [[Configurable]] 特性，若
    - [[Configurable]] 为 true，则返回 false；
    - 若是 frozen 校验，且属性是数据描述符，且 [[Writable]] 特性为 true，则返回 false。

若没有提前返回 false，则最终返回 true。



#### 与对象毫无关系的全局值比较方法 Object.is ( value1, value2 )

关键算法 SameValue(value1, value2)，逻辑如下：
![SameValue.png](https://s1.ax1x.com/2022/06/27/jVdMrt.png)


| 比较运算 | 强制类型转换 | +0、-0相等 | NaN、NaN 相等 |
| --- | --- | --- | --- |
| == | ✅ | ✅ | ❌ |
| === | ❌ | ✅ | ❌ |
| Object.is() | ❌ | ❌ | ✅ |

#### 对象原型相关属性及方法

- 属性 Object.prototype
- 方法 Object.getPrototypeOf ( O )
- 方法 Object.setPrototypeOf ( O, proto )

##### 属性 Object.prototype

Object.prototype 初始值是 Object 的原型对象属性（见20.1.3），其中
- 内部插槽 [[Prototype]] 为 null。
- 有普通对象定义的内部方法，除了 [[SetPrototypeOf]] 方法。
其属性特性是 
```javascript
{
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}
```
![prototype.png](https://s1.ax1x.com/2022/06/27/jVau7T.png)

##### 方法 Object.getPrototypeOf ( O )

作用：获取 O 的原型对象。

![getPrototypeOf.png](https://s1.ax1x.com/2022/06/27/jVaMAU.png)

> 1. 将 O 转成相应类型的对象。
> 2. 返回 O.[[Prototype]]。

10.1 章节中有 [[Prototype]] 的值为 null 或实例所继承的对象。

```javascript
var obj = {};
Object.getPrototypeOf(obj) === obj.__proto__; // true
```

##### 方法 Object.setPrototypeOf ( O, proto )
作用：指定 O 的原型对象为另一个对象 或 null。
> MDN 表示 ![MDN.png](https://s1.ax1x.com/2022/06/27/jVamn0.png)


![setPrototypeOf.png](https://s1.ax1x.com/2022/06/27/jVanBV.png)

1. 若 proto 不是 Object 或 Null，抛出 **TypeError** 异常。
2. 若 O 不是 Object, 返回 O。
3. 设置 O 的 prototype 为 proto。
4. 返回 O。
