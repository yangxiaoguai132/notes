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
