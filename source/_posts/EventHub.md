---
title: 手把手教你写EventHub（发布-订阅）
top: true
tags: eventHub
categories: 手写系列
comments: true
abbrlink: ee1ff54b
date: 2021-10-12 16:51:34
---

> 手写 EventBus 作为经典的前端手写面试题，在工作之余我觉得还是得记录一下 😊
> **什么是 EventHub**

EventHub 是基于**发布订阅模式**实现的一个实例, 翻译为事件中心,常用于多模块(组件)间的通信。

今天我们就简单的实现 EventHub 的`发布、订阅和取消订阅`：
首先我们确定`Api`:

- on 订阅（监听）
- emit 发布（触发）
- off 取消订阅

**实现发布和订阅**

我们得用一个变量**eventMap**来暂存事件名字（key）和事件(value)，这里我们用一个数组来记录事件（因为考虑多次订阅）：

```js
class EventHub {
  private eventMap: { [key: string]: Array<(params?) => void> } = {};
  on(eventName, fn) {}
  emit(eventName) {}
  off(eventName) {}
}
export default EventHub;
```

当我们订阅一个事件的时候，我们把这个事件 push 到订阅的东西的事件队列中，比如我在手机闹钟里面订了个 3 秒后的闹钟:

```js
import EventHub from "../src/index";

const eventHub = new EventHub();

eventHub.on("闹钟", () => {
  setTimeout(() => {
    console.log("闹钟响了");
  }, 3000);
});

eventHub.emit("闹钟");
```

然后我们在类里面的`on`函数把事件 push 进去：

```js
class EventHub {
  private eventMap: { [key: string]: Array<(params?：any) => void> } = {};
  on(eventName: string, fn: (params?: any) => void) {
    this.eventMap[eventName] = this.eventMap[eventName] || [];
    this.eventMap[eventName].push(fn);
  }
  emit(eventName) {}
  off(eventName) {}
}

export default EventHub;
```

我们在`emit`触发执行逻辑：

```js
class EventHub {
  private eventMap: { [key: string]: Array<(params?: any) => void> } = {};
  on(eventName: string, fn: (params?: any) => void) {
    this.eventMap[eventName] = this.eventMap[eventName] || [];
    this.eventMap[eventName].push(fn);
  }
  emit(eventName: string) {
    (this.eventMap[eventName] || []).forEach((fn) => fn());
  }
  off(eventName: string) {
    console.log(eventName);
  }
}

export default EventHub;
```

执行触发的代码，看控制台（提示：node 运行 ts 文件请下载 ts-node）

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7adeff31547c4b26b739a077107dd40e~tplv-k3u1fbpfcp-watermark.image?)
**实现取消订阅：**
当我们在 emit 之前 off 闹钟的订阅

```js
import EventHub from "../src/index";

const eventHub = new EventHub();

const fn = () => {
  setTimeout(() => {
    console.log("闹钟响了");
  }, 3000);
};
eventHub.on("闹钟", fn);
eventHub.off("闹钟", fn);
eventHub.emit("闹钟");
```

加上取消订阅的逻辑：

```js
 off(eventName: string, fn: (params?: any) => void) {
    const index = this.eventMap[eventName].findIndex((item) => item === fn);
    if (index === -1) return;
    this.eventMap[eventName].splice(index, 1);
    console.log("取消闹钟成功");
  }
```

执行取消成功：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cece88b0c35949d7b82db8fca752b106~tplv-k3u1fbpfcp-watermark.image?)
[源代码点击](https://github.com/leehome150/EventHub)
