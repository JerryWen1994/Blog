# 面试题库

- [1. Map、Set 和 WeakMap、WeakSet 有什么区别？Weak 的使用场景是什么？](#1)
- [2. 一直 Pending 的 Promise 会不会导致内存泄漏？](#2)
- [3. 在 Javascript 中什么情况下会进行装箱/拆箱转换？](#3)
- [4. 如何区分 null 和 undefined，我应该怎么使用？](#4)
- [5. new 操作符做了什么？模仿实现一个 create 函数](#5)
- [6. JavaScript 严格模式下有哪些不同？ ES6 还需要严格模式吗？](#6)
- [7. 模仿实现一下 ES6 的 class 继承。支持静态成员、super 等等](#7)

<br>
<br>
<br>

## 题解

<h3 id="1">1. Map、Set 和 WeakMap、WeakSet 有什么区别？Weak 的使用场景是什么？</h3>

Set 类似于数组，成员值不能重复，可以遍历，含有 keys(), values(), entries(), forEach() 方法

WeakSet 的成员只能是对象，对象都是弱引用，因此它无法遍历，不含有 keys(), values(), entries(), forEach() 方法

Map 类似于集合，可以把各种类型的值(包括对象)当做 key，可以遍历

WeakMap 只接受对象作为键名(null 除外)，无法遍历，键名所指向的对象，不计入垃圾回收机制

使用场景：

- 订阅者引用。我们通常使用对象来保存订阅者，当订阅者不再使用时，由订阅者手动释放。但是如果忘记释放就会导致内存泄漏。
- 实现缓存。没有被使用的对象自动释放。
- 保存其他同样也是 Weak 的资源。比如保存 DOM 对象，这些 DOM 对象随时可能被删除。

<br>
<br>

<h3 id="2">2. 一直 Pending 的 Promise 会不会导致内存泄漏？</h3>

<br>
<br>

<h3 id="3">3. 在 Javascript 中什么情况下会进行装箱/拆箱转换？</h3>

<br>
<br>

<h3 id="4">4. 如何区分 null 和 undefined，我应该怎么使用？</h3>

<br>
<br>

<h3 id="5">5. new 操作符做了什么？模仿实现一个 create 函数</h3>

new 操作符用于创建一个用户自定义对象或包含构造函数的内置对象。new 操作符只能作用在函数表达式上(括号可以省略)。

具体过程如下：

- 创建一个空对象，尚且称为 self。
- 连接 self 对象的原型对象到 构造函数(例如 Foo) 的 prototype(可以认为是原型模板)
- 以 self 作为 this (上下文) 调用构造函数 Foo
- 神奇的(不合理的)一步。`如果构造函数返回一个对象(非原始类型、null、undefined)，这个对象将作为 new 操作符的结果`。正常情况下 构造函数都是没有返回值的。

上面第四步是 JavaScript 中设计的不合理的地方，其他编程语言构造函数都是不能显式返回值的:

```jsx
function Foo() {
  this.bar = 1;

  return {}
}

Foo.prototype.baz = 2

const a = new Foo()
console.log(a.bar) // undefined
console.log(a.baz) // undefined
```

在 ES6 的 class 中这种行为还是被允许的。不过你在 Typescript 下面使用 class 构造函数返回类型不兼容的值会报错。

另外在 ES6 之后的标准中，我们可以通过 `new.target` 来判断当前函数是否以 new 操作符的函数调用的。如果你的函数只能以构造函数的形式调用，可以抛出异常。

```jsx
function create(Ctor, ...args) {
    const self = {}
    Object.setPrototypeOf(self, Ctor.prototype)
    const rtn = Ctor.apply(self, args)
    return typeof rtn === 'object' && rtn != null ? rtn : self
}
```

<br>
<br>

<h3 id="6">6. JavaScript 严格模式下有哪些不同？ ES6 还需要严格模式吗？</h3>

ES6 的模块自动采用严格模式

1. 禁止使用未声明的全局变量
2. 禁止使用 with 语句
3. 除了全局作用域和函数作用域，严格模式创设了第三种作用域：eval 作用域，变量作用范围只在 eval 函数内
4. 禁止this指向全局对象
5. 禁止在函数内部遍历调用栈
6. 禁止删除变量，只有在configurable设置为true的对象属性时，才能被删除
7. 禁止对只读属性进行赋值
8. 禁止对getter方法读取的属性进行赋值
9. 对禁止拓展的对象添加新属性时报错
10. 禁止删除不可删除的属性
11. 对象不能有重名属性
12. 函数不能有重名参数
13. 禁止八进制表示法
14. 不允许对arguments进行赋值
15. arguments不再追踪参数变化
16. 禁止使用arguments.callee，无法在匿名函数内部调用自身
17. 函数声明必须在顶层
18. 新增保留字：implements, interface, let, package, private, protected, public, static, yield

<br>
<br>

<h3 id="7">7. 模仿实现一下 ES6 的 class 继承。支持静态成员、super 等等</h3>

使用 ES5 来实现类的继承：通过原型或原型链的赋值，与 call/apply 方法混用

```js
var __extends = (function() {
  var extendStatics = function(d, b) {
    extendStatics = Object.setPrototypeOf ||
      ({ __proto__: [] }
        instanceof Array && function(d, b) { d.__proto__ = b; }) ||
      function(d, b) {
        for (var p in b)
          if (b.hasOwnProperty(p)) d[p] = b[p];
      };
    return extendStatics(d, b);
  }
  return function(d, b) {
    extendStatics(d, b);

    function __() { this.constructor = d; }
    d.prototype = b === null ? Object.create(b) : (__.prototype = b.prototype, new __());
  };
})();

var Animal = (function() {
  function Animal(name) {
    this.name = name;
  }
  Animal.prototype.move = function(num) {
    if (num == void 0) { num = 0; }
    console.log("Animal move " + num + ' m.');
  };
  return Animal
}());

var Snake = (function(_super) {
  __extends(Snake, _super);

  function Snake(name) {
    return _super.call(this, name) || this;
  }
  Snake.prototype.move = function(num) {
    if (num == void 0) { num = 5; }
    console.log('Slithering...');
    _super.prototype.move.call(this, num);
  };
  return Snake
}(Animal));

var a = new Snake('test');
```
