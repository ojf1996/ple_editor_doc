# How to develop a component

I will try to describe serval steps to construct a component as well as **handling router, ajax request,  and communication with other component.** 



## issue1: develop a simple login page

## overview

we start from a login page, which look similar to signup.

![](..\photo\HowTo\fig1.JPG)

![](..\photo\HowTo\fig3.JPG)

as you can see, this page can be divided into two part, the navbar and the loginBox. So we can create two directories, namely navbar and loginBox in component directory.

![](..\photo\HowTo\fig2.JPG)



**There is no basic rule** for how to organize your project. But I suggest you organize you source code like **when the project is not too large**:

```
├── config                     // 配置相关
└──  src                       // 源代码
    ├── api                    // 所有请求
    ├── assets                 // 主题 字体等静态资源
    ├── components             // 全局公用组件
    ├── icons                  // 项目所有 svg icons
    ├── lang                   // 国际化 language
    ├── mock                   // 项目mock 模拟数据
    ├── router                 // 路由
    ├── store                  // 全局 store管理
    ├── utils                  // 全局公用方法
    ├── views                  // view
    ├── App.vue                // 入口页面
    └── main.js                // 入口 加载组件 初始化等
```



## navbar

we start from a navbar. As you can see, there is a **language selector** and a **router-link in the navbar**, so we create a **component directory** under navbar directory:

```
├──  navbar
│   └── components
│   │	├──  LanguageSelector.vue
│   │	├──  xxx.vue
│   │	└──  xxx.vue
│   ├── xxx.vue
│   └── LoginNavbar.vue
└──  view
    └── Login
        └── index.vue
```

I organize the navbar like this because there may be **more than one** navbars but they may **share some common components** and it is easy to manage in such structure.

**There is no rule** of **how to give name  to the component**. You can call whatever you like, but please make it as clear as possible.



### languageSelector

the code of LanguageSeletor is [here](./src/components/navbar/components/LanguageSelector.vue).

I will not cover how to write a single file component for vue here. More detail, please see [here](<https://vuejs.org/v2/guide/>). r.



### use the languageSelector

next, we can put languageSelector as well as other html elements together, creating a navbar.

**First**, we must import them, like:

```
import LanguageSelector from '@/components/Navbar/components/LanguageSelector'
```

Here ''@'' means alias, I have configure it in webpack.base.conf.js

```
module.exports = {
	...
	resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      '@': resolve('src')
    }
  }
}
```

So here it refers to the src directory. More detail, see [here](<https://webpack.js.org/configuration/resolve/>)

**Second,** we need to put them together, like:

```vue
<template>
  <div class="navbar">
    <div class="right-menu">
      <LanguageSelector class="right-menu-item"/>
      <div class="right-menu-item">
        <router-link :to="isSignIn?'/signup': '/login'">
          {{ isSignIn?$t("common.signup"):$t("common.signin") }}
        </router-link>
      </div>
    </div>
  </div>
</template>
```

**Then**, we may add some style to beautify it.

```css
<style rel="stylesheet/scss" lang="scss" scoped>
.navbar {
  height: 50px;
  overflow: hidden;
}
...
</style>
```

**Last but not least,** we should complete the **computer attribute** to decide whether it is in ''/login' or '/signup' so the  content of the navbar will changes **as the route change**.

```js
computed: {
  isSignIn() {
    return this.$route.path === '/login'
  }
}
```

computed attribute is similar to watcher, see [here](<https://vuejs.org/v2/guide/computed.html>)



## loginBox

**First**, We need to write about the html structure, like:

```vue
<template>
  <div class="loginBox">
    <Form ref="form" :model="form" :rules="formRules" class="formWrapper">
      <FormItem prop="user">
        {{ $t('common.userName') }}
        <Input v-model="form.user" size="large" type="text" placeholder="Username">
        <Icon slot="prepend" type="ios-person-outline"/>
        </Input>
      </FormItem>
      <FormItem prop="password">
        {{ $t('common.password') }}
        <Input v-model="form.password" size="large" type="password" placeholder="Password">
        <Icon slot="prepend" type="ios-lock-outline"/>
        </Input>
      </FormItem>
      <FormItem>
        <Button type="primary" @click="handleSubmit('form')">
          {{ $t('common.signin') }}
        </Button>
      </FormItem>
    </Form>
  </div>
</template>
```

**Then**, we can add style here.

```css
<style rel="stylesheet/scss" lang="scss">
.loginBox {
  padding:24px 16px 24px 16px;
  background: #fff;
  border-radius: 10px;
  height: 280px;
  width: 384px;
  .formWrapper{
    width:80%;
    margin-left:10%!important;
    margin-right:10%!important;
  }
}
</style>
```



### handling ajax

The most important part of login is to communicate with backend. But it is a **asynchronous** operation.

In the past, we may use jquery to handle it, like:

```
$.ajax({
  type: "POST",
  url: url,
  data: data,
  success: success,
  dataType: dataType
});
```

But as the number of callback increases, you may face code like:

```js
connection.query(sql, (err, result) => {
    if(err) {
        console.err(err)
    } else {
        connection.query(sql, (err, result) => {
            if(err) {
                console.err(err)
            } else {
                ...
            }
        })
    }
})
```

**Callback hell,** 23333.

**In order to make life more easier**, I try to make it more simple.



#### configure host

we can configure host for all api in config/dev.env.js or config/prod.env.js

```
module.exports = {
  ....
  BASE_API: '"https://api-prod"'
}
```



#### hanlde ajax with axio

The axio is recommended to handle ajax request for vue. So I try to create a request module.

```
import axios from 'axios'

const service = axios.create({
  baseURL: process.env.BASE_API, // api 的 base_url
  timeout: 5000 // request timeout
})

service.interceptors.response.use(
  response => response,
  error => {
    console.log('err' + error)
    return Promise.reject(error)
  }
)

export default service
```

You can customize your behavior of a request, like adding attribute in http headers.



#### create a request

create a login request in login.js

```
├── config                     // 配置相关
└──  src                       // 源代码
    └── api                    // 所有请求
        └──  login.js 
```

```js
import request from '@/utils/request'

export function loginByUsername(data) {
  return request({
    url: '/login',
    method: 'post',
    data
  })
}
```



#### use api

now we can make ajax request in a much easier way with es7 syntax async/await.

in loginBox/index.vue

```js
methods: {
    handleSubmit(name) {
      this.$refs[name].validate(async(valid) => {
        if (valid) {
          try {
            const data = {
              userName: this.form.user,
              password: this.form.password
            }
            const result = await loginByUsername(data)
            ...
          } catch (err) {
            this.$Message.error('Fail!')
          }
        } else {
          this.$Message.error('Fail!')
        }
      })
    }
}
```

No more worry about callback, bravo!





## put them together

Now we can construct a login page with components above.

```js
<script>
import Navbar from '@/components/Navbar/loginNavbar'
import LoginBox from '@/components/LoginBox/index'

export default {
  name: 'Login',
  components: { Navbar, LoginBox },
  data() {
    return {}
  }
}
</script>
```

```vue
<template>
  <div class="loginContainer">
    <Layout>
      <Header>
        <Navbar/>
      </Header>
      <Content>
        <div class="contentWrapper">
          <div class="content">
            <div class="text">
              <h1> {{ $t("login.plcEditor") }}</h1>
              <p>{{ $t("login.description") }}</p>
            </div>
            <div class="loginWrapper">
              <LoginBox/>
            </div>
          </div>
        </div>
      </Content>
      <Footer/>
    </Layout>
  </div>
</template>
```



## put them in router

However, we still can not see this page because vue now do not "know" it. We have to put them in the vue-router in router/index.js.

```js
import Vue from 'vue'
import Router from 'vue-router'
Vue.use(Router)

export const constRouterMap = [
  ...,
  {
    path: '/login',
    component: () => import('@/views/login/index')
  }
]

export default new Router({
  routes: constRouterMap
})
```



## mock data

Though we have complete the front end, what if we do not have a backend yet?

We can use mock.js to handle this, which acts like a backend but in fact there is no backend.

### usage

 **First,** you need to write some function which return some data like backend does.

```
└──  mock                      
    ├── xxx.js                   
    ├── login.js 
    └── index.js 
```

under src/mock directory, add some src code about the api you mock, like login.js to imitate as a backend interface to login service.

```js
export default {
  loginByUsername: () => {
    return {
      message: 'ok',
      token: 'lalaland',
      userName: 'jimmy'
    }
  },
  signup: () => {
    return {
      message: 'ok'
    }
  }
}
```

Module which return data of which the structure is the same as the real backend return.

**Then**, add them in index.js

```js
if (process.env.ENV_CONFIG === 'dev' && process.env.USE_MOCK) {
  Mock.mock(/\/login/, 'post', loginAPI.loginByUsername)
  Mock.mock(/\/signup/, 'post', loginAPI.signup)
}
```

The parameter of Mock.mock like below:

```
Mock.mock(url, method, interceptor)
```



 ### turn mock off

But what if we have a real backend then?

The answer is simple, turn them off in conf.js

```js
module.exports = {
  NODE_ENV: '"development"',
  ENV_CONFIG: '"dev"',
  BASE_API: '"https://api-dev"',
  USE_MOCK: 'true'
}
```

set USE_MOCK to **false**, change BASE_API to **the real host**, then you can develop with a real backend.

