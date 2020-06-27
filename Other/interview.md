# 面试题库

- [1. Map、Set 和 WeakMap、WeakSet 有什么区别？Weak 的使用场景是什么？](#1)
- [2. 一直 Pending 的 Promise 会不会导致内存泄漏？](#2)
- [3. 在 Javascript 中什么情况下会进行装箱/拆箱转换？](#3)
- [4. 如何区分 null 和 undefined，我应该怎么使用？](#4)
- [5. new 操作符做了什么？模仿实现一个 create 函数](#5)
- [6. JavaScript 严格模式下有哪些不同？ ES6 还需要严格模式吗？](#6)

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

<h3 id="2">2. 一直 Pending 的 Promise 会不会导致内存泄漏？</h3>

<h3 id="3">3. 在 Javascript 中什么情况下会进行装箱/拆箱转换？</h3>

<h3 id="4">4. 如何区分 null 和 undefined，我应该怎么使用？</h3>

<h3 id="5">5. new 操作符做了什么？模仿实现一个 create 函数</h3>

<h3 id="6">6. JavaScript 严格模式下有哪些不同？ ES6 还需要严格模式吗？</h3>
