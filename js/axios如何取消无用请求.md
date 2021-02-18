### 问题场景

* 用户频繁切换筛选条件去请求数据，初次的筛选条件数据量大。用的时间比较多。后面的筛选条件的数据量小。导致后面请求的数据先返回。内容先显示在页面上。但是等一段时间，初次(或者前面)的请求数据返回了， 会覆盖后面的请求的数据。这就导致了筛选条件和内容不一致的情况。

* 用户点击了一次提交按钮，接口没有很快响应，导致页面没办法做逻辑语句判断的提示。用户觉得可能没提交上，便会快速又点了按钮几次。如果后端没有去重的判断，就会导致数据中有很多条重复的数据。

### 解决思路
**我们把目前处于pending的请求存储（假如我们放在一个数组）起来。每个请求发送之前我们都要判断当前这个请求是否已经存在于这个数组。如果存在，说明请求重复了，我们就在数组中找到重复的请求并且取消。如果不存在，说明这个请求不是重复的，正常发送并且把这个请求api添加在数据中，等请求结束之后删除数组中的这个api**


### axios 如何取消请求

* 第一种方法

通过axios.CancelToken.source生成取消令牌token和取消方法cancel。
```
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // 处理错误
  }
});

axios.post('/user/12345', {
  name: 'new name'
}, {
  cancelToken: source.token
})

// 取消请求 (消息参数是可选的)
source.cancel('Operation canceled by the user.');
```

* 第二种方式

通过传递一个 executor 函数到 CancelToken 的构造函数来创建 cancel token。
```
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收一个 cancel 函数作为参数
    cancel = c;
  })
});

// 取消请求
cancel();
```

### 开始封装
采用axios 如何取消请求的第一种方式。

声明一个Map。用来存储每个请求的 标识 和 取消的函数
```
// 存储每个请求的标识和取消的函数
this.pendingAjax = new Map();
```

自定一个字段来让用户自己决定是否需要取消重复的请求
```
// 是否取消重复的请求
cancelDuplicated = false,
```

自定一个字段来让用户是否有全局的统一的设置重复标识的函数。如果没有设置全局的统一的函数，则默认是请求的method 和url作为重复的标识
```
// 生成重复标识的方式
duplicatedKeyFn,
this.duplicatedKeyFn = isFunction(duplicatedKeyFn) ? duplicatedKeyFn : (config) => `${config.method}${config.url}`;
```

添加请求：
```
/**
 * 将请求添加到pendingAjax
 * @param {Object} config
 */
addPendingAjax(config) {
  // 是否需要取消重复的请求
  if (!this.cancelDuplicated) return
  const veryConfig = config.veryConfig || {};
  const duplicatedKey = JSON.stringify({
    duplicatedKey: veryConfig.duplicatedKey || this.duplicatedKeyFn(config),
    type: REQUEST_TYPE.DUPLICATED_REQUEST,
  });
  config.cancelToken = config.cancelToken || new axios.CancelToken((cancel) => {
    // 如果pendingAjax中不存在当前请求，添加进去
    if (duplicatedKey && !this.pendingAjax.has(duplicatedKey)) {
      this.pendingAjax.set(duplicatedKey, cancel);
    }
  });
}
```

这里面我们可以使用duplicatedKey字段来让用户对单一请求自定义重复的标识。或者可以使用一个函数duplicatedKeyFn统一的让用户自定义重复的标识


删除请求
```
/**
   * 从pendingAjax中删除请求
   * @param {Object} config
   */
  removePendingAjax(config) {
    // 是否需要取消重复的请求
    if (!this.cancelDuplicated) return
    const veryConfig = config.veryConfig || {};
    const duplicatedKey = JSON.stringify({
      duplicatedKey: veryConfig.duplicatedKey || this.duplicatedKeyFn(config),
      type: REQUEST_TYPE.DUPLICATED_REQUEST,
    });
    // 如果pendingAjax中存在当前请求, 取消当前请求并将其删除
    if (duplicatedKey && this.pendingAjax.has(duplicatedKey)) {
      const cancel = this.pendingAjax.get(duplicatedKey);
      cancel(duplicatedKey);
      this.pendingAjax.delete(duplicatedKey);
    }
  }
```
 
封装好了， 我们在哪里使用呢？肯定是在请求开始之前和请求完成之后使用。

在请求之前
```
// 拦截请求
this.axios.interceptors.request.use((config) => {
  // 在请求开始之前检查先前的请求
  this.removePendingAjax(config);
  // 将当前请求添加到pendingAjax
  this.addPendingAjax(config);
  // ...
});
```

在请求完成之后去掉该请求

```
// 拦截响应
this.axios.interceptors.response.use(response => {
  removePending(response)
  return response
}, error => {
  // ...
})
```

到现在已经完成了该有的功能， 但是取消请求的错误我们不该返回给用户。所以：
```
(err) => {
// 类型是否为重复请求
let isDuplicatedType;
try {
  const errorType = (JSON.parse(error.message) || {}).type
  isDuplicatedType = errorType === REQUEST_TYPE.DUPLICATED_REQUEST;
} catch (error) {
  isDuplicatedType = false
}
if (isDuplicatedType) return;
}
```
我们在请求完成之后的err里面做一个判断，判断如果当前请求是取消的类型，我们就不返回给用户错误的提示信息。

参考文章：
[axios取消无效请求](https://mp.weixin.qq.com/s/nDPM6HzLKkAIqmW91I9W-A)
  
  




