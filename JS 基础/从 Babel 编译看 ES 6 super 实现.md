[Babel 在线编译工具](https://babeljs.io/repl/#?browsers=&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=MYGwhgzhAECC0G8BQ1rAPYDsIBcBOArsDungBQCUiKq0OAFgJYQB0Y0AvNAOT0CmIEOm4BuGgF8kkpKEgwAQtD4APHH0wATGPGSoM2fERLkqyAJAQCABz4mxZhsxYAjTjwDupEBtESpSIA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=true&fileSize=false&timeTravel=false&sourceType=module&lineWrap=false&presets=es2015%2Creact%2Cstage-2&prettier=false&targets=&version=7.14.6&externalPlugins=)

```javascript
class A {
  constructor() {
    this.a = 'hello';
  }
}

class B extends A {
  constructor() {
	super();
	this.b = 'world';
  }
}
```

Babel 编译结果：

![image](https://user-images.githubusercontent.com/31687804/122192997-b9252d80-cec6-11eb-8d75-3459df6dceb3.png)

第一步 _inherits

PS. class 本身就是函数

```javascript
function _inherits(subClass, superClass) {
  if (typeof superClass !== 'function' && superClass !== null) {     // 1. 父级必须是函数类型，并且不能为null；

    throw new TypeError('Super expression must either be null or a function');
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {     // 2. 将子级的 prototype 指向父级 prototype；
    constructor: { value: subClass, writable: true, configurable: true },     // 3. 将子级的 constructor 指向自身。
  });
  if (superClass) _setPrototypeOf(subClass, superClass);
}

```

第二步 _super 修饰子级 this

```javascript
function _createSuper(Derived) {
  var hasNativeReflectConstruct = _isNativeReflectConstruct();     // 1. 校验执行环境中是否有可用的 Reflect，[阮一峰讲过 Reflect](https://es6.ruanyifeng.com/#docs/reflect)
  return function _createSuperInternal() {
    var Super = _getPrototypeOf(Derived),     // 2. 获取到子级的原型对象, 即父级
      result;
    if (hasNativeReflectConstruct) {     // 3. Reflect 可用时，调用 Reflect.construct 创建构造函数
      var NewTarget = _getPrototypeOf(this).constructor;
      result = Reflect.construct(Super, arguments, NewTarget);
    } else {     // 4. Reflect 不可用时，调用 父级.apply 创建构造函数
      result = Super.apply(this, arguments);
    }
    return _possibleConstructorReturn(this, result);     // 5. result 是父级实例对象， 使用 call 方法用 result 修饰子类的 this 
  };
}

function _isNativeReflectConstruct() {     // 确保 Reflect.construct 能用
  if (typeof Reflect === 'undefined' || !Reflect.construct) return false;
  if (Reflect.construct.sham) return false;
  if (typeof Proxy === 'function') return true;
  try {
    Boolean.prototype.valueOf.call(Reflect.construct(Boolean, [], function() {}));
    return true;
  } catch (e) {
    return false;
  }
}
function _getPrototypeOf(o) {    // 做兼容性处理
  _getPrototypeOf = Object.setPrototypeOf
    ? Object.getPrototypeOf
    : function _getPrototypeOf(o) {
        return o.__proto__ || Object.getPrototypeOf(o);
      };
  return _getPrototypeOf(o);
}
function _possibleConstructorReturn(self, call) {     // 确保 var _super = _createSuper(B); 后面的方法中 _super 的方法可以被调用
  if (call && (_typeof(call) === 'object' || typeof call === 'function')) {
    return call;
  }
  return _assertThisInitialized(self);
}
function _assertThisInitialized(self) {
  if (self === void 0) {
    throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
  }
  return self;
}
```
第三步 子级添加自己的实例/静态属性和方法.

```javascript
  function B() {
    var _this;

    _classCallCheck(this, B);

    _this = _super.call(this);
    _this.b = 'world';
    return _this;
  }

  return B;
```
