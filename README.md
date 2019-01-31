# ssrDemo
ssr测试项目
 **hello，蕾蒂丝and浸透们** 
   前一阵子对vue SSR一顿学习跟百度，也终于对ssr有一个大致的了解，自己也根据官网讲解撸了一个小demo，今天就把学习的一些东西分享给各位同学，希望大家共同进步！
   
   知识点一：什么是SSR
      SSR中文学名服务端渲染，英文全拼server side render ，大家中所周知随着vue等前端框架的兴起。现在经过打包后请求到的首次页面时这样的：
      
```
<!DOCTYPE html><html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title></head>

<body>

    <div id="app"></div>
    <script type=text/javascript src=./static/js/bundle.js></script>
    
</body>

</html>
```
 这样的页面只有一个div 没有任何实质性的内容，只有等js执行完才能有内容的填充。但是爬虫并不会等待你js的执行，所以那你的页面没有SEO(
搜索引擎优化),没有SEO优化也就意味着你的网站被搜索到的几率很低，大家看不到你的网站，对于流量变现的网站就是少了很多money啊。

 知识点二：SSR的优缺
所有有了需求就会有生产，vue SSR应运而生。但是也并不是说SSR就一定要用，根据自己的需求合理的运用SSR。
  列出一些优缺点大家自己衡量（文字很多大家不想看跳过即可）： 
   优点：
     1.更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。 
     2.更快的内容到达时间 (time-to-content)，特别是对于缓慢的网络情况或运行缓慢的设备。无需等待所有的 JavaScript 都完成下载并执行，才显示服务器渲染的标记，所以你的用户将会更快速地看到完整渲染的页面。通常可以产生更好的用户体验，并且对于那些「内容到达时间(time-to-content) 与转化率直接相关」的应用程序而言，服务器端渲染 (SSR) 至关重要。 
    权衡之处：
    1.开发条件所限。浏览器特定的代码，只能在某些生命周期钩子函数 (lifecycle hook) 中使用；一些外部扩展库 (external library) 可能需要特殊处理，才能在服务器渲染应用程序中运行。 
    2. 涉及构建设置和部署的更多要求。与可以部署在任何静态文件服务器上的完全静态单页面应用程序 (SPA) 不同，服务器渲染应用程序，需要处于 Node.js server 运行环境。 
    3.更多的服务器端负载。在 Node.js 中渲染完整的应用程序，显然会比仅仅提供静态文件的 server 更加大量占用 CPU 资源 (CPU-intensive - CPU 密集)，因此如果你预料在高流量环境 (high traffic) 下使用，请准备相应的服务器负载，并明智地采用缓存策略。
    
知识点三：SSR的基本原理
    SSR的基本原理就是我们在客户端第一次请求某个页面时，服务端会把当前页面的数据以及页面都请求过来，然后render渲染成完整的页面以后再返回给客户端，然后客户端在操作跳转其他路由页面时走的是仍然就是vue的单页流程，就不再走服务渲染。这样我们请求页面时就会有页面内容，爬虫就能爬取页面内容，也保留了单页切换页面时不用再向服务器请求页面的有点，又能保证首次有SEO网页优化。
    
```
<!DOCTYPE html><html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title></head>

<body>

    <div id="app">
        <div>this is home page</div>
    </div>
    <script type=text/javascript src=./static/js/bundle.js></script>
    
</body>

</html>
```
  怎么实现的呢？各位同学可能会有疑惑。其实关键精髓就在下面这张首次请求ssr服务器的返回页面： 
  ![600663f90c3b0c60441496a95d4d0515.png](en-resource://database/723:1)
  第一个重点 ：我服务端渲染的时候我把整个客户端的包塞进来，那么等我再次在页面操   作跳转页面时那走的就是客户端的路由。 
  第二个重点： 当我服务端取完数据更新完store状态机里面的状态以后，客户端并不知道服务端更新里那些store数据，这时服务端在渲染你的第一个页面时一并把store的数据赋值给window._INITIAL_TATE_。 客户端只需要添加一句取window._INITIAL_TATE_的代码即可取到服务端store状态。然后前后数据同步。
```
// 同步服务端信息
if (window.__INITIAL_STATE__) {
app.$store.replaceState(window.__INITIAL_STATE__)
}
```

知识点四：SSR的代码结构
说到这不得不说前后端代码结构：
  1.SSR既然采用的是服务端渲染，那一定需要服务端的代码。所以代码除了有客户端代码，还有服务端代码。
  2.但是服务端跟客户端代码有通用的代码部分。例如路由组建（前后台都是相同的路由组件。相同的store状态库。代码虽然都是相同但是前后端因为生成的实例不一致，所以也是不同的子类）
  2.客户端代码就是正常的vue打包后的代码，如果不进行chunk split 生成的也就是一个入口html一个js两个文件（上面图片重点的js就是说的这个）。服务端代码主要是启动服务，匹配路由，请求数据。这三块下面会详细说一下。
  ![31cd90821d44faf0d26b02b64fa0c10e.png](en-resource://database/725:1)
  
  下面我贴代码的时候了：
  package.json 
```
{
  "name": "ssr",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "server": "webpack --config ./webpack/webpack.server.js",
    "client": "webpack --config ./webpack/webpack.client.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "axios": "^0.16.0",
    "babel": "^6.23.0",
    "babel-plugin-transform-runtime": "^6.23.0",
    "babel-polyfill": "^6.26.0",
    "babel-preset-env": "^1.7.0",
    "body-parser": "^1.18.3",
    "compression": "^1.7.2",
    "express": "^4.15.4",
    "express-http-proxy": "^1.2.0",
    "gulp": "^3.9.1",
    "gulp-shell": "^0.6.5",
    "http-proxy-middleware": "^0.18.0",
    "less": "^3.0.4",
    "less-loader": "^4.1.0",
    "shell": "^0.5.0",
    "superagent": "^3.8.3",
    "vue": "^2.2.2",
    "vue-meta": "^1.5.0",
    "vue-router": "^2.2.0",
    "vue-server-renderer": "^2.2.2",
    "vue-ssr-webpack-plugin": "^3.0.0",
    "vuex": "^2.2.1",
    "vuex-router-sync": "^4.2.0"
  },
  "devDependencies": {
    "babel-core": "^6.26.3",
    "babel-loader": "^6.4.1",
    "babel-preset-es2015": "^6.24.1",
    "css-loader": "^0.28.4",
    "style-loader": "^0.18.2",
    "vue-loader": "^11.1.4",
    "vue-template-compiler": "^2.2.4",
    "webpack": "^2.7.0"
  }
}
```
 entry-server.js  
```
import { createApp } from '../src/main'

export default context => {
    return new Promise((resolve, reject) => {
        const app = createApp()

        // 更改路由
        app.$router.push(context.url)

        // 获取相应路由下的组件
        const matchedComponents = app.$router.getMatchedComponents()

        // 如果没有组件，说明该路由不存在，报错404
        if (!matchedComponents.length) { return reject({ code: 404 }) }

        // 遍历路由下所以的组件，如果有需要服务端渲染的请求，则进行请求
        Promise.all(matchedComponents.map(component => {
            if (component.serverRequest) {
                return component.serverRequest(app.$store)
            }
        })).then(() => {
            resolve(app)
        }).catch(reject)
    })

}
```

```
/* store.js */
import Vue from 'vue'import Vuex from 'vuex'import axios from 'axios'

Vue.use(Vuex)

export default function createStore() {
      let store =  new Vuex.Store({
            state: {
                  homeInfo: ''
            },
            actions: {
                  getHomeInfo({ commit }) {
                        return axios.get('http://localhost:8080/api/getHomeInfo').then((res) => {
                              commit('setHomeInfo', res.data)
                        })
                  }
            },
            mutations: {
                  setHomeInfo(state, res) {
                        state.homeInfo = res
                  }
            }
      })

      return store
}
```
  
```

/* main.js */
import Vue from 'vue'import createRouter from './route.js'import App from './App.vue'import createStore from './store'


// 导出一个工厂函数，用于创建新的vue实例export function createApp() {
    const router = createRouter()
    const store = createStore()
    const app = new Vue({
        router,
        store,
        render: h => h(App)
    })

    return app
}
```
```
/* server.js */
const exp = require('express')
const express = exp()
const renderer = require('vue-server-renderer').createRenderer()
const createApp = require('./dist/bundle.server.js')['default']


// 设置静态文件目录
express.use('/', exp.static(__dirname + '/dist'))


// 客户端打包地址const clientBundleFileUrl = '/bundle.client.js'


// getHomeInfo请求
express.get('/api/getHomeInfo', (req, res) => {
    res.send('SSR发送请求')
})


// 响应路由请求
express.get('*', (req, res) => {
    const context = { url: req.url }

    // 创建vue实例，传入请求路由信息
    createApp(context).then(app => {
        let state = JSON.stringify(context.state)

        renderer.renderToString(app, (err, html) => {
            if (err) { return res.state(500).end('运行时错误') }
            res.send(`
                <!DOCTYPE html>
                <html lang="en">
                    <head>
                        <meta charset="UTF-8">
                        <title>Vue2.0 SSR渲染页面</title>
                        <script>window.__INITIAL_STATE__ = ${state}</script>
                        <script src="${clientBundleFileUrl}"></script>
                    </head>
                    <body>
                        <div id="app">${html}</div>
                    </body>
                </html>
            `)
        })
    }, err => {
        if(err.code === 404) { res.status(404).end('所请求的页面不存在') }
    })
})


// 服务器监听地址
express.listen(8080, () => {
    console.log('服务器已启动！')
})
```
 
```
/* entry-client.js */
import { createApp } from '../src/main'


const app = createApp()

// 同步服务端信息if (window.__INITIAL_STATE__) {
      app.$store.replaceState(window.__INITIAL_STATE__)
}


// 绑定app根元素window.onload = function() {
       app.$mount('#app')
}
```

```

/* entry-server.js */
import { createApp } from '../src/main'

export default context => {
    return new Promise((resolve, reject) => {
        const app = createApp()

        // 更改路由
        app.$router.push(context.url)

        // 获取相应路由下的组件
        const matchedComponents = app.$router.getMatchedComponents()

        // 如果没有组件，说明该路由不存在，报错404
        if (!matchedComponents.length) { return reject({ code: 404 }) }

        // 遍历路由下所以的组件，如果有需要服务端渲染的请求，则进行请求
        Promise.all(matchedComponents.map(component => {
            if (component.serverRequest) {
                return component.serverRequest(app.$store)
            }
        })).then(() => {
            context.state = app.$store.state
            resolve(app)
        }).catch(reject)
    })

}
```
```
<!-- home.vue -->
<template>
  <div>
    home
    <div>{{ homeInfo }}</div></div></template>
<script>
    export default {
        serverRequest(store) {
            return store.dispatch('getHomeInfo')
        },
        mounted() {
            
        },
        computed: {
            homeInfo() {
              return this.$store.state.homeInfo
          }
      }
  }
</script>
<style scoped>
</style>
```
  代码下载地址：https://github.com/genggguanghao/ssrDemo.git
  
  详细的代码讲解：https://www.jianshu.com/p/c6a07755b08d
    

   
