# EventEmitter

EventEmitter 本质上是一个观察者模式的实现， 它的核心就是事件触发与事件监听器功能的封装

## api 介绍

在 EventEmitter 中所实现的 api 有:

- on(event, handler): 为指定事件注册一个监听器，其接受一个事件名称及回调函数
- off(event, handler): 将指定的 event 监听器移除，其接受一个事件名称及回调函数
- emit(event, ...args): 执行指定的 event 监听器，其接受一个事件名称及传入监听器的其他参数
- once(event, handler): 和 on 类似，但只触发一次，随后便解除事件监听
- addListener(event, handler): on 方法别名
- removeListener(event, handler): off 方法别名
- removeAllListeners([event]): 移除指定事件的所有监听回调，如果不传 event，则清空所有监听器
- setMaxListeners(n): 用于设置监听器的最大限制的数量。（默认监听器中含有 10 个回调将产生警告）
- listeners(event): 返回指定事件的监听器数组

## 实现代码

```ts
type Handler = (...args) => void;

class EventEmitter {
  private handlers: { [key: string]: Handler[] } = {};
  private maxLength: number = 10; // 默认最大监听器数量

  public on = (eventName: string, handler: Handler) => {
    if (!this.handlers[eventName]) {
      this.handlers[eventName] = [];
    }
    if (this.handlers[eventName].length >= this.maxLength) {
      throw new Error(`添加失败, 监听器数量将超过 ${this.maxLength} 个`);
    } else {
      this.handlers[eventName].push(handler);
    }
  };

  public off = (eventName: string, handler: Handler) => {
    if (this.handlers[eventName]) {
      const idx = this.handlers[eventName].indexOf(handler);
      if (idx !== -1) {
        this.handlers[eventName].splice(idx, 1);
        return true;
      }
    }
    return false;
  };

  public removeAllListeners = (eventName?: string) => {
    if (eventName && this.handlers[eventName]) {
      delete this.handlers[eventName];
    } else {
      this.handlers = {};
    }
  };

  public addListener = (eventName: string, handler: Handler) => {
    this.on(eventName, handler);
  };

  public once = (eventName: string, handler: Handler) => {
    const wrapHandler = (...args) => {
      handler(...args);
      this.off(eventName, wrapHandler);
    };
    this.on(eventName, wrapHandler);
  };

  public removeListener = (eventName: string, handler: Handler) => {
    this.off(eventName, handler);
  };

  public emit = (eventName: string, ...args) => {
    if (this.handlers[eventName]) {
      this.handlers[eventName].forEach((handler) => {
        handler(...args);
      });
    }
  };

  public listeners = (eventName: string) => {
    return this.handlers[eventName];
  };

  public setMaxListeners = (max: number) => {
    this.maxLength = max;
  };
}
```
