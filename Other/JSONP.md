# JSONP

## 同源策略

同源协议指的是页面的域名、协议、端口号等都相同。同源策略就是为了保证用户的信息安全，禁止恶意网站窃取数据。在实际开发中会经常遇到需要解决同源策略的问题，这类问题我们称之为跨域，即不同域名之间的数据交互。

解决跨域问题的方法有很多种，JSONP 就是其中之一。

## 原理

JSONP 全称是 JSON with Padding, 在服务器端收集 script 标签，并将返回参数以 json 格式返回至客户端，相当于是服务器调用客户端接口。

其原理就是利用 script 标签的 src 属性调用资源不跨域的特性。客户端向服务器端发起一个携带了 callback name 回调名称的请求，并将该回调函数注入全局方法。在服务器端接收请求后，将返回数据构造成 json 对象作为传参，并以 callback name 作为函数名进行调用。

## 优缺点

优点：兼容性好，实现比较简单
缺点：只能发送 get 请求，响应失败没有状态码， 数据容易被劫持

## 实现

```ts
export default function jsonp(url, option = { timeout: 3000 }) {
  const removeCode = (fn) => {
    document.body.removeChild(document.getElementById(fn));
    delete window[fn];
  };
  let timer;
  return new Promise((resolve, reject) => {
    const funcName = `jsonpcallback_${Date.now()}_${Math.random() * 10000}`;
    const src = new URL(url);
    src.searchParams.append("callback", funcName);
    window[funcName] = (res) => {
      resolve(res);
      timer = setTimeout(() => {
        removeCode(funcName);
      }, option.timeout);
    };
    const script = document.createElement("script");
    script.id = funcName;
    script.src = src.href;
    script.type = "text/javascript";
    script.onerror = () => {
      reject(new Error(`fetch ${url} error`));
      removeCode(funcName);
      if (timer) {
        clearTimeout(timer);
      }
    };
    document.body.appendChild(script);
  });
}
```
