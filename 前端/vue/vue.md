## 插件

## scoped样式

## mixin混入（合）

## 浏览器本地缓存

## 自定义事件

一种组件间通信的方式，适用于：子组件===>父组件

使用场景：A是父组件，B是子组件，B想给A传数据，那么就要在A中给B绑定自定义事件（事件的回调在A中）。

### 绑定自定义事件

**父组件：App.vue**

```vue
<template>
  <div>
    <!-- 通过父组件给子组件传递函数类型的props实现：子组件给父组件传递数据 -->
    <School :getSchoolName="getSchoolName"></School>

    <!-- 通过父组件给子组件绑定一个自定义事件实现：子组件给父组件传递数据(第一种写法，使用@或者v-on) -->
    <Student @showName.once="getStudentName"></Student>

    <!-- 通过父组件给子组件绑定一个自定义事件实现：子组件给父组件传递数据(第二种写法，使用ref) -->
    <Student ref="student"></Student>
  </div>
</template>

<script>
import School from './components/School.vue'
import Student from './components/Student.vue'

export default {
  name: 'App',
  components: {
    School,
    Student,
  },
  methods: {
    getSchoolName(SchoolName) {
      console.log('App收到了School组件发来的学校名：', SchoolName)
    },
    getStudentName(studentName, ...params) {
      console.log('App收到Student组件发过来的学生姓名: ', studentName, params)
    },
  },
  mounted() {
    setTimeout(() => {
      // this.$refs.student.$on('showName', this.getStudentName)
      this.$refs.student.$once('showName', this.getStudentName) // 只调用一次
    }, 3000)
  },
}
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
}
</style>
```

**子组件：Student.vue**

```vue
<template>
  <div>
    <h2>学生姓名：{{ name }}</h2>
    <h2>学生年龄：{{ age }}</h2>
    <button @click="sendStudentName">点击我把学生名发给父组件App</button>
  </div>
</template>

<script>
export default {
  name: 'Student',
  data() {
    return {
      name: '张三',
      age: 18,
    }
  },
  methods: {
    sendStudentName() {
      // 触发Student组件实例身上的showName事件
      this.$emit('showName', this.name, this.age)
    },
  },
}
</script>

<style scoped>
div {
  background-color: blue;
}
</style>
```

**子组件：School.vue**

```vue
<template>
  <div>
    <h2>学校名称：{{ schoolName }}</h2>
    <h2>学校地址：{{ address }}</h2>
    <button @click="showName">点我提示学校名</button>
    <button @click="getSchName()">点我调用props拿到的函数给父组件APP传递数据</button>
  </div>
</template>

<script>
export default {
  name: 'School',
  data() {
    return {
      schoolName: '尚硅谷',
      address: '北京昌平',
    }
  },
  methods: {
      showName(){
          alert(this.schoolName)
      },
      getSchName() {
        this.getSchoolName(this.schoolName)
      }
  },
  props: ['getSchoolName']
}
</script>

<style scoped>
div {
  background-color: pink;
}
</style>
```

### 解绑自定义事件

```vue
this.$off('showName') // 解绑一个自定义事件
// this.$off(['showName']) // 解绑多个自定义事件
// this.$off() // 解绑所有自定义事件
```

> 注意：销毁当前组件的实例后，实例上的自定义事件全部失效

## 全局事件总线

是一种组件间通信的方式，适用于任意组件间通信

安装全局事件总线

main.js

```js
new Vue({
  ...
  beforeCreate() {
    Vue.prototype.$bus = this // 安装全局事件总线
  },
  ...
})
```

使用事件总线

接收数据：A组件想接收数据，则在A组件中给$bus绑定自定义事件，事件的回调留在A组件自身

```vue
methods() {
  demo(data){...}
},
mounted() {
  this.$bus.$on('xxx', this.demo)
}
```

提供数据

```vue
this.$bus.$emit('xxx', 数据)
```

最好在beforeDestroy钩子中，用$off去解绑当前组件所用到的事件

## 消息订阅与发布

