---
title: 借助axios的拦截器实现Vue.js中登陆状态校验的思路
date: 2018-03-02 11:13:10
tags:
     - vue.js
     - axios
---

<div class="RichContent-inner RichContent-inner--collapsed"><span class="RichText CopyrightRichText-richText" itemprop="articleBody"><p>首先，后台系统中，不一定每一个页面都要进行校验权限，所以我们要通过路由的meta标签对需要做校验的路由页面做好标记：<!-- more --></p><div class="highlight"><pre><code class="language-text">{
  path: '/userInfo',
  name: 'userInfo',
  meta: {
    requireAuth: true,  // 该路由项需要权限校验
  }
  component: userInfo
}, {
  path: '/userList',
  name: 'userList',    // 该路由项不需要权限校验    
  component: userInfo
}
</code></pre></div><p>之后，我们可以定义一个路由守卫，每次路由跳转，我们都来做一下权限校验，参考如下代码 </p><div class="highlight"><pre><code class="language-text">router.beforeEach((to, from, next) =&gt; {
  if(to.meta.requireAuth) { // 如果路由项需要权限校验
    /* 
      从Vuex拿出token码，说明已登陆
     （前端的token可伪造，所以这并不是完全靠谱，后面继续说）
    */
    if (store.state.token) { 
      next() // 正常跳转页面
    } else {
      next({
        path: '/login',
        query: {redirect: to.fullPath}  
        /* 将跳转的路由地址作为参数带给登陆页，登录成功后跳转回该页面 */
      })
    }
  } else { // 如果不需要权限校验，直接进入路由页面
      next();
  }
})
</code></pre></div><p>然而，单纯这么来做并不靠谱，首先，token可以用户自己伪造，其次，有可能token在前端存在，但是后端中，这个token已经失效。</p><p>所以，这么做权限校验并不准确，那么接下来，我们可以在发请求的时候，通过后端的一层校验，进一步确保权限状态的准确性。</p><p>这一步，要用到axios里面的拦截器，所以以下代码建立在项目使用axios的基础上。<b>axios的拦截器（Interceptors），默认配置（axios.defaults）还有 axios.create() 创建出的实例实际上在封装请求时非常有用，如果你没接触过，赶紧到官方文档看一下！</b></p><p>我们先来看，在每次请求时，我们进行如下的拦截，这么做的意思是，如果前端有token，我每次都把token给后端，后端拿到这个token后会到后端再进行一次权限校验，然后把校验结果在接口中反馈：</p><div class="highlight"><pre><code class="language-text">axios.interceptors.request.use( requestConfig =&gt; {
  if (store.state.token) {
    config.headers.Authorization = `${store.state.token}`;
  }
  return requestConfig;
}, err =&gt; {
  return Promise.reject(err);
});
</code></pre></div><p>然后我们再来看，对应一定要再写一个response拦截器，处理响应数据。如果后端拿到前端的token，发现token失效，我们要求后端接口返回401的返回码，这样，前端就可以这么处理了：</p><div class="highlight"><pre><code class="language-text">axios.interceptors.response.use( response =&gt; {
    return response;
  }, error =&gt; {
    if (error.response) {
      switch (error.response.status) {
        case 401:  
          /* 
            返回 401 表示前端的token已失效
            当然，你也可以和后端也定其他的方式来表示token失效
            需要前端清除Vuex中的token，页面跳转到登陆页
          */ 
          store.commit(types.LOGOUT);
          router.replace({
            path: 'login',
            query: {
              redirect: router.currentRoute.fullPath
            }
          })
      }
    }
    return Promise.reject(error.response.data) //返回接口返回的错误信息
});
</code></pre></div><p>借助axios的拦截器，实现权限校验其实还是挺方便的。</p><p></p><p></p></span><!-- react-empty: 40129 --></div>