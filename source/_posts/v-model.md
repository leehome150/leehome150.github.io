---
title: 如何理解Vue的.sync修饰符以及v-model
tags: Vue
abbrlink: 1c175475
date: 2021-11-26 14:34:32
categories: Vue
---

### .sync

vue 修饰符 sync 的功能是：当一个子组件改变了一个 props 的值时，这个变化也会同步到父组件中所绑定。

这个功能是怎么实现的呢？

Vue 组件不能修改 props 外部数据，但是$emit可以触发事件，并传参，$event 可获取$emit 的参数。

我们看代码：

```
//Child.vue
<template>
  <div id="child">
   子组件：{{money}}
//触发update:money事件，并传参
   <button @click="$emit('update:money',money-100)"><span>花钱</span></button>

 </div>
</template>

<script>


export default {

props:["money"]


};
</script>

<style>
#child{
  border :1px solid red;
  padding:2px;
}

</style>

```

然后$event 获取传的参数：

```
//App.vue
<template>
  <div id="app">
   父组件： 我现在有{{total}}
    <hr>
//$event 获取传的参数赋值给total
    <Child :money="total" v-on:update:money="total=$event"/>

 </div>
</template>

<script>
import Child from"./Child.vue";

export default {
  data(){
    return {
total:10000
    }


  },
  components: { Child:Child}

};
</script>

<style>
#app{
  border :2px solid blue;
  padding:2px;
}

</style>

```

因为这种场景很常见，所以尤雨溪发明了.sync 修饰符，上面的代码<strong>:money="total" v-on:update:money="total=$event"可以简化为:money.sync="total"</strong>

下面是运行动画：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/12/17207b0ebb162b47~tplv-t2oaga2asx-watermark.awebp)

[代码点击](https://codesandbox.io/s/eloquent-wave-bc7bb)

### v-model

v-model 也是个语法糖

```
v-model="x"等价于 ：value="x",@input="x=$event.target.value"

```

当 v-model 用于普通 input 上

```
<template>
  <div>
    {{message}}
    <hr/>
    <input v-model="message" placeholder="你好" />
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      message: ""
    };
  }
};
</script>

<style lang="scss">
</style>

```

动画演示：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/3/17277f6ca1d24f4d~tplv-t2oaga2asx-watermark.awebp)

当 v-model 用于组件上

父组件：

```
<template>
    <div>
        {{price}}
        <hr/>
        <test v-model="price"/>

    </div>
</template>

<script>
    import test from './components/Test.vue';
    export default {
        components:{test},
        name: "App",
        data(){
            return{
                price:100
            }
        }
    };
</script>

<style lang="scss">
</style>子组件

```

子组件：

```
<template>
  <div>
    <label>
      <input ref="input" :value="value"
      @input="$emit('input', $event.target.value)"//子组件触发事件，把输入的参数传给price
      >
    </label>
  </div>
</template>

<script>
export default {
  name: "App",
  props: ["value"],//子组件把value传给父组件，父组件绑定value="price"
};
</script>

<style lang="scss">
</style>

```

由上面代码我们可以看出子组件把 value 传给父组件，因为<strong> v-model="price"等价于 ：value="price",@input="price=$event.target.value",</strong>父组件动态绑定了 value，然后 input 事件那输入的 value 参数传给了 price,所以我们就能看到以下效果：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/3/1727846768ecc1e0~tplv-t2oaga2asx-watermark.awebp)
