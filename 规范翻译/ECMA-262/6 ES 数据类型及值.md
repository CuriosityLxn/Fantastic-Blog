[原文](https://tc39.es/ecma262/#sec-ecmascript-data-types-and-values)
ES 规范中的数据类型有两大类：
- ES 语言类型（基本数据类型）
- ES 规范类型

## ES 语言类型（基本数据类型）

- Undefined
- Null
- Boolean
- String
- Symbol
- Number
- BigInt
- Object

#### String 类型

String 类型是
- 由16位无符号整型（“元素”）组成
```math
0 <= length <= 2^{53} - 1
```

的有序序列，通常用于表达文本数据。

String 的每个元素
- 都被视为 UTF-16 代码单元值，
- 都占据了序列中的一个位置。
- 这些位置用非负整数索引，从0开始。

String 的长度
- 是元素的数量。
- 空字串的长度为0，因为不包含任何元素。

规范化 String 值的函数：
- String.prototype.normalize 显式规范化
- String.prototype.localeCompare 隐式规范化

##### StringIndexOf ( string, searchValue, fromIndex )

| 参数名 | 参数类型 |
| --- | --- |
| string | String |
| searchValue | String |
| fromIndex | 非负整数 |

执行步骤：
1. 令 len 是 string 的长度；
2. 若 searchValue 是空字串，且 fromIndex ≤ len， 则 return fromIndex。
3. 令 searchLen 是 searchValue 的长度；
4. 
```javascript
for (let i = fromIndex; i <= len - searchLen; i++) {
  let candidate = string[i] 至 string[i+searchLen] 的子串;
  if (candidate === searchValue) return i;
}
```
5. return -1;

⚠️ 1. 若 `searchValue` 是空字串，且 `fromIndex` <= `string`.lengh，则该算法返回 `fromIndex`。空字串会在字符串的每个位置被有效的找到，包括最后一个编码单元之后。
⚠️ 2. 若 `fromIndex` > `string`，则该算法一定返回 -1。

#### Symbol 类型

Symbol 是所有可用做对象属性 key 的所有非字符串值的集合。

Symbol 的值是唯一且不可改变的。

Symbol 的值不可改变的保留一个称为`[[Description]]`的关联值，这个值是 undefined 或者一个字符串。

##### 常见 symbols

常见 symbol 是 Symbol 的内置值，通常用作属性的 key，这些 key 的值用于规范扩展。除非另有说明，这些 Well-known symbol 的值由所有域（Realm）共享。

该规范使用 @@name 形式引用 Well-known symbol，下表列出所有 name：

![symbols.png](https://i.postimg.cc/C1N1Nt1V/D0-A4378-B-99-B3-4800-A7-B9-369-C69-ACA0-D7.png)

#### 数字类型（Number、BigInt）

数字类型进行转换时通常会丢失精度或者被截断，所以 ES 不提供这些类型的隐式转换。开发者需显式调用 Number()方法和 BigInt()来进行类型转换。

有些操作符只能操作数字类型的值，并返回操作结果：
- 一元负值 -
- 指数运算符 **
- 乘法/除法/取余 * / %
- 加法/自增 +｜x++｜++x
- 减法/自减 -｜x--｜--x
- 左移/算术右移/无符号右移 << | >> | >>>
- 比较和相等：>|>=|<|<=|==|!=|===|!==
- 按位非 ~
- 按位与 &
- 按位或 ｜
- 按位异或 ^

⚠️：ES 初版及后面几版为某些操作符做了可能会引起精度丢失和截断的隐式数字转换，但没有为 BigInt 做。

##### Number 类型

包含区间
```math
[2^{64}, 2^{53} + 3]
```
中的 64 位二进制浮点数字，除了唯一的 NaN。
NaN 是 Number 类型的“非数字”，值为
```math
2^{53} - 2
```
还有两个特殊值：正无穷和负无穷（+Infinity，-Infinity）。

##### BigInt 类型

整数值，没有大小限制，不被特定位宽（bit-width）限制。

#### Object 类型

逻辑上 Object 是属性的集合。每个属性是数据属性或者访问器属性：
![img.png](https://i.postimg.cc/q72gtgHq/6561-C9-D8-091-E-4-D1-A-9144-C598-D72-B4-B31.png)
![img.png](https://i.postimg.cc/vTZmSzgx/97-B5-FA55-C7-B7-49-C6-9-D22-91-B6-E33-EC888.png)


属性的 key 值是 String 或者 Symbol ，包括空字串，值可以是任意类型的值。

![img.png](https://i.postimg.cc/GhpBMcGg/9831-F3-EE-0338-44-C5-AA74-F9-CC044-B9097.png)

- **整形索引**作为 key 时，是一个规范的[数字字符串](https://tc39.es/ecma262/#sec-canonicalnumericindexstring)，范围是
+0 或 小于等于
```math
2^{53} - 1
```
的整型。
- **数组索引**是数值范围
```math
[+0, 2^{32} - 1)
```
的整形索引。

##### Object 内置方法和内置插槽

- **内置方法是多态的**。对不同的对象值调用相同的公共方法时，可能执行不同的算法。调用内部方法的实际对象是调用的“目标”。在运行时，对象不支持被调用的内部方法则会会抛出 TypeError 异常。
- **内部插槽表示对象的内部状态**，可以被 ES 规范中的许多算法使用。
    - 不是对象属性，不能被继承。
    - 其值可以是任何基本类型的值。
    - 作为创建对象过程中的一部分被分配，不能被动态添加进对象。除非显式指定。
    - 值默认是 **undefined**，除非另行说明。通过规范提供的算法创建出的对象都有内部插槽。

所有对象都有内部插槽 `[[PrivateElements]]`，值是私有元素列表，包括 private field 的值、方法、对象的访问器。初始化为空列表。

1. 普通对象要满足如下标准：
- 有下表中的基本内置方法。
![func.png](https://i.postimg.cc/XqsrVhSg/7-A967-E58-5522-4669-AF55-BD716727-B3-D2.png)
- 如果是函数对象，还需要有内置方法`[[Call]]`和`[[Construct]]`定义的。
![call.png](https://i.postimg.cc/htkfPTwN/49832727-546-D-40-EA-9-F2-C-6-E091-E2-D3-BAE.png)
![construct.png](https://i.postimg.cc/sgQMrwnp/8-D113-FB4-59-D6-4179-ABA7-A13-EF0516543.png)

2. 怪异对象：非普通对象。常见的怪异对象有
    - 数组怪异对象，即数组对象
    - 绑定函数怪异对象，它包装了原函数对象，调用绑定函数通常会导致执行包装函数。bind() 函数就是绑定函数怪异对象，具有以下内部属性：
    ![img1.png](https://i.postimg.cc/tgzk3qhR/0130146-B-9-ADA-48-CF-9871-4-B8-D9-EA9-E4-CA.png)

下表总结了可作为函数调用的对象支持的额外的内置方法，
- 函数对象需要有内置方法 `[[Call]]`
- 构造函数需要有内置方法 `[[Construct]]`
- 每个有 `[[Construct]]` 的对象必须有 `[[Call]]`
- 构造函数必须是函数对象
因此，构造函数被称为 「constructor 函数」 「constructor 函数对象」。
![img2.png](https://i.postimg.cc/N0tb292P/5-C498581-B896-4-C87-8995-59085-E74729-C.png)

## ES 规范类型
ES 规范类型指算法中用于描述 ES 语言构造和 ES 语言类型语义的元值，不一定对应实体。
ES 规范类型可以用来描述 ES 表达式求值的中间结果，但这些值不能存储为对象的属性或 ES 语言变量的值（即不可被开发者通过 API 访问到）。

- Reference
- List
- Completion
- Property Descriptor
- Environment Record
- Abstract Closure
- Data Block

#### List 和 Record 规范类型

List 类型用于解释
- new 表达式中的参数列表
- 函数调用中的参数列表
- 其他算法中所需的简单有序的值列表
列表是有序的、任意长度的，列表元素使用 0 为原点的索引访问。

Record 类型用于描述该规范算法中的数据集合。
Record 类型的值由1个或多个已命名的 field 组成。
field 的值要么是 ES 值或一个Record 类型的抽象值。
field 的名称被双方括号包围，比如 `[[Value]]`。

该规范中为了便于标注，可以使用类似于对象字面量的语法来表示 Record 值。比如 `{ [[Field1]]: 42, [[Field2]]: false, [[Field3]]: empty }`。

#### Set 和 Relation 规范类型

Set 类型用于解释在内存模型中使用的无需集合。
Set 类型的值是不重复的简单元素集合。
Set 中可以增删元素。Set 可以互相并集、相交、相减。

Relation 类型用于解释 Set 中的约束。
Relation 类型的值是来自其值域有序值对的集合。例如，时间的关联是一组有序的事件对。

#### Completion Record 规范类型

Completion 类型是一种 Record，用于解释运行时值的传播和控制流，比如语句的行为（break, continue, return, throw）。

Completion 类型的值被称为 Completion Records，是具有下表 field 的 Record 值。
![Completion-Record.png](https://i.postimg.cc/mZjzNczF/9-E536-BC7-9-DAF-442-B-BB56-EA9-E07-CE42-F3.png)

“意外结束”：`[[Type]]`值为 normal 以外的其他值。
可调用的对象只会返回正常结束和抛出结束。

#### Environment Record 规范类型

用于解释嵌套函数和块中的名称解析行为。详见 9.1。

