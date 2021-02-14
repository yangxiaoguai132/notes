### 首先先科普一下http的请求方法们：
* get：通常用于请求某些资源
* post：发送数据给服务器
* head：请求资源的头部信息, 并且这些头部与 HTTP GET 方法请求时返回的一致. 该请求方法的一个使用场景是在下载一个大文件前先获取其大小再决定是否要下载, 以此可以节约带宽资源
* options：用于获取目的资源所支持的通信选项
* put：用于```新增资源```或者使用请求中的有效负载替换目标资源的表现形式
* DELETE: 用于删除指定的资源
* PATCH: 用于对资源进行部分修改
* CONNECT: HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器
* TRACE: 回显服务器收到的请求，主要用于测试或诊断

### GET和POST的相同之处
1.都是http1.0就存在的请求方法

2.都是http请求，底层都是TCP/IP，所以get和post都是tcp链接，GET和POST能做的事情是一样一样的。你要给GET加上request body，给POST带上url参数，技术上是完全行的通的。 

### GET和POST的区别
1.GET在浏览器回退时是无害的，而POST会再次提交请求。
2.GET产生的URL地址可以被Bookmark，而POST不可以
3.GET请求会被浏览器主动cache，而POST不会，除非手动设置。
4.GET请求只能进行url编码，而POST支持多种编码方式。
5.GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
6.GET请求在URL中传送的参数是有长度限制的，而POST么有。
7.对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
8.GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
9.GET参数通过URL传递，POST放在Request body中。
10.**GET产生一个TCP数据包；POST产生两个TCP数据包。**
>对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

### 为何get请求不在request body带参数，post请求不在url上面带参数
如果说tcp就像是汽车，为了保证传输数据的秩序性，http规定了，当执行GET请求的时候，要给汽车贴上GET的标签（设置method为GET），而且要求把传送的数据放在车顶上（url中）以方便记录。如果是POST请求，就要在车上贴上POST的标签，并把货物放在车厢里。

当然也是可以在get请求的request body李曼放参数，但是会比较傻，也可以在post的url请求中放参数，也会比较傻。

**既然是这样，那么，区别里面的参数限制又是哪里来的呢?**
是不同浏览器与服务器限制的。因为传递数据也是有成本的，所以会限制单次数据运输量。
业界不成文的规定是，（大多数）浏览器通常都会限制url长度在2K个字节，而（大多数）服务器最多处理64K大小的url。超过的部分，恕不处理。如果你用GET服务，在request body偷偷藏了数据，不同服务器的处理方式也是不同的，有些服务器会帮你卸货，读出数据，有些服务器直接忽略，所以，虽然GET可以带request body，也不能保证一定能被接收到哦。


### PUT和POST方法的区别
1.PUT方法是幂等的：连续调用一次或者多次的效果相同（无副作用），而POST方法是非幂等的。
2.除此之外还有一个区别，通常情况下，PUT的URI指向是具体单一资源，而POST可以指向资源集合。
举个例子，我们在开发一个博客系统，当我们要创建一篇文章的时候往往用POST https://www.jianshu.com/articles，这个请求的语义是，在articles的资源集合下创建一篇新的文章，如果我们多次提交这个请求会创建多个文章，这是非幂等的。
而PUT https://www.jianshu.com/articles/820357430的语义是更新对应文章下的资源（比如修改作者名称等），这个URI指向的就是单一资源，而且是幂等的，比如你把『刘德华』修改成『蔡徐坤』，提交多少次都是修改成『蔡徐坤』


### PUT和PATCH的区别：PUT整体更新与PATCH局部更新
PUT和PATCH都是更新资源，而PATCH用来对已知资源进行局部更新。
比如我们有一篇文章的地址https://www.jianshu.com/articles/820357430,这篇文章的可以表示为:
```
article = {
    author: 'dxy',
    creationDate: '2019-6-12',
    content: '我写文章像蔡徐坤',
    id: 820357430
}
```
当我们要修改文章的作者时，我们可以直接发送PUT https://www.jianshu.com/articles/820357430，这个时候的数据应该是:
```
{
      author:'蔡徐坤',
      creationDate: '2019-6-12',
      content: '我写文章像蔡徐坤',
      id: 820357430
}
```
这种直接覆盖资源的修改方式应该用put，但是你觉得每次都带有这么多无用的信息，那么可以发送PATCH https://www.jianshu.com/articles/820357430，这个时候只需要:
```
{
    author:'蔡徐坤',
}
```


### 参考文章：
[99%的人理解错 HTTP 中 GET 与 POST 的区别](https://mp.weixin.qq.com/s/hh-7VDGC9h5igDaRaqfvhQ)