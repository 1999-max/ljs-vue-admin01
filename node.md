<!-- 基于vue-element-template-master开发admin -->
# 一、项目了解
  1. 先开这些文档
  [手摸手，带你用 vue 撸后台 系列一（基础篇）](https://juejin.cn/post/6844903476661583880)
  [手摸手，带你用 vue 撸后台 系列四(vueAdmin 一个极简的后台基础模板)](https://juejin.cn/post/6844903486241374221#heading-7)
  [手摸手，带你优雅的使用 icon](https://juejin.cn/post/6844903517564436493)

  2.模仿`vue-element-admin`项目，实现一个简易的项目
  主要使用到得技术 vue-cli vue2 vue-router vuex element-ui scss axois
  实现以下功能:
      1) 划分好项目结构
      2) 登录页面
      3) 路由和侧边栏
      4) mock

  3.src文件：
    (1)api & views
    建议根据业务模块来划分views，并且 将views 和 api 两个模块一一对应，从而方便维护。
    <!-- views文件夹下面放文件夹（文件夹里面有组件），api里面放同名JS文件（也有单独JS文件--单独API模块） -->
    (2)components 
      components放置的都是全局公用的一些组件(文件夹+index.vue)，如上传组件，富文本等等。
      一些页面级的组件建议还是放在各自views文件下(index.vue)，方便管理。
    (3)vuex
      dispacth(actions)用于异步操作修改state,commit(mutations)用于同步操作来修改state;
      异步和同步的区别在于异步更新必须通过派发action来更新mutation,而同步更新可以直接提交mutation来更新state;
      action不能直接修改state值,action 必须通过提交mutation来更新state,只有mutation才能更新state.
      <!-- this.$store.dispatch() ：含有异步操作，例如向后台提交数据，写法：this.$store.dispatch(‘action方法名’,值) -->
      <!-- this.$store.commit()：同步操作，，写法：this.$store.commit(‘mutations方法名’,值) -->

    (4)ESLint
      <!-- 代码规范 ———— 先不用！！ -->
      首先安装eslint插件,回到 VSCode 进行扩展设置，依次点击 文件 > 首选项 > 设置 打开 VSCode 配置文件,添加如下配置:
      "files.autoSave":"off",
      "eslint.validate": [
        "javascript",
        "javascriptreact",
        "html",
        { "language": "vue", "autoFix": true }
      ],
      "eslint.options": {
          "plugins": ["html"]
      }
      这样每次保存的时候就可以根据根目录下.eslintrc.js你配置的eslint规则来检查和做一些简单的fix.
      <!-- https://github.com/PanJiaChen/vue-element-admin/blob/master/.eslintrc.js -->

    (5)alias
      <!-- vue-cli新版本中默认 src文件夹 可以直接使用 @ -->
      <!-- node_modules@vue\cli-service\lib\config\base.js   里可以找到，其他别名就需要在vue.config.js 中自行配置。 -->
      const path = require("path");
      function resolve(dir) {
        return path.join(__dirname, dir);
      }
      module.exports = {
        chainWebpack: (config) => {
          config.resolve.alias
            .set("@", resolve("./src"))
            .set("assets", resolve("./src/assets"))
        },
      };
      <!-- 提醒css引用的时候记得加 “~” 符号，不然无法显示 <img src="~assets/img/tabbar/category.svg" alt="">  -->
      该项目 ———— vue.config.js
        resolve: {
          alias: {
            '@': resolve('src')
          }
        }

  4.封装 axios -- @/utils/request.js
    <!-- 由于我们的项目需要在不同环境下进行运行(开发，生产，测试等)，这避免我们需要多次的去切换请求的地址以及相关的配置，
    vue-cli2是可以直接在config文件中进行配置的，但是vue-cli4和vue-cli3已经简化了，没有config文件. -->
    (1)首先我们在根目录新建3个文件，分别为.env.development，.env.production，.env.test
      <!-- 注意文件是只有后缀的 -->
      .env.development 模式用于serve，开发环境，就是开始环境的时候会引用这个文件里面的配置
      .env.production模式用于build，线上环境。
      .env.test 测试环境
      然后分别在文件内写上：
          <!-- /dev-api 是需要请求API-->
        VUE_APP_BASE_API = '/dev-api';
      接着更改package.json文件：
          "scripts": {
            "dev": "vue-cli-service serve",
            "test": "vue-cli-service serve --mode test",
            "build": "vue-cli-service build",
            "build:test": "vue-cli-service build --mode test",
            "lint": "vue-cli-service lint"
          },
          <!-- 运行的时候走的package -->
      当需要用到`VUE_APP_BASE_API`变量，可以用“ process.env.VUE_APP_BASE_API ”进行取值：
      <!-- @/utils/request.js里面 ，创建axios实例-->
        const service = axios.create({
          baseURL: process.env.VUE_APP_BASE_API,   
          timeout: 5000 // request timeout
        })
    (2) axios拦截器 
        // 添加一个请求拦截器
          axios.interceptors.request.use(function (config) {
              <!-- // 在发送请求之前做些什么 -->
              return config;
            }, function (error) {
              <!-- // 对请求错误做些什么 -->
              return Promise.reject(error);
            });
          // 添加响应拦截器
          axios.interceptors.response.use(function (response) {
              <!-- // 对响应数据做点什么 -->
              return response;
            }, function (error) {
              <!-- // 对响应错误做点什么 -->
              return Promise.reject(error);
            });

    (3)token
      当输入用户名及密码，登录成功后，后台会返回一个token，在之后发送的请求都要带上这个token，因为后台设置了拦截，
      如果token一致，则允许访问，否则请求不成功，所以要设置请求头到token。
      用法：加一个http request拦截器
          - 通过window.localStorage.getItem("accessToken") 来获取token的value
          - 通过config.headers.accessToken = token;将token放到请求头发送给服务器，放在请求头中 
                axios.interceptors.request.use(function (config) {
                    <!-- window.localStorage.getItem("accessToken") 获取token的value -->
                    let token = window.localStorage.getItem("accessToken")
                    if (token) {
                        <!-- 将token放到请求头发送给服务器,将token-key放在请求头中 -->
                        config.headers.accessToken = token;     
                        <!-- //也可以这种写法 -->
                        <!-- // config.headers['accessToken'] = Token; -->
                        return config;
                    }
                },function (error) {
                    return Promise.reject(error);
                });
      vue token设置：
        1、接口没有自定义 token头部，用以下的写法
        const token = getToken()
        config.headers['Authorization'] = `Bearer ${token}`
        2、如果接口有定义token，例如定义了X-Token'，用以下的写法
        config.headers['X-Token'] = getToken()
        <!-- 让每个请求携带token--['X-Token']为自定义key 请根据实际情况自行修改 -->

    (4)code
      50008:非法的token; 50012:其他客户端登录了;  50014:Token 过期了; 20000：正确；
      if (res.code === 50008 || res.code === 50012 || res.code === 50014) {
          MessageBox.confirm('你已被登出，可以取消继续留在该页面，或者重新登录', '确定登出', {
            confirmButtonText: '重新登录',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
              store.dispatch('FedLogOut').then(() => {
                location.reload();
                <!-- 为了重新实例化vue-router对象 避免bug -->
              }); //then-store
          }); //then-if
      }//if

    (5)使用
        import request from '@/utils/request'

        export function getInfo(params) {
          return request({
            url: '/user/info',
            method: 'get',
            params
          });
        }

  5.前后端的交互问题
    (1)swagger -- 接口规范化
    (2)mock.js
    (3)iconfont
      element-ui 默认的icon不是很多，这里要安利一波阿里的iconfont简直是神器.
      https://www.iconfont.cn/

  6.router-view
    different router the same component vue
    在 router-view上加上一个唯一的key，来保证路由切换时都会重新渲染触发钩子了
    <router-view :key="key"></router-view>
    computed: {
        key() {
            return this.$route.name !== undefined? this.$route.name + +new Date(): this.$route + +new Date()
        }
    }

  7.打包优化
    npm install
    npm run build:prod 

# 二.登录页面
  1.目标：不同的权限对应着不同的路由，同时侧边栏也需根据不同的权限，异步生成。
  2.实现登录和权限验证的思路
    -登录： 当用户填写完账号和密码后向服务端验证是否正确，验证通过之后，服务端会返回一个token，拿到token之后
          （我会将这个token存贮到cookie中，保证刷新页面后能记住用户登录状态），前端会根据token再去拉取一个 user_info 的接口来
           获取用户的详细信息（如用户权限，用户名等等信息）。
    -权限验证：通过token获取用户对应的 role，动态根据用户的 role 算出其对应有权限的路由，
              通过 router.addRoutes 动态挂载这些路由。 

  3.布局
    vue-element-admin/src/layout/components/AppMain.vue
    <!-- 里面是 首页（不包括侧边栏） -->
    (1)transition 标签：Vue 的内置动画标签
      作用：在 [插入] / [更新] / [移除] DOM 元素时，在合适的时候给元素添加样式类名（配合 CSS 样式使用，实现动画效果）
      注意：transition 标签只能包含 1 个元素；transition 包裹的标签需要设置 v-show / v-if 属性控制元素的显示
      name 属性：用于自动生成 CSS 动画类名；如果 transition 标签元素没有设置 name 属性，则对应的动画类名为 v-XXX
                如果设置了 name 属性，则对应的动画类名为 属性值-XXX
    (2)<keep-alive :include="cachedViews">:
        在组件切换过程中将状态保留在内存中，防止重复渲染DOM，减少加载时间及性能消耗，提高用户体验性。
        include - 字符串或正则表达式，只有名称匹配的组件会被缓存。
    (3)store
      <!-- cachedViews() {
        return this.$store.state.tagsView.cachedViews
      }, -->
     - vue中的全局变量 this.$store.state 的取值: store目录下的 index.js
     - 










  4.登录
    随便找一个空白页面撸上两个input的框，一个是登录账号，一个是登录密码。再放置一个登录按钮。我们将登录按钮上绑上click事件，
    点击登录之后向服务端提交账号和密码进行验证。 这就是一个最简单的登录页面。
    <!-- vue-element-admin/src/views/login/index.vue -->
    



