## 一、安装 axios
```bash
npm install --save
```
## 二、创建axios.js
<p>`axios.js` 文件封装方法是为了让业务调用方可以更少的操作 axios 的 api ，甚至不需要了解 axios 的工作原理，直接使用封装的方法即可，这样可以形成一套统一的请求规范（如： 想要post请求就直接this.$http.post(url, params)，想要 get 请求就调用this.$http.get(url, params)…）</p>

```js
// axios.js
import axios from 'axios'

const http = { // 只写项目中用到的常用请求方法，需要更多方法可以自行添加
  get (url, data, config=null) { // get请求
    let instance = axios.create()
    config.params = data;
    return instance.get(url, config)
  },
  post (url, data, config=null) { // post 请求
    let instance = axios.create()
    return instance.post(url, {}, {
      ...config,
      params: data
    })
  },
  postJson (url, data, config=null) { // post json数据
    let instance = axios.create()
    instace.defaults.headers.post['Content-Type'] = 'application/json'
    instance.defaults.timeout = 0
    return instance.post(url, data, config)
  },
  upload (url, data, config=null) { // 文件上传
    let instance = axios.create()
    instace.defaults.headers.post['Content-Type'] = 'multipart/form-data'
    instance.defaults.timeout = 0
    let formData = new FormData()
    let keys = Object.keys(data)
    keys.forEach(key => {
      formData.append(key, data[key])
    })
    return instance.post(url, formData, config)
  }
}

export default http
```
## 三、axios 配置
### 全局配置：
```js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```
### 实例默认配置：
```js
// 创建实例时修改配置
const instance = axios.create({
  baseURL: 'https://api.example.com'
});

// 实例创建之后修改配置
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```
### 请求方法传入config参数进行配置

```js
axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.options(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
```
<p>配置项通过一定的规则合并，request config > instance.defaults > 系统默认，优先级高的覆盖优先级低的。</p>

```js
// axios.js
import axios from 'axios'

const isProd = process.env.NODE_ENV === 'production'; // 判断是否是生产环境
axios.defaults.baseURL = isProd ? '/backend/path' : '/api/backend/path' // 生产环境默认接口地址与后端同域，开发环境使用node代理请求地址/api
axios.defaults.timeout = 10000
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded'

const http = { // 只写项目中用到的常用请求方法，需要更多方法可以自行添加
  get (url, data, config=null) { // get请求
    let instance = axios.create()
    config.params = data;
    return instance.get(url, config)
  },
  post (url, data, config=null) { // post 请求
    let instance = axios.create()
    config.params = data;
    return instance.post(url, {}, config)
  },
  postJson (url, data, config=null) { // post json数据
    let instance = axios.create()
    instace.defaults.headers.post['Content-Type'] = 'application/json'
    return instance.post(url, data, config)
  },
  upload (url, data, config=null) { // 文件上传
    let instance = axios.create()
    let formData = new FormData()
    let keys = Object.keys(data)
    keys.forEach(key => {
      formData.append(key, data[key])
    })
    instace.defaults.headers.post['Content-Type'] = 'multipart/form-data'
    return instance.post(url, formData, config)
  }
}

export default http
```
## 四、拦截器
<p>请求拦截：可以在请求发出之前进行登录校验、loading展示等处理</p>
<p>响应拦截：可以在服务器响应被then或catch前处理响应结果（如：loading关闭，响应数据格式化等）</p>

```js
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    handleError()
    return Promise.reject(error)
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 处理响应数据
    return response;
  }, function (error) {
    // 处理报错信息
    if (error.response)
      handleError(error.response)
    else
      Message.error(error.message)
    return Promise.reject(error)
  });
// 处理报错信息
function handleError(response) {
  let actions = new Map([
    [
      401,
      () => {
        Message.error("没登录不给看")
        store.dispatch('ToLogin')
      }
    ],
    [
      403,
      () => {
        Message.error("管理员说你没权限")
      }
    ],
    [
      404,
      () => {
        try {
          Message.error("接口走丢了")
        } catch (e) {
          console.log(e)
        }
      }
    ],
    [
      500,
      () => {
        Message.error("你的网络又开小差了")
      }
    ],
    [
      502,
      () => {
        Message.error("服务器断片了，稍后片刻")
      }
    ],
    [
      503,
      () => {
        Message.error("服务器进化中，少侠稍待")
      }
    ],
    [
      "default",
      () => {
        Message.error(response.data)
      }
    ]
  ]);
  let action = actions.get(response.status) || actions.get("default")
  action();
}
```

## 五、在项目中使用
#### 1. 新建http.js，请求都写在这里

```js
// http.js
import axios from './axios'

const request = {
   getMenu(params) {
     return axios.post('/getMenu', params)
   },
   getUserInfo(params) {
     return axios.get('/getUserInfo', params)
   },
   // 在此处新增请求接口方法...
}

export default {
   install (Vue, options) {
     Vue.prototype.$http = request
   }
}
```

#### 2. 在项目的入口文件引入 http.js
```js
// main.js
import Vue from 'vue'
import Http from './request/http.js'

Vue.use(Http)
```

#### 3. 具体页面中调用接口:

```js
// user.vue 省略template和style
export default {
  data () {
    return {}
  },
  methods: {
    getUserInfo () {
      let params = { id: 'xxxxxxxx123' }
      this.$http.getUserInfo(params)
      .then(({data}) => { // 返回数据处理
      })
      .catch(err => { // 自定义错误处理，请求如果报错会先执行响应拦截中的错误处理方法后再执行此方法
      })
    },
    getMenu () {
      let params = { id: 'xxxxxxxx123' }
      this.$http.getMenu(params)
      .then({ data }) => {})
    }
  }
}
```
