title: 从简单的列表页浅谈Vue组件
date: 2015-11-12 10:46:27
categories: Vue
tags: [Vue]
---

##组件

什么是组件？

组件（Component）是 Vue.js 最强大的功能之一。组件可以扩展 HTML 元素，封装可重用的代码。在较高层面上，组件是自定义元素， Vue.js 的编译器为它添加特殊功能。在有些情况下，组件也可以是原生 HTML 元素的形式，以 js 特性扩展。

##例子


举个小例子：豆瓣电影排行榜

此例子包含文章列表，分页，这里把列表和分页分别当作单独的组件，加上父页面组件一共是3个

父组件：`app.vue`

子组件：`list.vue` `pagination.vue`

接口用的是豆瓣的jsonp，`https://api.douban.com/v2/movie/top250`

首先在main.js引入父组件，引入less，引入VueResource，实例化vue

```js
//main.js

import Vue from 'vue'
import app from './example1/app.vue'
import './example1/style.less'

import VueResource from 'vue-resource'
Vue.use(VueResource)

new Vue({
  render: h => h(app)
}).$mount('#app')

```

从分页开始，在src/example1/components文件夹下建立一个pagination.vue文件

```html
<!--pagination.vue-->
<template>
  <div class="text-center">
    <ul class="pagination">
      <li></li>
    </ul>
  </div>

</template>

<script type="text/javascript">
export default {
  data () {
    return {
    }
  },
  props: {
  },
  methods: {
  },
  computed: {
  },
  components: {
  },
  mounted () {
  }
}
</script>
```
通过props从父组件接收数据。组件可以为props指定验证要求。如果未指定验证要求，Vue会发出警告，如果不需要验证，可以用数组形式

```js
//pagination.vue

props: {
    //当前页码
    now: {
      type: Number,
      default: 1
    },
    //总页数
    all: {
      type: Number,
      default: 1,
      required: 1
    }
  }
```

父组件app.vue引入组件，并且把数据传给子组件

```html
<!--app.vue-->

<!--代码片段-->
<pagination :now="page.now" :all="page.all"></pagination>
<!--代码片段-->

<script type="text/javascript">

import pagination from './components/pagination.vue'
export default {
  data () {
    return {
    	page: {
    		now: 1,
    		all: 10
    	}
           
    }
  },
  components: {
    pagination
  }
}
</script>
```

根据传入的数据计算分页数据

```js
//pagination.vue

computed: {
    pages () {
      var pages = [];
      if(this.now > 1) pages.push({
        text: '上一页',
        num: this.now - 1
      });
      if(this.now - 1 > 2) pages.push({
        text: 1,
        num: 1
      });
      if(this.now - 1 > 3) pages.push({
        text: '...',
        num: 0
      });
      for (var i = 1; i <= this.all; i++) {
        if(this.now - i < 3 && this.now >= i || i - this.now < 3 && this.now <= i){
          pages.push({
            text: i,
            num: i
          });          
        }        
      }
      if(this.all - this.now > 3) pages.push({
        text: '...',
        num: 0
      });
      if(this.all - this.now > 2) pages.push({
        text: this.all,
        num: this.all
      });
      if(this.now < this.all) pages.push({
        text: '下一页',
        num: this.now + 1
      });
      
      return pages;
    }

  }
```
用v-for把分页渲染出来

```html
<!--pagination.vue-->

<template>
  <ul class="pagination">
    <li v-for="page in pages" :class="{ 'active' : page.num == now }">
        <a v-if="page.num" @click="toPage(page.num)">{{ page.text }}</a>
        <span v-else>{{ page.text }}</span>
    </li>
  </ul>
</template>
```

子组件通过自定义事件向父组件传事件和数据

```js
//pagination.vue

methods: {
    toPage (page) {
      this.$emit('toPage', page);

    }

  }
```

父组件监听这个事件，然后调用方法changePage

```html
<!--app.vue-->

<pagination :now="page.now" :all="page.all" @toPage="changePage"></pagination>
    
<script>
//代码片段
methods: {
    changePage (page) {
      this.page.now = page
    } 

}
//代码片段
</script>
```

至此完成了父组件把数据丢给子组件处理，子组件把操作反馈给父组件这一过程

列表组件list.vue

```html
<!--list.vue-->

<template>
  <div>
    <slot name="title"></slot>
    <div class="list clearfix row">
      <div class="col-xs-6" v-for="item in items">
        <div class="item">
          <div class="img">
            <img :src="item.images.small"></div>
          <slot name="year" :year="item.year">
            <div class="year">{{ item.year }}</div>
          </slot>
          <div class="title">{{ item.title }}</div>
          <div class="dis">{{ item.original_title }}</div>
        </div>
      </div>

    </div>
  </div>
</template>

<script type="text/javascript">
export default {
  data () {
    return {
      
    }
  },
  props: ['items']
}
</script>
```


来点动态的数据

```html
<!--app.vue-->

<template>
  <div class="container">
    <div class="loading" v-show="loading"></div>

    <list :items="items">
      <h1 class="text-center" slot="title">豆瓣电影排行榜</h1>
      <template slot="year" scope="props">
        <div class="year red">{{ props.year }}</div>
      </template>
    </list>
    <pagination :now="page.now" :all="page.all" @toPage="changePage"></pagination>
  </div>
</template>

<script type="text/javascript">
import list from './components/list.vue'
import pagination from './components/pagination.vue'
export default {
  data () {
    return {
      loading: 0,
      params: {
        //每页显示条数
        count: 10,
        //从第几条开始显示，初始0
        start: 0,
        //总条数
        total: 250
      },      
      items: []      
    }
  },
  methods: {
    getItems () {
      this.loading = 1;
      this.$http.jsonp('https://api.douban.com/v2/movie/top250', {
        params: this.params
      }).then(response => {
        if(response.ok) {
          this.items = response.data.subjects;
          this.params = {
            count: response.data.count,
            start: response.data.start,
            total: response.data.total
          }
        }

      }, response => {
        console.log(response);

      }).finally(function(){
        this.loading = 0;
      })

    },
    changePage (page) {
      this.params.start = (page - 1) * this.params.count;
      this.getItems();
    } 

  },
  components: {
    list,
    pagination
  },
  computed: {
    page () {
      return {
        now: this.params.start / this.params.count + 1,
        all: Math.ceil(this.params.total / this.params.count)
      }      
    }

  },
  mounted () {
    this.getItems();
  }
}
</script>
```


##几个点：

###prop传递数据

> 组件实例的作用域是孤立的。这意味着不能并且不应该在子组件的模板内直接引用父组件的数据。可以使用 props 把数据传给子组件。

> 类似于用 v-bind 绑定 HTML 特性到一个表达式，也可以用 v-bind 动态绑定 props 的值到父组件的数据中。每当父组件的数据变化时，该变化也会传导给子组件：

###自定义事件

> 父组件可以在使用子组件的地方直接用 v-on 来监听子组件触发的事件

###slot分发内容

> 组件不知道它的挂载点会有什么内容。挂载点的内容是~~由父组件决~~定的;组件很可能有它自己的模版

> 为了让组件可以组合，我们需要一种方式来混合父组件的内容与子组件自己的模板。这个过程被称为 内容分发 (或 “transclusion” 如果你熟悉 Angular)。Vue.js 实现了一个内容分发 API ，参照了当前 Web 组件规范草案，使用特殊的 `<slot>` 元素作为原始内容的插槽。

`通过slot插槽将标题插入到子组件`

###作用域插槽
> 父组件模板的内容在父组件作用域内编译；子组件模板的内容在子组件作用域内编译

> 作用域插槽是一种特殊类型的插槽，用作使用一个（能够传递数据到）可重用模板替换已渲染元素。
在子组件中，只需将数据传递到插槽，就像你将 prop 传递给组件一样

`通过作用域插槽将子组件数据传给父组件，再作为自定义模板重新插入到子组件`

###计算
`豆瓣接口接收参数和设计的不同，pages需要通过computed计算`

###分页的算法

##引申问题：
watch params变化

loading广播