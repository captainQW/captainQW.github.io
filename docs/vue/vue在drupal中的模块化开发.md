title: vue在drupal中的模块化开发
date: 2017-1-27 22:44:41
tags:
  - drupal
  - vue
categories:
  - drupal
---

`vue`的模块化开发，基于`webpack`，`gulp`

由于`vue`组件系统，`drupal`中的`vue`可以直接在主题中写组件

OK，初始化一个主题，`lancer`

本地先安装node

然后在`lancer`目录下先放个`package.json`文件

```json
{
  "name": "voss",
  "version": "0.0.1",
  "description": "webapp with Vue.js for voss.",
  "author": "panjun.liu <panjun.liu@verycloud.cn>",
  "private": true,
  "scripts": {
    "dev": "node build/dev-server.js",
    "build": "node build/build.js",
    "unit": "karma start test/unit/karma.conf.js --single-run",
    "test": "npm run unit",
    "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs"
  },
  "dependencies": {
    "vue": "^2.0.3",
    "babel-runtime": "^6.11.6"
  },
  "devDependencies": {
    "babel-core": "^6.16.0",
    "babel-loader": "^6.2.5",
    "babel-plugin-transform-runtime": "^6.15.0",
    "babel-preset-es2015": "^6.16.0",
    "babel-preset-stage-2": "^6.16.0",
    "bulma": "0.2.1",
    "chai": "^3.5.0",
    "chart.js": "^2.3.0",
    "connect-history-api-fallback": "^1.3.0",
    "css-loader": "^0.25.0",
    "element-ui": "^1.0.2",
    "eslint": "^3.7.0",
    "eslint-config-standard": "^6.2.0",
    "eslint-friendly-formatter": "^2.0.6",
    "eslint-loader": "^1.5.0",
    "eslint-plugin-html": "^1.5.3",
    "eslint-plugin-promise": "^2.0.1",
    "eslint-plugin-standard": "^2.0.1",
    "eventsource-polyfill": "^0.9.6",
    "express": "^4.14.0",
    "extract-text-webpack-plugin": "^1.0.1",
    "fastclick": "^1.0.6",
    "file-loader": "^0.9.0",
    "function-bind": "^1.1.0",
    "html-webpack-plugin": "^2.22.0",
    "http-proxy-middleware": "^0.17.2",
    "inject-loader": "^2.0.1",
    "isparta-loader": "^2.0.0",
    "json-loader": "^0.5.4",
    "karma": "^1.3.0",
    "karma-coverage": "^1.1.1",
    "karma-mocha": "^1.2.0",
    "karma-phantomjs-launcher": "^1.0.2",
    "karma-sinon-chai": "^1.2.3",
    "karma-sourcemap-loader": "^0.3.7",
    "karma-spec-reporter": "0.0.26",
    "karma-webpack": "^1.8.0",
    "less": "^2.7.1",
    "less-loader": "^2.2.3",
    "lolex": "^1.5.1",
    "mocha": "^3.1.0",
    "moment": "^2.14.1",
    "ora": "^0.3.0",
    "phantomjs-prebuilt": "^2.1.12",
    "shelljs": "^0.7.4",
    "sinon": "^1.17.6",
    "sinon-chai": "^2.8.0",
    "swiper": "^3.3.1",
    "url-loader": "^0.5.7",
    "vue-hot-reload-api": "^2.0.6",
    "vue-html-loader": "^1.2.3",
    "vue-loader": "^9.7.0",
    "vue-resource": "^1.0.3",
    "vue-router": "^2.0.1",
    "vue-style-loader": "^1.0.0",
    "vuex": "^2.0.0",
    "webpack": "^1.13.2",
    "webpack-dev-middleware": "^1.8.4",
    "webpack-hot-middleware": "^2.13.0",
    "webpack-merge": "^0.14.1"
  }
}
```

运行`npm install`开始初始化环境

之后`npm install gulp gulp-util gulp-less gulp-concat gulp-open gulp-uglify gulp-cssmin gulp-md5-plus gulp-file-include gulp-clean gulp-css-spriter gulp-css-base64 webpack gulp-connect --save-dev`安装`gulp`环境

初始化`gulpfile.js`

```js
var gulp = require('gulp'),
  os = require('os'),
  gutil = require('gulp-util'),
  less = require('gulp-less'),
  concat = require('gulp-concat'),
  gulpOpen = require('gulp-open'),
  uglify = require('gulp-uglify'),
  cssmin = require('gulp-cssmin'),
  md5 = require('gulp-md5-plus'),
  fileinclude = require('gulp-file-include'),
  clean = require('gulp-clean'),
  spriter = require('gulp-css-spriter'),
  base64 = require('gulp-css-base64'),
  webpack = require('webpack'),
  webpackConfig = require('./build/webpack.prod.conf.js'),
  connect = require('gulp-connect');

var host = {
  path: 'dist/',
  port: 3000,
  html: 'index.html'
};

//mac chrome: "Google chrome", 
var browser = os.platform() === 'linux' ? 'Google chrome' : (
  os.platform() === 'darwin' ? 'Google chrome' : (
    os.platform() === 'win32' ? 'chrome' : 'firefox'));
var pkg = require('./package.json');

//将图片拷贝到目标目录
gulp.task('copy:images', function (done) {
  gulp.src(['src/images/**/*']).pipe(gulp.dest('dist/images')).on('end', done);
});

//压缩合并css, css中既有自己写的.less, 也有引入第三方库的.css
gulp.task('lessmin', function (done) {
  gulp.src(['src/css/main.less', 'src/css/*.css'])
    .pipe(less())
    //这里可以加css sprite 让每一个css合并为一个雪碧图
    //.pipe(spriter({}))
    .pipe(concat('style.min.css'))
    .pipe(gulp.dest('dist/css/'))
    .on('end', done);
});

//将js加上10位md5,并修改html中的引用路径，该动作依赖build-js
gulp.task('md5:js', ['build-js'], function (done) {
  gulp.src('dist/js/*.js')
    .pipe(md5(10, 'dist/app/*.html'))
    .pipe(gulp.dest('dist/js'))
    .on('end', done);
});

//将css加上10位md5，并修改html中的引用路径，该动作依赖sprite
gulp.task('md5:css', ['sprite'], function (done) {
  gulp.src('dist/css/*.css')
    .pipe(md5(10, 'dist/app/*.html'))
    .pipe(gulp.dest('dist/css'))
    .on('end', done);
});

//用于在html文件中直接include文件
gulp.task('fileinclude', function (done) {
  gulp.src(['src/app/*.html'])
    .pipe(fileinclude({
      prefix: '@@',
      basepath: '@file'
    }))
    .pipe(gulp.dest('dist/app'))
    .on('end', done);
  // .pipe(connect.reload())
});

//雪碧图操作，应该先拷贝图片并压缩合并css
gulp.task('sprite', ['copy:images', 'lessmin'], function (done) {
  var timestamp = +new Date();
  gulp.src('dist/css/style.min.css')
    .pipe(spriter({
      spriteSheet: 'dist/images/spritesheet' + timestamp + '.png',
      pathToSpriteSheetFromCSS: '../images/spritesheet' + timestamp + '.png',
      spritesmithOptions: {
        padding: 10
      }
    }))
    .pipe(base64())
    .pipe(cssmin())
    .pipe(gulp.dest('dist/css'))
    .on('end', done);
});

gulp.task('clean', function (done) {
  gulp.src(['dist'])
    .pipe(clean())
    .on('end', done);
});

gulp.task('watch', function (done) {
  gulp.watch(['/var/www/html/voss/**/*.vue', '/var/www/html/voss/**/*.js', '/var/www/html/voss/**/*.less'], ['lessmin', 'build-js', 'fileinclude'])
    .on('end', done);
});

gulp.task('connect', function () {
  console.log('connect------------');
  connect.server({
    root: host.path,
    port: host.port,
    livereload: true
  });
});

gulp.task('open', function (done) {
  gulp.src('')
    .pipe(gulpOpen({
      app: browser,
      uri: 'http://localhost:3000/app'
    }))
    .on('end', done);
});

var myDevConfig = Object.create(webpackConfig);

var devCompiler = webpack(myDevConfig);

//引用webpack对js进行操作
gulp.task("build-js", ['fileinclude'], function(callback) {
  devCompiler.run(function(err, stats) {
    if(err) throw new gutil.PluginError("webpack:build-js", err);
    gutil.log("[webpack:build-js]", stats.toString({
      colors: true
    }));
    callback();
  });
});

//发布
gulp.task('default', ['connect', 'fileinclude', 'md5:css', 'md5:js', 'open']);

//开发
gulp.task('dev', ['connect', 'copy:images', 'fileinclude', 'lessmin', 'build-js', 'watch', 'open']);
```

命令行下`gulp`回车，启动监控任务

OK，现在可以开始`vue`的开发了

主题下新建个目录`src`用于存放`vue`的组件，以及启动`js`文件`main.js`

```js
import Vue from 'vue'
import Router from 'vue-router'
import Resource from 'vue-resource'

// Contrib.
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-default/index.css'

// Voss.
import App from './App'
import Notfound from './pages/base/404'
import Organization from './pages/base/Organization'
import Region from './pages/base/Region'
import Position from './pages/base/Position'
import Education from './pages/base/Education'
import CompanyAuthority from './pages/base/CompanyAuthority'
import CompanyAuthorityConfig from './pages/base/CompanyAuthorityConfig'
import Memory from './pages/base/Memory'
import Variables from './pages/base/Variables'

// vuex store
import store from './store'

Vue.use(Router)
Vue.use(Resource)
Vue.use(ElementUI)

// 全局闭包，公用函数库，也可以放置到vuex2.0中
global.$fn = {
  // 获取url中的参数，允许默认值的设定
  getparam (name, defaultValue) {
    let reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)')
    let r = window.location.search.substr(1).match(reg)
    return (!r) ? ((!defaultValue) ? '' : defaultValue) : unescape(r[2])
  },
  init () {
    // 登录或注销后需要清空cache
    // store.commit('isAuth', false)
    // this.csrf.requestToken()
  },
  // 随机字符串
  getCode (n) {
    let chars = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'
    let res = ''
    for (let i = 0; i < n; i++) {
      res += chars[Math.floor(Math.random() * chars.length)]
    }
    return res
  },
  // cache 机制
  cache: [],
  // 默认300秒数据缓存时间
  exp: 300,
  getCache (url) {
    let json = this.cache[url]
    return (json && (json.exp >= this.timestamp())) ? json.data : null
  },
  setCache (url, data, exp) {
    // 默认300秒,{data,exp}
    this.cache[url] = {data: data, exp: this.timestamp() + ((!exp) ? this.exp : exp)}
    // console.log(this.cache)
  },
  httprl (method, url, sendData) {
    var options = {
      headers: {
        'Content-Type': 'application/json;charset=utf-8',
        'Accept': 'application/json'
      }
    }
    // handel get method.
    if (method === 'GET' || method === 'get') {
      options.params = sendData
      let promise = new Promise(function (resolve, reject) {
        Vue.http.get(url, options).then((result) => {
          var data = result.data
          resolve(data)
        }, (error) => {
          reject(error)
        })
      })
      return promise
    } else {
      // handle post put delete method.
      if (this.csrf.isExpired()) {
        let promise = new Promise(function (resolve, reject) {
          global.$fn.csrf.requestToken().then(function (token) {
            options.headers['X-CSRF-Token'] = token
            switch (method) {
              case 'post':
              case 'POST':
                Vue.http.post(url, sendData, options).then((result) => {
                  resolve(result.data)
                }, (error) => {
                  reject(error)
                })
                break
              case 'put':
              case 'PUT':
                Vue.http.put(url, sendData, options).then((result) => {
                  resolve(result.data)
                }, (error) => {
                  reject(error)
                })
                break
              case 'del':
              case 'DEL':
                Vue.http.delete(url, sendData, options).then((result) => {
                  resolve(result.data)
                }, (error) => {
                  reject(error)
                })
                break
            }
          })
        })
        return promise
      } else {
        options.headers['X-CSRF-Token'] = this.csrf.getToken()
        let promise = new Promise(function (resolve, reject) {
          switch (method) {
            case 'post':
            case 'POST':
              Vue.http.post(url, sendData, options).then((result) => {
                resolve(result.data)
              }, (error) => {
                reject(error)
              })
              break
            case 'put':
            case 'PUT':
              Vue.http.put(url, sendData, options).then((result) => {
                resolve(result.data)
              }, (error) => {
                reject(error)
              })
              break
            case 'del':
            case 'DEL':
              Vue.http.delete(url, sendData, options).then((result) => {
                resolve(result.data)
              }, (error) => {
                reject(error)
              })
              break
          }
        })
        return promise
      }
    }
  },
  csrf: {
    getToken: function () {
      return store.state.csrf.token
    },
    getExpire: function () {
      return store.state.csrf.expire
    },
    nowTime: function () {
      let date = new Date()
      return date.getTime()
    },
    isExpired: function () {
      let csrfToken = global.$fn.csrf.getToken()
      let tokenExpire = global.$fn.csrf.getExpire()
      let nowTime = global.$fn.csrf.nowTime()
      return (csrfToken === undefined || !csrfToken.length || (nowTime >= tokenExpire))
    },
    requestToken: function () {
      let promise = new Promise(function (resolve, reject) {
        Vue.http.get('rest/session/token').then(function (result) {
          store.state.csrf.token = result.data
          store.state.csrf.expire = global.$fn.csrf.nowTime() + store.state.csrf_token_expire * 1000
          resolve(result.data)
        }, function (error) {
          console.error('Error! Fail to get csrf token!')
          console.error(error)
          reject(error)
        })
      })
      return promise
    }
  },
  toast (mode, title) {
    store.commit('toast', {
      id: new Date().getTime(),
      title: (title && title.length) > 0 ? title : '操作完成',
      mode: mode
    })
  },
  // 时间戳
  timestamp () {
    return Math.floor(new Date().getTime() / 1000)
  }
}

/*
global.$fn.httprl('POST', 'api/login', {
  username: 'agent', password: '123.com'
}).then(
  (result) => {
    console.log(result)
  })
*/
const routes = [
  {
    path: '/base/organization',
    component: Organization,
    meta: {
      title: '组织架构'
    }
  },
  {
    path: '/base/organization/department',
    component: Organization,
    meta: {
      title: '部门'
    }
  },
  {
    path: '/404',
    component: Notfound
  },
  {
    path: '*',
    redirect: '/404'
  }
]

const router = new Router({
  base: '/profile/',
  mode: 'history',
  linkActiveClass: 'is-active', // router-link active样式
  /*
  由于本项目采用内滚动布局，此处代码无效，需要自行hack vue-router获得此能力
  saveScrollPosition: true,
  scrollBehavior (to, from, savedPosition) {
  },
  */
  routes // short for routes: routes
})

router.beforeEach((to, from, next) => {
  // if (store.getters.getIsAuth || !to.meta.auth) {
  //  next()
  // } else {
  // 判断是否登录，（可以通过接口，Vuex状态 token）
  // 没有登录走下面逻辑
  document.title = to.meta.title + ' ' + '| 后台管理'
  if (global.$fn.csrf.isExpired()) {
    global.$fn.csrf.requestToken()
  }
  next()
  // }
})

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

`src`目录下新建个文件`App.vue`，作为应用的主文件

```html
<template>
  <section id="app-container">
    <xheader></xheader>
    <sidebar></sidebar>
    <div id="main-content">
      <router-view keep-alive></router-view>
    </div>
    <transition name="fade">
    <modal v-show="isModal" @closemodal="closeModal()" :mode="modalinfo.mode" :title="modalinfo.title"" :content="modalinfo.content" :width="modalinfo.width" :ok="modalinfo.ok"" :cancel="modalinfo.cancel"></modal>
    </transition>
  </div>
</template>
<script>
// vuex全局数据
import store from './store'
import Xheader from './components/header'
import Sidebar from './components/sidebar'
import { Modal } from './components/modal'
export default {
  store: store,
  components: {
    Modal,
    Xheader,
    Sidebar
  },
  data () {
    return {
      modalinfo: {
        mode: 'alert',
        title: '',
        content: '',
        callback: null
      }
      // data
    }
  },
  computed: {
    isModal () {
      return this.$store.getters.getModal
    }
    /*
    isAuth () {
      return this.$store.getters.getIsAuth
    },
    toastArray () {
      return this.$store.getters.getToast
    },
    preloaderShow () {
      return this.$store.getters.getPreloader
    }
    */
  },
  methods: {
    showModal (mode, title, content, width, ok, cancel) {
      this.$set(this.modalinfo, 'mode', mode)
      this.$set(this.modalinfo, 'title', title)
      this.$set(this.modalinfo, 'content', content)
      this.$set(this.modalinfo, 'width', width)
      this.$set(this.modalinfo, 'ok', ok)
      this.$set(this.modalinfo, 'cancel', cancel)
      store.commit('modal', true)
    },
    closeModal () {
      store.commit('modal', false)
    }
  },
  mounted () {
    // let ico = require('./assets/images/vueico.png')
    // let icon = require('./assets/images/vuelogo.png')

    // document.getElementById('linkIcon').href = ico
    // document.getElementById('linkAppIcon').href = icon

    window.applicationCache.addEventListener('updateready', (e) => {
      if (window.applicationCache.status === window.applicationCache.UPDATEREADY) {
        console.log('system update')
        window.location.reload()
      }
    }, false)
  }
}
</script>
<style lang="less">
  @import './lancer.less';
</style>
```

哦，对了，还有对应的`css`文件，当然这里可以直接用`less`文件，gulp监控任务会自动编译成css文件

这里我们直接`import`引入`less`文件，基于相对路径
