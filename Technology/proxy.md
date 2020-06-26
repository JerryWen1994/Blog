# 浅谈 Proxy

![1_lc5wkELvaZgI9YfHKkZ1CQ](https://user-images.githubusercontent.com/20969011/62336659-ea2f2b80-b503-11e9-89b2-b55239a888c5.jpeg)

## 元编程

Proxy 用于修改某些操作的默认行为，所以属于一种“元编程”（meta programming）。
在这里介绍下什么是元编程？

meta-knowledge 就是「关于知识本身的知识」，meta-data 就是「关于数据的数据」，meta-language 就是「关于语言的语言」，而 meta-programming 也是由此而来，是「关于编程的编程」，也可以说是「自相关数据」、「自相关语言」、「自相关编程」，也就是编写把要执行的代码当文本进行操作的代码。

举个不严谨的例子：比如你要输出从 1 到 5 五个数字，用 python 的话就是

```tsx
print(1);
print(2);
print(3);
print(4);
print(5);
```

这就是正常编程，你还这以这么做

```tsx
for i in range(5):
    print('print(' + str(i + 1) + ')')
```

理解了元编程的概念后，就可以清楚了解到为什么 Proxy 被称为元编程了。

## 代理设计模式

![proxyFeaturedImage](https://user-images.githubusercontent.com/20969011/62016426-b6db5c80-b1e4-11e9-930e-48824faf15b0.png)

来源 [代理设计模式](https://dzone.com/articles/scala-proxy-design-pattern?source=post_page---------------------------)

> A proxy is a class functioning as an interface to something else.
>
> - Provide a surrogate or placeholder for another object to control access to it.
> - Use an extra level of indirection to support distributed, controlled, or intelligent access.
> - Add a wrapper and delegation to protect the real component from undue complexity.
>
> There are four common situations in which the Proxy pattern is applicable.
>
> 1. A virtual proxy is a placeholder for "expensive to create" objects. The real object is only created when a client first requests/accesses the object.
> 2. A remote proxy provides a local representative for an object that resides in a different address space. This is what the "stub" code in RPC and CORBA provides.
> 3. A protective proxy controls access to a sensitive master object. The "surrogate" object checks that the caller has the access permissions required prior to forwarding the request.
> 4. A smart proxy interposes additional actions when an object is accessed. Typical uses include:
>
> > - Counting the number of references to the real object so that it can be freed automatically when there are no more references (aka smart pointer)
> > - Loading a persistent object into memory when it's first referenced
> > - Checking that the real object is locked before it is accessed to ensure that no other object can change it.

代理是一类为其他东西提供接口的功能。

- 为另一个对象提供代理或占位符以控制对它的访问。
- 使用额外的间接级别来支持分布式，受控或智能访问。
- 添加一个包装器和委托以保护真实的组件免受过度的复杂影响。

代理模式有四种常见情况适用。

1. 虚拟代理是一个为了‘创建昂贵’对象的占位符。仅在客户端第一次请求/访问时才会创建真实对象。
2. 远程代理为驻留在不同地址空间的对象提供本地代表，这就是 RPC 和 CORBA 中提供了‘存根’代码。
3. 保护代理控制对敏感主对象的访问。代理对象在转发请求之前检查调用者是否具备所需的访问权限。
4. 当对象被访问时，智能代理会插入额外的操作。典型的例子包括：

> - 计算真实对象的引用数量，以便在没有更多引用时自动释放（又称智能指针）
> - 在第一次引用时将持久对象加载到内存中
> - 访问真实对象之前检查是否被锁，确保没有其他对象可以更改它

## 介绍 Proxy

Proxy 在目标对象之前架设一层“拦截”，外界对该对象的访问都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

```tsx
const obj = new Proxy(
  {},
  {
    get: function(target, key, receiver) {
      console.log(`getting ${key}!`);
      return Reflect.get(target, key, receiver);
    },
    set: function(target, key, value, receiver) {
      console.log(`setting ${key}!`);
      return Reflect.set(target, key, value, receiver);
    },
  }
);
```

上面的代码对一个空的对象进行了一层拦截，重新定义了该对象的读取和写入。
当对该对象属性进行赋值操作时，将会进行 set 函数；当对该对象属性进行读取操作时，将会执行 get 函数。
执行结果为：

```tsx
obj.count = 1;
// setting count!
++obj.count;
// getting count!
// setting count!
// 2
```

ES6 原生提供了 Proxy 构造函数，用来生成 Proxy 实例。
Proxy 有许多用途，它可以作为拦截器，代理器，运算符的重载，处理对象变化，甚至是 vue3 内部响应系统的动力。
使用 Proxy ，你可以将一只猫伪装成一只老虎。

```tsx
const proxy = new Proxy(target, handler);
```

每个 proxy 对象写法都与之相同，需要 new Proxy 来构造代理，必须要传入两个值，target 与 handler。target 代表所代理的对象，handler 所定义的对象是对 target 对象进行操作。
举个简单的例子：

```tsx
const target = {
  x: 10,
  y: 20,
};

const handler = {
  get: (obj, prop) => 42,
};

const obj = new Proxy(target, handler);

obj.x; // 42
obj.y; // 42
```

无论访问 obj 的任何属性操作，输出的都是 42，这包括`target.x`，`target['x']`，`Reflect.get(target, 'x')`等。

当然，proxy 代理不仅仅是 get 获取操作，还有十几种操作方法：

- handler.get
- handler.set
- handler.has
- handler.apply
- handler.construct
- handler.ownKeys
- handler.deleteProperty
- handler.defineProperty
- handler.isExtensible
- handler.preventExtensions
- handler.getPrototypeOf
- handler.setPrototypeOf
- handler.getOwnPropertyDescriptor

这里我就不一一介绍 proxy handler 方法了，阮一峰老师讲的更加的详细 [ECMAScript 6 入门-proxy](http://es6.ruanyifeng.com/#docs/proxy)

## 有趣的实现

例子来源 👉 [Click This](https://medium.com/dailyjs/how-to-use-javascript-proxies-for-fun-and-profit-365579d4a9f8)

### 🎁重新封装 api

通过实现一个代理，将有 http 请求时可以通过这个代理的 handler 来实现

![method](https://user-images.githubusercontent.com/20969011/64394721-181b0900-d08a-11e9-9836-017d59b2af3a.png)

```ts
const { METHODS } = require("http");
const api = new Proxy(
  {},
  {
    get(target, propKey) {
      const method = METHODS.find(method =>
        propKey.startsWith(method.toLowerCase())
      );
      if (!method) return;
      const path =
        "/" +
        propKey
          .substring(method.length)
          .replace(/([a-z])([A-Z])/g, "$1/$2")
          .replace(/\$/g, "/$/")
          .toLowerCase();
      return (...args) => {
        const finalPath = path.replace(/\$/g, () => args.shift());
        const queryOrBody = args.shift() || {};
        // You could use fetch here
        // return fetch(finalPath, { method, body: queryOrBody })
        console.log(method, finalPath, queryOrBody);
      };
    },
  }
);

api.get(); //  GET /
api.getUsers(); //  GET /users
api.getUsers$Likes("1234", { page: 2 }); //  GET /users/1234/likes?page=2
api.foobar(); // return
```

### 🔍数据查询方法代理

可以使用代理实现一个数组的包装，通过解析其方法名去调用并执行该方法

![search](https://user-images.githubusercontent.com/20969011/64394871-b5763d00-d08a-11e9-87a0-50c88baf6d7f.png)

```ts
const camelcase = require("camelcase");
const prefix = "findWhere";
const actions = {
  Equals: (object, value) => object === value,
  IsNull: (object, value) => object === null,
  IsUndefined: (object, value) => object === undefined,
  IsEmpty: (object, value) => object.length === 0,
  Includes: (object, value) => object.includes(value),
  IsLowerThan: (object, value) => object === value,
  IsGreaterThan: (object, value) => object === value,
};
const actionNames = Object.keys(actions);
const wrap = arr => {
  return new Proxy(arr, {
    get(target, propKey) {
      if (propKey in target) return target[propKey];
      const actionName = actionNames.find(action => propKey.endsWith(action));
      if (propKey.startsWith(prefix)) {
        const field = camelcase(
          propKey.substring(prefix.length, propKey.length - actionName.length)
        );
        const action = actions[actionName];
        return value => {
          return target.find(item => action(item[field], value));
        };
      }
    },
  });
};
const arr = wrap([
  { name: "John", age: 23, skills: ["mongodb"] },
  { name: "Lily", age: 21, skills: ["redis"] },
  { name: "Iris", age: 43, skills: ["python", "javascript"] },
]);
console.log(arr.findWhereNameEquals("Lily")); // finds Lily
console.log(arr.findWhereSkillsIncludes("javascript")); // finds Iris
```

## Mobx Observable

![data](https://user-images.githubusercontent.com/20969011/62336675-fe732880-b503-11e9-9107-4f9cfd7f80b2.png)

精细颗粒度的组件绑定上数据，通过 Mobx 数据的 getter 和 setter，在组件渲染的时候会触发 getter ，然后把这个组件对应的 Watcher 添加到 getter 相关的数据的依赖中，然后当数据发生变化的时候，相对应的 Watcher 会去重绘组件，精确地知道哪些组件需要重新绘制

Observable 值可以是 JS 基本数据类型、引用类型、普通对象、类实例、数组和映射

1. 如果 **value** 是 ES6 的 Map: 会返回一个新的 `Observable Map`
1. 如果 **value** 是数组：会返回一个 `Observable Array`
1. 如果 **value** 是 ES6 的 Set：会返回一个新的 `Observable Set`
1. 如果 **value** 是没有原型的对象，那么对象会被克隆并且所有属性都会转换成可观察的对象
1. 如果 **value** 是有原型的对象，JavaSript 原始数据类型或者函数，`observable` 会抛出 `提供的值无法转换为可观察值。如果想要为这样的值创建一个独立的可观察引用，请使用'observable.box（value）'`

```tsx
function createObservable(v: any, arg2?: any, arg3?: any) {
  if (isObservable(v)) return v;

  const res = isPlainObject(v)
    ? observable.object(v, arg2, arg3)
    : Array.isArray(v)
    ? observable.array(v, arg2)
    : isES6Map(v)
    ? observable.map(v, arg2)
    : isES6Set(v)
    ? observable.set(v, arg2)
    : v;

  if (res !== v) return res;

  fail(
    process.env.NODE_ENV !== "production" &&
      `The provided value could not be converted into an observable. If you want just create an observable reference to the object use 'observable.box(value)'`
  );
}
```

### Observable Box

observable.box 方法简单地返回一个 ObservableValue 实例。

```tsx
box<T = any>(value?: T, options?: CreateObservableOptions): IObservableValue<T> {
  if (arguments.length > 2) incorrectlyUsedAsDecorator("box")
  const o = asCreateObservableOptions(options)
  return new ObservableValue(value, getEnhancerFromOptions(o), o.name, true, o.equals)
}
```

ObservableValue 实现了 get 和 set 方法，用户在使用时自行使用这两个方法获取和设置「可观察原始值」的值，而在实现 **可观察** ，就是在这两个方法中使用事件报告。

观察到一个值的变化后，会执行 set 方法：

1. 将新值进行预处理：其中包括 `checkIfStateModificationsAreAllowed` 判断边界情况，是否存在并允许进行修改； `hasInterceptors` && `interceptChange` 判断是否有拦截器对该值进行转换； `enhancer` 将新值转换成可观察对象

1. 如果允许开启报告，则会开始通信报告：准备更换新值，再进行 `setNewValue` 处理

1. 如果开启了报告，最后会执行报告结束

整个过程如下：

![Untitled Diagram](https://user-images.githubusercontent.com/20969011/62989249-d24b9600-be79-11e9-86af-4a50f7189f0e.png)

### Observable Object

observable.object 方法，是把一个普通的 JavaScript 对象的所有属性都将被拷贝至一个克隆对象并将克隆对象转变成可观察的，而且 observable 是 **递归应用** 的，这样对象中每个属性都是可观察的。

```ts
object<T = any>(
  props: T,
  decorators?: { [K in keyof T]: Function },
  options?: CreateObservableOptions
): T & IObservableObject {
  if (typeof arguments[1] === "string") incorrectlyUsedAsDecorator("object")
  const o = asCreateObservableOptions(options)
  if (o.proxy === false) {
    return extendObservable({}, props, decorators, o) as any
  } else {
    const defaultDecorator = getDefaultDecoratorFromObjectOptions(o)
    const base = extendObservable({}, undefined, undefined, o) as any
    const proxy = createDynamicObservableObject(base)
    extendObservableObjectWithProperties(proxy, props, decorators, defaultDecorator)
    return proxy
  }
}
```

从上面代码可以看出，可以手动设置代理。当执行 `asCreateObservableOptions(options)` ，如果没有传入 `options`， o 会赋值为以下对象，所以默认可观察对象都会设置为代理对象。

```ts
export const defaultCreateObservableOptions: CreateObservableOptions = {
  deep: true,
  name: undefined,
  defaultDecorator: undefined,
  proxy: true,
};
```

当设置 `proxy = false` ，只是将普通对象转换成可观察对象，`observable.object(object)` 实际上是 `extendObservable({}, object)` 的别名。

`extendObservable` 方法的第一个参数是 {} 可以看到，最终产生的观察值对象是基于全新的对象，不影响原始传入的对象内容。 再将生成的对象 `base` 传入 `createDynamicObservableObject` 方法，生成 `proxy` 对象。

在 `createDynamicObservableObject` 方法中可以看到，生成 `Proxy` 对象传入两个参数，第一个参数是 `target`：所代理的目标对象，第二个参数是 `handler`：对代理对象的操作处理。

## 总结

Proxy 不仅可以代理对象，也可以代理数组；还可以代理动态增加的属性如 type 。这也是 Object.defineProperty 做不到的。它们增加了一些开销，但另一方面，它能够在运行时通过名称动态实现方法，使代码超级优雅和可读。

在使用 mobx 5 时，相对与 mobx 4 有两个优点：

1. mobx 现在可以检测普通可观察对象上的属性添加，可以使用普通的可观察对象作为动态集合。
2. 可观察数组现在被所有第三方库识别为数组，这将避免对它们进行切片。

但使用 mobx 5，也存在系统要求的提升：

1. mobx 5 只能在支持的环境中使用 Proxy 。
2. 由于 mobx 5 在某种原因下，项目无法满足要求，还是需要降级使用 mobx 4。
3. 在性能方面，mobx 5 与 mobx 4 非常相似，但也做出了改进。

proxy 在 mobx 5 的运用也在慢慢变化，采用 proxy 来实现 Observable object ，只是在细节方面将 Object.defineProperty 替换成 new Proxy 的写法， observable.object 方法已经改用 [createDynamicObservableObject](https://github.com/mobxjs/mobx/blob/master/src/types/dynamicobject.ts#L74) 来创建 proxy， 所创建的 proxy 模型来自于 [objectProxyTraps](https://github.com/mobxjs/mobx/blob/master/src/types/dynamicobject.ts#L21) 方法

今天分享的知识有点浅，也不敢班门弄斧，主要还是笔者的知识点不够，还需再继续努力。

如有错误，还望不吝赐教。
