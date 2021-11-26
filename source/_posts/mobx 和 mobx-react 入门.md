---
title: mobx 和 mobx-react 入门
tags: React
top: true
abbrlink: e13966ac
date: 2021-11-25 16:18:15
---

## 1.为什么需要状态管理？

### 组件内

- class 组件 state
- 函数组件 useState

### 组件间

- 父子之间通信用 props 传递
- 爷孙之间通信传递两次 props
- 但是任意组件？

这个时候我们就需要用到 mobx 和 mobx-react（这里不讲 redux）

## 2.mobx

### 核心概念

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/7/173270ed17dabb91~tplv-t2oaga2asx-image.image)

### 文档

[mobx 中文文档](https://cn.mobx.js.org/)

### 基本用法

- State

```
import { observable, computed, action, autorun } from "mobx";
//State，被观察者
const todos = (observable([
 {
 title: "起床",
 completed: false
 },
 {
 title: "穿⾐",
 completed: false
 },
 {
 title: "洗漱",
 completed: false
 }
]));
```

- Computed

```
//Computed values ,由 State 的更新触发
let uncompletedCount = computed(
() => todos.filter(todo => !todo.completed).length
);
```

- Reactions

```
//Reactions， 由 State 和 Computed Values 的改变触发执⾏
autorun(() => {
console.log(
`剩余任务:${uncompletedCount}`,
todos
.filter(todo => !todo.completed)
.map(todo => todo.title)
.join(", ")
);
});
```

- Actions

```
const doTask = action(() => {
todos.find(todo => !todo.completed).completed = true
});
doTask()
```

### 装饰器

**什么是装饰器 Decorators**

- 核⼼思想：保留原来的名字和主功能，添加⼀
  些附庸功能
- ⼀种设计模式，⽤来“装饰”⼀个函数或对象
- ES7 新语法，⽤来装饰类或者属性

@observable 可以在实例字段和属性 getter 上使用。 对于对象的哪部分需要成为可观察的，@observable 提供了细粒度的控制。

```
import { observable, computed } from "mobx";

class OrderLine {
    @observable price = 0;
    @observable amount = 1;

    @computed get total() {
        return this.price * this.amount;
    }
}
```

### mobx 的装饰器写法

```
import { observable, autorun, computed, action } from 'mobx';
class Todo {
 @observable todos = []
 constructor() {
 autorun(() => {
 console.log(
 `剩余任务:${this.uncompletedCount}`,
 this.todos
 .filter(todo => !todo.completed)
 .map(todo => todo.title)
 .join(", ")
 );
 });
 }
 @computed get uncompletedCount() {
 return this.todos.filter(todo => !todo.completed).length;
 }
 @action addTodo(title) {
 this.todos.push({
 title: title,
 completed: false
 });
 }
 @action doTask(){
 this.todos.find(todo => !todo.completed).completed = true
 }
}

```

### 核心思想

状态的改变引发⼀系列⾃动⾏为

### React 项⽬中使⽤装饰器

- yarn eject
- yarn add @babel/plugin-proposal-decorators
- 修改 package.json， 找到 babel 字段， 添加`"plugins": [ "@babel/plugin-proposal-decorators" ]`

## 3.mobx-react

### 在 Class 组件使⽤ mobx

- 设置 Store

```
import { observable, action } from 'mobx';
class AboutStore {
 @observable counter = 1;
 @action add() {
 this.counter++;
 }
}
export default new AboutStore;
```

- 提供数据源

```
import { Provider } from 'mobx-react';
ReactDOM.render(
 <React.StrictMode>
 <Router>
 <Provider homeStore={homeStore} aboutStore={aboutStore}>
 <App />
 </Provider>

 </Router>
 </React.StrictMode>,
 document.getElementById('root')
);
```

- 使用数据源

```
import { observer, inject } from 'mobx-react';
@inject('aboutStore')
@inject('homeStore')
@observer
class About extends Component {
 render() {
 return (
 <div>
 <h1>About</h1>
 <p>current counter : {this.props.aboutStore.counter}</p>
 <p>home counter: {this.props.homeStore.counter}</p>
 <Link to="/">去⾸⻚</Link>
 </div>
 );
 }
}

```

### 在函数组件使⽤ mobx

- 设置 Store

```
import { observable, action, computed } from 'mobx';
export class HomeStore {
 @observable counter = 1;
 @computed get doubleCounter() {
 return this.counter * 2;
 }
 @action add() {
 this.counter++;
 }
}
```

- 创建 Context 对象

```
import React from 'react';
import { HomeStore } from '../stores/home';
import { AboutStore } from '../stores/about';
export const storesContext = React.createContext({
 homeStore: new HomeStore(),
 aboutStore: new AboutStore()
});
```

- useContext

设置 useStores 函数，⽤于在函数组件
内获取 context 对象

```
import React from 'react';
import { storesContext } from '../contexts';
export const useStores = () => React.useContext(storesContext);
```

- 使⽤ context

观察组件，通过 useStore 获取 context
对象

```
import React, { Component } from 'react';
import { observer } from 'mobx-react';
import { useStores } from '../hooks/use-stores';
const About = observer(() => {
 const { homeStore, aboutStore } = useStores();
 return (
 <div>
 <h1>About</h1>
 <p>current counter : {aboutStore.counter}</p>
 <p>home counter: {homeStore.counter}</p>
 </div>
 )
}
```

**参考**

- [mobx 中文文档](https://cn.mobx.js.org/)
