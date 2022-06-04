title: Vue路由进阶
date: 2017-1-15 10:46:27
categories: Vue
tags: [Vue]
---

### 导航钩子
有的时候，我们可能有这样的需求，当页面切换时，需要做一些操作来更改某些信息，这就需要拦截路由切换。vue-router 提供了导航钩子来拦截导航，让它完成跳转或取消。有多种方式可以在路由导航发生时执行钩子：全局的, 单个路由独享的, 或者组件级的。

#### 全局钩子
你可以使用 router.beforeEach 注册一个全局的 before 钩子：
```
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

当一个导航触发时，全局的 before 钩子按照创建顺序调用。钩子是异步解析执行，此时导航在所有钩子 resolve 完之前一直处于 等待中。
每个钩子方法接收三个参数：
to: Route: 即将要进入的目标 路由对象

from: Route: 当前导航正要离开的路由

next: Function: 一定要调用该方法来 resolve 这个钩子。执行效果依赖 next 方法的调用参数。

next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed （确认的）。

next(false): 中断当前的导航。如果浏览器的 URL 改变了（可能是用户手动或者浏览器后退按钮），那么 URL 地址会重置到 from 路由对应的地址。

next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。

确保要调用 next 方法，否则钩子就不会被 resolved。
同样可以注册一个全局的 after 钩子，不过它不像 before 钩子那样，after 钩子没有 next 方法，不能改变导航：
```
router.afterEach(route => {
  // ...
})
```

#### 某个路由独享的钩子
你可以在路由配置上直接定义 beforeEnter 钩子：
```
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
这些钩子与全局 before 钩子的方法参数是一样的。

#### 组件内的钩子
最后，你可以在路由组件内直接定义以下路由导航钩子：

beforeRouteEnter
beforeRouteUpdate (2.2 新增)
beforeRouteLeave

```
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当钩子执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是改组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```

介绍完上述三种不同作用域的导航钩子，我们来看一个更实际的例子。使用vue-router之后，页面的切换时无刷新的，也就是说和服务端没有交互，这样就会产生一个问题，如果某个页面这个用户没有权限访问，这个如何来控制？有了导航钩子，这个自然就可以放到导航钩子来执行，示例代码如下：
```
// 权限数组
const permissions = {
  'user': 1,   // 其中user表示路由名称  1表示有权限访问 0 表示无权限访问
  'userProfile': 0
}

router.beforeEach((to, from, next) => {
  let name = to.name;
  if (name) {
    if (permissions[name]) {
       next();
    }
    else {
       alert(无权限访问);
       next(false);
    }
  }
  else {
    next();
  }
})
```

### 路由元信息
路由定义的时候，除了path、name、component(s)属性外，还可以有meta属性，这个属性用来为路由定义一些扩展的属性，比如每个页面都有title等。vue-router不会自动处理meta里面的属性，需要我们在导航钩子里面去处理这些元信息，我们就以页面title为例：
```
const routes = [
  {
    path: '/profile/user',
    name: 'user',
    component: User,
    meta: {
      title: '用户列表'
    },
    children: [
      {
        path: ':id',
        component: UserProfile,
        meta: {
          title: '用户详情'
        }
      }
    ]
  }
]


router.beforeEach((to, from, next) => {
  document.title = to.meta ? to.meta.title : '';
})
```

### 过渡动效
过渡效果的基础知识已经在vue基础中讲过，如果不清楚过度效果，请先学习过渡效果。<router-view> 是基本的动态组件，所以我们可以用 <transition> 组件给它添加一些过渡效果：
```
<transition>
  <router-view></router-view>
</transition>
```

上面的用法会给所有路由设置一样的过渡效果，如果你想让每个路由组件有各自的过渡效果，可以在各路由组件内使用 <transition> 并设置不同的 name。
```
const User = {
  template: `
    <transition name="slide">
    <div>
      <div>这里是用户列表</div>
      <ul>
        <li><router-link to="/profile/user/1">root</router-link></li>
        <li><router-link to="/profile/user/2">test</router-link></li>
      </ul>
     <router-view></router-view>
   </div>
   </transition>
   `
}

// 用户详情组件
const UserProfile = {
  template: '<transition name="fade"><div>用户# {{ $route.params.id }}详情页</div></transition>'
}
```

### 滚动行为
由于vue-router是单页无刷新应用，所以假设有一个页面内容很多，有垂直滚动条，如果把页面滚动到最底部，然后再切换路由，默认情况下，新的页面也在最底部，如果想要页面滚到顶部，vue-router 能做到，而且更好，它让你可以自定义路由切换时页面如何滚动。
注意: 这个功能只在 HTML5 history 模式下可用。
当创建一个 Router 实例，你可以提供一个 scrollBehavior 方法：
```
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
  }
})
```
scrollBehavior 方法接收 to 和 from 路由对象。第三个参数 savedPosition 当且仅当 popstate 导航 (通过浏览器的 前进/后退 按钮触发) 时才可用。
这个方法返回滚动位置的对象信息，长这样：

{ x: number, y: number }
{ selector: string }
如果返回一个布尔假的值，或者是一个空对象，那么不会发生滚动。

举例：
```
scrollBehavior (to, from, savedPosition) {
  return { x: 0, y: 0 }
}
```

上例对于所有路由导航，简单地让页面滚动到顶部。
返回 savedPosition，在按下 后退/前进 按钮时，就会像浏览器的原生表现那样：
```
scrollBehavior (to, from, savedPosition) {
  if (savedPosition) {
    return savedPosition
  } else {
    return { x: 0, y: 0 }
  }
}
```

上例表示如果是按了浏览器的前进、后退按钮，则页面滚动到历史页面原来的位置，否则滚动到页面顶部。