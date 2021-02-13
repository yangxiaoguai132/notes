## GraphQL

### 参考文章：
[https://developer.ibm.com/zh/articles/wa-using-graphgl-to-customize-data-structure/]



### 简介
查询语言，开发者通过声明其所需要的数据格式，向 GraphQL Service 发起请求并取得对应的数据。

#### 未引入GraphQL时候的架构
在传统的客户端/服务器通信过程中，客户端向服务器发送网络请求，服务器收到请求后执行相应操作并返回数据。
![avatar](https://github.com/yangxiaoguai132/images/blob/master/WX20210213-171320%402x.png)

引入 GraphQL 后，客户端不再与服务器直接通信，而是通过 GraphQL Service 获取数据。下图展示了引入 GraphQL 的系统结构。
![avatar](https://github.com/yangxiaoguai132/images/blob/master/WX20210213-170300%402x.png)

下图展示了从多个 service 请求特定数据时，引入 GraphQL 的前后对比。
![avatar](https://github.com/yangxiaoguai132/images/blob/master/WX20210213-170309%402x.png)

与传统的客户端/服务器通信模式相比，GraphQL 的引入为整个系统增加了一个中间层，屏蔽了前后端的具体数据结构。其主要优势包括以下几点：

1.定制所需的数据 ：客户端可以定制自己想要的数据结构。通过在请求中声明所需的数据结构，从 GraphQL Service 中获取想要的数据。Service 返回的数据结构将与客户端的请求完全一致，从而减少冗余数据的传输。在不使用 GraphQL 的情况下，返回的数据格式不受客户端控制。

2.单个请求获取多个资源 ：在 GraphQL 中，客户端不再关心请求的来源，而是直接将需要的资源写入请求字段。在不使用 GraphQL 的情况下，当需要从不同的 Service 取得数据时，客户端开发者需要对不同的 Service 发起请求。

但是这种情形下其实一个restful api的http接口也是可以搞定的，但是，有一种情形，就是后端会乐意返回所有数据给前端，但是前端却偏偏不用那么多，所以这种情形，后端给了很多无用的数据给前端，会浪费网络请求。引入graphql确实不一样的，前端可以从后端一堆数据中选择要哪些数据，也可以定制返回的数据结构。

3.结构即文档 ：GraphQL 具有友好的调试界面，开发者可直接在该界面中查询 GraphQL 提供的服务。这种数据结构即文档的显示方式，使开发者能够方便快捷地查找所需要的数据类型。

4.订阅功能 ：GraphQL 提供订阅功能，使得客户端能够监听数据变化，让 GraphQL Service 能够主动将变动的数据推送至客户端，实时在界面上进行显示。



