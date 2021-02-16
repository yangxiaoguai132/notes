### axios实现excel文件下载
一般下载都是通过a标签的href这个属性来实现，但是如果是使用axios这个方式的话，应该要怎么样做呢？

可以按照下面这个做。

这里一定要记得设置responseType，一定要记得记得记得记得
```
axios
.post(
    this.baseUrl+"/exportUser”, 
    { 
        admin: “",
        keys: “",
        keyword:this.keyword,
    },
    {
        responseType: ‘blob'
    }
).then(function(response) {
    var blob = new Blob([response.data])
    var downloadElement = document.createElement('a');
    var href = window.URL.createObjectURL(blob); //创建下载的链接
    downloadElement.href = href;
    downloadElement.style.display = ’none';
    downloadElement.download = '用户数据.xlsx'; //下载后文件名
    document.body.appendChild(downloadElement);
    downloadElement.click(); //点击下载
    document.body.removeChild(downloadElement); //下载完成移除元素
    window.URL.revokeObjectURL(href); //释放掉blob对象
    console.log(response);
})
.catch(function(error) {
    console.log(error);
});
```