title: Vue路由入门
date: 2017-1-12 10:46:27
categories: Vue
tags: [Vue]
---

### 1、概述
Vue路由（vue-router）是Vue用来创建单页应用的第三方库，使用vue-router可以很方便的创建无刷新切换页面的应用。vue-router基于vue的组件系统（不熟悉组件请先学习Vue组件），要使用vue-router只需要将路径和组件建立对应关系，这样路由系统就知道哪个路径需要渲染哪个组件。

### 2、开始
在开始写第一个路由示例之前，我们先来了解一些基本概念。
（1）路由
路由即url路径和组件之间的对应关系。
（2）router-link组件
 router-link组件即页面上连接到某个路由的超链接，最终渲染后会变成a标签。比如我需要在页面上设置连接到/profile/user的超链接，则只要在页面放上&lt;router-link to="/profile/user">用户列表&lt;/router-link>就行了。
 （3）router-view组件
 router-view组件是路由组件渲染的容器，一旦路由匹配到了，vue就会将路由对应的组件渲染到router-view标签所在的位置。比如点击上例中的profile/user超链接，vue首先查找profile/user对应的路由是什么，查找到后再找出路由对应的组件，然后将组件渲染到router-view标签。

了解上述概念之后，我们可以开始写我们的第一个路由示例，我们首先创建index.html,代码如下：
```
<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/profile/user">用户列表</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<!-- 注意，这里引入vue-router.js对于script方式引入文件的方式是必不可少的.我们以后再讨论使用ES6模块引入的方式 -->
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
<!-- 应用入口文件 -->
<script src="/js/app.js"></script>
```

接着我们继续创建app.js，代码如下：
```
// 1. 定义（路由）组件。
// 在现实的开发中，一般将组件放在单独的文件中，然后从其他文件 import 进来
// 为了简单起见，我们这里先使用字符串模板组件，以后再讨论单文件组件（.vue文件）
const User = {
  template: '<div>这里是用户列表</div>' 
}

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
const routes = [
  { 
    path: '/profile/user', 
    component: User 
  }
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  routes // ES6（缩写）相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')

// 现在，应用已经启动了！
```

至此，一个最简单的路由应用就完成了，部署到服务器后，打开http://localhost,则可以看到界面有一个“用户详情”超链接，点击超链接，界面会显示“这里是用户列表”，同时界面并没有刷新，所有的操作都在一个界面内完成，所以这也叫单页应用，与以前那些点击超链接就要刷新页面的应用相比是不是用户体验更高？

### 动态路由
有的时候，我们的url里面需要带参数，比如上例中，我们需要查看不同用户的详情，那么url应该是/profile/user/1这样，里面的“1”即用户ID，可以动态变化，要实现这样的功能，我们只需要对我们的路由定义做一些微小变动。
```
const routes = [
  { 
    path: '/profile/user/:id', 
    component: User 
  }
]

const router = new VueRouter({
  routes // ES6（缩写）相当于 routes: routes
})

```

可以看到路由的定义唯一不同的地方就是在url上加了“:id”占位符，那么像“/profile/user/1”、“/profile/user/2”这样的url都会匹配到这个路由。
一个『路径参数』使用冒号 : 标记。当匹配到一个路由时，参数值会被设置到 this.$route.params，可以在每个组件内使用。于是，我们可以更新 User 的模板，输出当前用户的 ID：
```
const User = {
  template: '<div>这里是用户# {{ $route.params.id }}详情页</div>'
}
```

你可以在一个路由中设置多段『路径参数』，对应的值都会设置到 $route.params 中。例如：profile/user/:id/post/:pid，则可以使用$route.params.id和$route.params.pid来引用路由参数。如果url带有查询参数，则参数会被设置到$route.query对象。

### 嵌套路由
在实际的应用中，我们可能会碰到某个页面下面还有子页面的情况，比如用户列表页面里面点击某个用户，显示用户详情，我们定义/profile/user为用户列表url，/profile/user/:id为用户详情url，那么这两个URL就是父子关系，即嵌套关系，这样的URL我们应该怎么定义呢？
```
const routes = [
    { path: '/profile/user', component: User,
      children: [
        {
          path: ':id',
          component: UserProfile
        }
      ]
    }
  ]
```
你会发现，children 配置就是像 routes 配置一样的路由配置数组，所以呢，你可以嵌套多层路由。
上面只是定义了路由的结构，如果要在父页面里面显示子页面的内容，那么父页面的模板我们该如何定义？我们前面已经说过，要让路由的组件能在页面中渲染，必须在页面中放router-view组件，那么父页面的模板的定义可以这样：
```
const User = {
  template: `
    <div class="user">
      <h2>这是用户列表页面</h2>
      <router-link to="/profile/user/1">root</router-link>
      <router-link to="/profile/user/2">test</router-link>
      <router-view></router-view>
    </div>
  `
}
```
### 编程式的导航
除了使用 <router-link> 创建 a 标签来定义导航链接，我们还可以借助 router 的实例方法，通过编写代码来实现。
#### router.push(location)
想要导航到不同的 URL，则使用 router.push 方法。这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，则回到之前的 URL。
当你点击 <router-link> 时，这个方法会在内部调用，所以说，点击 <router-link :to="..."> 等同于调用 router.push(...)。该方法的参数可以是一个字符串路径，或者一个描述地址的对象。例如：
```
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: 123 }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

#### router.replace(location)
跟 router.push 很像，唯一的不同就是，它不会向 history 添加新记录，而是跟它的方法名一样 —— 替换掉当前的 history 记录。

#### router.go(n)
这个方法的参数是一个整数，意思是在 history 记录中向前或者后退多少步，类似 window.history.go(n)。
```
// 在浏览器记录中前进一步，等同于 history.forward()
router.go(1)

// 后退一步记录，等同于 history.back()
router.go(-1)

// 前进 3 步记录
router.go(3)

// 如果 history 记录不够用，那就默默地失败呗
router.go(-100)
router.go(100)
```

### 命名路由
到此为止，我们例子中定义的路由都只有path和component两个属性，有时候，通过一个名称来标识一个路由显得更方便一些，特别是在链接一个路由，或者是执行一些跳转的时候。你可以在创建 Router 实例的时候，在 routes 配置中给某个路由设置名称。
```
const router = new VueRouter({
  routes: [
    {
      path: '/profile/user/:id',
      name: 'user',
      component: User
    }
  ]
})
```

要链接到一个命名路由，可以给 router-link 的 to 属性传一个对象：
```
<router-link :to="{ name: 'user', params: { id: 123 }}">User</router-link>
```

### 命名视图
我们上面的例子中，所有的模板里面都只有一个router-view组件，也就是说，一个路由只能渲染一个组件，但是有时候想同时（同级）展示多个视图，而不是嵌套展示，例如创建一个布局，有 sidebar（侧导航） 和 main（主内容） 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 router-view 没有设置名字，那么默认为 default。
```
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 components 配置（带上 s）：

```
const router = new VueRouter({
  routes: [
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

### 重定向 和 别名
#### 重定向
重定向也是通过 routes 配置来完成，下面例子是从 /a 重定向到 /b：
```
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})
```

重定向的目标也可以是一个命名的路由：
```
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})
```

甚至是一个方法，动态返回重定向目标：
```
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})
```

#### 别名
『重定向』的意思是，当用户访问 /a时，URL 将会被替换成 /b，然后匹配路由为 /b，那么『别名』又是什么呢？
/a 的别名是 /b，意味着，当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样。
上面对应的路由配置为：
```
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```
