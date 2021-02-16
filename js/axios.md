### axios简介
axios 是一个基于Promise 用于```浏览器```和```nodejs```的HTTP客户端。
>本质上也是对原生XHR的封装，只不过它是Promise的实现版本。

##### 特征：
1.从浏览器中创建XMLHttpRequest对象发送请求
2.支持 Promise API
3.客户端支持防止CSRF
4.提供了一些并发请求的接口（重要，方便了很多的操作）
5.在node.js使用http对象，创建http请求
6.拦截请求和响应
7.转换请求和响应数据
8.取消请求
9.自动转换JSON数据
10.跨平台的请求，浏览器可以支持xhr发送，服务端可以支持http请求发送。
11.批量发送多个请求，实现：
```
function getUserAccount() {
  return axios.get('/user/12345');
}
 
function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}
axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // 两个请求现在都执行完成
  }));

```
##### 源码
根据process或者是其他的来判断是node还是浏览器，来决定底层请求使用什么实现。

* node，其实还是http模块的server来处理的
* 浏览器，里面其实是new Promise，里面使用了一个xhr对象，里面监听了xhr的属性，或者是onreadystatechange这种函数里面将状态改成resolve或者是reject


##### 跨域解决方式：
* 服务器端设置：```Access-Control-Allow-Origin:*```，允许跨域的域名。
* 服务端设置：```Access-Control-Allow-Headers":"Authorization,Origin, X-Requested-With, Content-Type, Accept"```来设置允许跨域的http请求头(具体根据你的代码来定)。
类似的还有```"Access-Control-Allow-Methods":"GET,POST”```。

##### 使用方法
一般引入的时候，都愿意在开头使用下面这句话：import axios from axios

>创建实例如下：
```
const instance = axios.create({
  timeout: 60000
});
```

>修改请求头：
![avatar](https://github.com/yangxiaoguai132/images/blob/master/axios%E4%BF%AE%E6%94%B9%E8%AF%B7%E6%B1%82%E5%A4%B4%E7%A4%BA%E4%BE%8B.png)

>添加拦截器:
![avatar](https://github.com/yangxiaoguai132/images/blob/master/axios%E6%B7%BB%E5%8A%A0%E6%8B%A6%E6%88%AA%E5%99%A8%E7%A4%BA%E4%BE%8B.png)

##### axios拦截器原理
下面是官网的拦截器示例：
```
axios.interceptors.request.use(
  function(config) {
    // 在发送请求之前做些什么
    return config;
  },
  function(error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  }
);

// 添加响应拦截器
axios.interceptors.response.use(
  function(response) {
    // 对响应数据做点什么
    return response;
  },
  function(error) {
    // 对响应错误做点什么
    return Promise.reject(error);
  }
);
```
内部使用了promise这个机制，
```
// 先构造一个对象 存放拦截器
axios.interceptors = {
  request: [],
  response: []
};

// 注册请求拦截器
axios.useRequestInterceptor = (resolved, rejected) => {
  axios.interceptors.request.push({ resolved, rejected });
};

// 注册响应拦截器
axios.useResponseInterceptor = (resolved, rejected) => {
  axios.interceptors.response.push({ resolved, rejected });
};

// 运行拦截器
axios.run = config => {
  const chain = [
    {
      resolved: axios,
      rejected: undefined
    }
  ];

  // 把请求拦截器往数组头部推
  axios.interceptors.request.forEach(interceptor => {
    chain.unshift(interceptor);
  });

  // 把响应拦截器往数组尾部推
  axios.interceptors.response.forEach(interceptor => {
    chain.push(interceptor);
  });

  // 把config也包装成一个promise
  let promise = Promise.resolve(config);

  // 暴力while循环解忧愁
  // 利用promise.then的能力递归执行所有的拦截器
  while (chain.length) {
    const { resolved, rejected } = chain.shift();
    promise = promise.then(resolved, rejected);
  }

  // 最后暴露给用户的就是响应拦截器处理过后的promise
  return promise;
};
```

调用：
```
axios.useRequestInterceptor(config => {
  return {
    ...config,
    extraParams1: "extraParams1"
  };
});

axios.useRequestInterceptor(config => {
  return {
    ...config,
    extraParams2: "extraParams2"
  };
});

axios.useResponseInterceptor(
  resp => {
    const {
      extraParams1,
      extraParams2,
      result: { code, message }
    } = resp;
    return `${extraParams1} ${extraParams2} ${message}`;
  },
  error => {
    console.log("error", error);
  }
);

(async function() {
  const result = await axios.run({
    message: "message1"
  });
  console.log("result1: ", result);
})();

输出result1: extraParams1 extraParams2 message1
```

拦截器机制参考文章：[前端网红框架的插件机制全梳理](https://mp.weixin.qq.com/s/HhjDQgK8FcPiy5BEfZ7lYQ)

##### 携带cookie
默认人家是不携带cookie的，要想让人家携带东西，那么就是需要将这个进行一下配置，请这样配置：
![avatar](https://github.com/yangxiaoguai132/images/blob/master/axios%E6%90%BA%E5%B8%A6cookie%E7%A4%BA%E4%BE%8B.png)
记住这个1，要在create方法执行之前配置。
参考文章，[Ajax允许跨域携带cookie](https://www.jianshu.com/p/045c737466f9)



##### get请求与post请求参数携带

get请求浏览器表现：
![avatar](https://github.com/yangxiaoguai132/images/blob/master/get%E8%AF%B7%E6%B1%82%E6%B5%8F%E8%A7%88%E5%99%A8%E8%A1%A8%E7%8E%B0.png)


>注意：get请求的参数要写在params里面
![avatar](https://github.com/yangxiaoguai132/images/blob/master/axios%E7%9A%84get%E8%AF%B7%E6%B1%82%E5%8F%82%E6%95%B0%E7%A4%BA%E4%BE%8B.png)

>注意：post请求的参数要写在data里面
![avatar](https://github.com/yangxiaoguai132/images/blob/master/axios%E7%9A%84post%E8%AF%B7%E6%B1%82%E7%A4%BA%E4%BE%8B.png)

如果不按照上面两个方法进行参数配置，例如如果是get请求，却将参数放在了data里面，请求不会出错，但是会出现一个问题，就是参数是无法传递给服务端的。





