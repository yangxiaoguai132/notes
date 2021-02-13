## GraphQL

### 参考文章：
>[使用 GraphQL 定制数据结构](https://developer.ibm.com/zh/articles/wa-using-graphgl-to-customize-data-structure/)

>[GraphQL和RESTful的比较](https://developer.ibm.com/zh/articles/wa-using-graphgl-to-customize-data-structure/)

>[面对极度复杂的前后端业务场景，使用 GraphQL 正确的姿势](https://cloud.tencent.com/developer/article/1367400)

>[GraphQL 在酒店系统上的实践](https://mp.weixin.qq.com/s/Pmut13GYP-kwR2xm8fH-7Q)


### 简介
查询语言，开发者通过声明其所需要的数据格式，向 GraphQL Service 发起请求并取得对应的数据。

### 未引入GraphQL时候的架构
在传统的客户端/服务器通信过程中，客户端向服务器发送网络请求，服务器收到请求后执行相应操作并返回数据。
![avatar](https://github.com/yangxiaoguai132/images/blob/master/WX20210213-171320%402x.png)

引入 GraphQL 后，客户端不再与服务器直接通信，而是通过 GraphQL Service 获取数据。下图展示了引入 GraphQL 的系统结构。
![avatar](https://github.com/yangxiaoguai132/images/blob/master/WX20210213-170300%402x.png)

下图展示了从多个 service 请求特定数据时，引入 GraphQL 的前后对比。
![avatar](https://github.com/yangxiaoguai132/images/blob/master/WX20210213-170309%402x.png)

与传统的客户端/服务器通信模式相比，GraphQL 的引入为整个系统增加了一个中间层，屏蔽了前后端的具体数据结构。其主要优势包括以下几点：

1.定制所需的数据 ：客户端可以定制自己想要的数据结构。通过在请求中声明所需的数据结构，从 GraphQL Service 中获取想要的数据。Service 返回的数据结构将与客户端的请求完全一致，从而减少冗余数据的传输。在不使用 GraphQL 的情况下，返回的数据格式不受客户端控制。

2.单个请求获取多个资源 ：在 GraphQL 中，客户端不再关心请求的来源，而是直接将需要的资源写入请求字段。在不使用 GraphQL 的情况下，当需要从不同的 Service 取得数据时，客户端开发者需要对不同的 Service 发起请求。这种情形，pc还好处理，但是app的话，是需要等到所有数据都获取到之后才可以渲染的。

```但是这种情形下其实一个restful api的http接口也是可以搞定的，但是，有一种情形，就是后端会乐意返回所有数据给前端，但是前端却偏偏不用那么多，所以这种情形，后端给了很多无用的数据给前端，会浪费网络请求。引入graphql确实不一样的，前端可以从后端一堆数据中选择要哪些数据，也可以定制返回的数据结构。```

3.结构即文档 ：GraphQL 具有友好的调试界面，开发者可直接在该界面中查询 GraphQL 提供的服务。这种数据结构即文档的显示方式，使开发者能够方便快捷地查找所需要的数据类型。

4.订阅功能 ：GraphQL 提供订阅功能，使得客户端能够监听数据变化，让 GraphQL Service 能够主动将变动的数据推送至客户端，实时在界面上进行显示。


那么相比restful api，总结下来，graphql的优势就是：
* 数据定制困难
* 文档手动维护
* 冗余字段传输



### graphql在去哪酒店应用

#### 解决的问题
##### 数据定制困难：
服务最早都是基于PC端而设计的，而随着时代的发展，移动端的各个终端变得越来越丰富，包括现在的 app、小程序、touch 等。所以如果不是拆分服务，我们注定要面临多端适配的一个演进过程。这其中一个典型例子，就是对于同一接口，不同端的需求可能是不同的。拿我们酒店系统举例，对于酒店列表页展示，在 PC 端和 APP 端我可以把一家酒店的很多信息都放到列表中展示，每一家酒店就会包含比较完整的酒店模型的字段，然而在一些情况下，比如小程序的这种场景下，可能我需要每家酒店只展示图片，报价，名称三个字段，客户端这种多变的需求是非常常见的，但是我们的接口都是同一个。
```
hotelInfo: {
    name: "酒店",
    price:232,
    imgUrl:"img.jpg",
    tags:[{
        name:"亲子家庭"
    }],
    score:4.2,
    rank:"XX酒店排名第X名"
}
```
它将会把酒店模型下的所有字段直接返回给客户端。

而对于小程序端，我们就需要增加一个参数，判断请求来源是小程序，再去将模型中不显示的字段单独删除，才能达到这种字段级的控制。伪代码如下图：
```
if ( source === "XIAOCHENGXU" ) {
    hotelInfo.tags = undefined;
    hotelInfo.score = undefined;
    hotelInfo.rank = undefined;
}
```

假设现在我的小程序改版了，我新版本可能又想展示四个字段，老版本还是只展示三个，这时候我们就面临了版控问题。

作为服务端，我们有两个选择，不去管是新老版本的差异，冗余的返回四个字段，这样带来的好处就是开发简单，只增不减，没有风险。而另一种选择就是加一段这样的代码：

```
if ( source === "XIAOCHENGXU" ) {
    hotelInfo.tags = undefined;
    if( version <120 ){
        hotelInfo.score = undefined;
    }
    hotelInfo.rank = undefined;
}
```

这样针对不同的版本，我们就可以保证返回不同的定制字段了，好处就是不会有冗余数据的传输。坏处也显而易见，让服务端代码的可读性逐渐下降，凌乱松散，可维护性也会越来越低。

##### 多请求拼接：
PC 端酒店详情内，主要包括了房型、交通、详情、点评这四类资源，我们目前是通过两个接口拼接出来的。而在 APP 端，同样的酒店详情页，我们因为瀑布流的布局，可能除了以上内容，又多返回了周边推荐、住客秀两块资源。
对于这种不同资源拼接在同一个页面的需求。基于 restful 的接口，我们通常有两种处理方式：
> 在客户端发多次请求，我们可能会在原基础上，再增加两次请求，分别请求住客秀资源，以及周边推荐资源，再对数据进行处理。
```
// 方法1:
/api/detail {交通,点评}
/api/detailprice {房型,详情}
/api/clientShow {住客秀}
/api/recommendation {周边推荐}
```

> 在服务端针对接口去做定制，也就是根据不同的端，去变换我们某一接口的返回内容，在服务端去拼接不同资源，在 APP 的情况下，我们去拼接这两个资源，而这两个资源本身可能也是要在后端发两次请求而得来。
```
// 方法2:
/api/detail {交通，点评}
/api/detailprice
if( source === "APP" ){
    return {房型,详情,住客秀,周边推荐}
}
return {房型,详情}
```
这种方式的好处就是客户端减少了发请求的次数，但是它同时使得原本可以无关系的资源出现了强耦合，与 restful 设计时的初衷实际上相违背，在后续功能迭代和开发中，带来了隐患。比如我们如果修改了周边推荐里面的一些内容，也许他就会直接导致 detaiPrice 这个接口也需要有一个为了透传而产生的工作量。同时，这种做法也让 detailPrice 变得越来越臃肿复杂，提高了这个接口后续维护的困难。





