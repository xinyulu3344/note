## vue-router的安装配置

vue-cli脚手架搭建的项目中，可选router

```bash
npm install vue-router --save
```

## 动态路由

url的path可能是不确定的，比如/user/:userId

## vue-router打包文件解析

## 路由懒加载

## 路由嵌套

## 路由参数传递

路由参数传递有两种方式：params参数和query参数

### params参数

定义路由组件User.vue

```vue
<template>
  <div>
    <h2>这是用户界面</h2>
    <p>这里是用户页面的内容</p>
    <p>用户ID是: {{ userId }}</p>
  </div>
</template>

<script>
  export default {
    name: "User",
    computed: {
      // 获取传递过来的参数userId
      userId() {
        return this.$route.params.userId
      }
    }
  }
</script>

<style scoped>
</style>
```

在router/index.js中定义路由

```js
const User = () => import("../components/User")
{
  path: "/user/:userId", //这里的userId，就是路由组件中使用的参数
  name: "User",
  component: User
}
```

在App.vue中使用组件，并给组件传参

```vue
<router-link :to="'/user/' + userId" tag="button" replace>用户</router-link>

data() {
    return {
      userId: "xinyulu"
  }
}
```



### query参数

定义路由组件Profile.vue

```vue
<template>
  <div>
    <h2>我是Profile</h2>
    <!--获取传递过来的参数-->
    <h2>{{$route.query}}</h2>
    <h2>{{$route.query.name}}</h2>
  </div>
</template>

<script>
  export default {
    name: "Profile"
  }
</script>

<style scoped>
</style>
```

在router/index.js中定义路由

```js
const Profile = () => import("../components/Profile")
{
  path: "/Profile",
  component: Profile
}
```

在App.vue中使用组件，并给组件传参

```vue
<router-link :to="{path: '/profile', query: {name: 'xinyulu'}}">档案</router-link>
```

## 导航守卫

推荐阅读：<https://zhuanlan.zhihu.com/p/54112006>

> 导航守卫就是路由跳转过程中的一些钩子函数，再直白点路由跳转是一个大的过程，这个大的过程分为跳转前中后等等细小的过程，在每一个过程中都有一函数，这个函数能让你操作一些其他的事儿的时机，这就是导航守卫

### 导航守卫的分类

导航守卫分为：全局的、单个路由独享的、组件内的三种

### 全局守卫

> 是指路由实例上直接操作的钩子函数，他的特点是所有路由配置的组件都会触发，直白点就是触发路由就会触发这些钩子函数

```js
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

1. beforeEach

   在路由跳转前触发，参数包括to,from,next（参数会单独介绍）三个，这个钩子作用主要是用于登录验证，也就是路由还没跳转提前告知，以免跳转了再通知就为时已晚

2. beforResolve

   这个钩子和beforeEach类似，也是路由跳转前触发，参数也是to,from,next三个。

   在 beforeEach 和 组件内beforeRouteEnter 之后，afterEach之前调用。

3. afterEach

   和beforeEach相反，他是在路由跳转完成后触发，参数包括to,from没有了next（参数会单独介绍）,他发生在beforeEach和beforeResolve之后，beforeRouteEnter（组件内守卫，后讲）之前。

### 路由独享守卫

> 是指在单个路由配置的时候也可以设置的钩子函数

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```



1. beforeEnter

   和beforeEach完全相同，如果都设置则在beforeEach之后紧随执行

### 组件内守卫

> 是指在组件内执行的钩子函数，类似于组件内的生命周期，相当于为配置路由的组件添加的生命周期钩子函数。钩子函数按执行顺序包括beforeRouteEnter、beforeRouteUpdate (2.2+)、beforeRouteLeave三个

```vue
<template>
  ...
</template>
<script>
export default{
  data(){
    //...
  },
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
</script>

<style>
  ...
</style>
```



### 完整的导航解析流程

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数。

## keep-alive

> `<keep-alive>`是Vue的内置组件，能在组件切换过程中将状态保留在内存中，防止重复渲染DOM。
>
> `<keep-alive>` 包裹动态组件时，会**缓存不活动的组件实例**，而不是销毁它们。
>
> `<keep-alive>` 与 `<transition>`相似，只是一个抽象组件，它不会在DOM树中渲染(真实或者虚拟都不会)，也不在父组件链中存在，比如：你永远在 `this.$parent` 中找不到 `keep-alive` 。

### 两个重要属性：include，exclude

include: 字符串或正则表达式。只有匹配的组件会被缓存。

exclude: 字符串或正则表达式。任何匹配的组件都不会被缓存。

exclude优先级大于include

```vue
<keep-alive>
    <router-view v-if="$route.meta.keepAlive"></router-view>
</keep-alive>
```

```js
// router/index.js
export default new Router({
  routes: [
    {
      path: '/',
      name: 'Hello',
      component: Hello,
      meta: {
        keepAlive: false // 不需要缓存
      }
    },
    {
      path: '/page1',
      name: 'Page1',
      component: Page1,
      meta: {
        keepAlive: true // 需要被缓存
      }
    }
  ]
})
```

