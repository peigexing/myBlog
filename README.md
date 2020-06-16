@[TOC](项目优化记录)

## 1. 该项目性能提升方案
1. 对于vue，elementui这些框架采用**CDN**的方式引入项目，使得项目的体积缩小，优化了首屏加载。
2. 对于axios，router这些库也采用**CDN**方式引入。
3. 资源压缩,其中**关闭sourcemap**，包的体积减少将近一半，sourcemap是为了方便线上调试用的，因为线上代码都是压缩过的，导致调试极为不便，而有了sourcemap，就等于加了个索引字典，出了问题可以定位到源代码的位置。 但是，这个玩意是每个js都带一个sourcemap，有时sourcemap会很大，拖累了整个项目加载速度，为了节省加载时间，我们将其关闭掉。就这一句话就可以关闭sourcemap了，很简单。
![](https://imgkr.cn-bj.ufileos.com/6196fefa-5136-4bb1-bfa4-9859f81316a7.png)
4. 使用gzip压缩，文件体积缩小，优化了首屏加载。

5. 缓存方面，对于js和css采用了**强缓存**的缓存方式，时间为1年。对于html，以及数据库数据采用**协商缓存**方式，方便了数据的更新。
6. 细节方面，搜索框采用了**防抖策略**，因而用户若还在输入中，则会取消之前的请求。



## 优化打包体积

  将vue，elementui，axios，采用cdn的方式引入，体积从9m多变成5m多，打包时设置不生成map文件，体积变成1.7m
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407084229287.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407084410405.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk2NDE0OA==,size_16,color_FFFFFF,t_70)
对应的首屏加载的时间也从
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407084630331.png)
变成![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407084651542.png)


使用gzip压缩，首屏渲染从10s左右降低到5s左右


将静态资源设置为强缓存，时间为1年
```js
app.use(express.static(path.join(config.publicPath, 'dist'), { maxAge: 60 * 1000 * 60 * 24 * 365 }))
```

### 用户体验提升

- 搜索框实现搜索联想-采用防抖提升性能

```js
watch:{
    keyWord(val){
        if (val){
            this.deBounce(this.getSearchListBykeyWord,1000)
        }else{
            this.searchList = []
        }

    },
}

onClickOfSearchItem(searchItem){
    this.keyWord =  searchItem
    this.$nextTick(()=>{
        this.searchList = []
    })
},
async getSearchListBykeyWord(){
    let {status,data:{titleList}} = await this.$http.get(encodeURI(BASE_URL+'/api/article/getTitleListByKeyWord?keyWord='+this.keyWord))
    console.log(titleList);
    this.searchList = titleList
},
// 防抖函数
deBounce: (function () {
    let timer = 0;
    return function(callback, ms) {
        clearTimeout(timer);
        timer = setTimeout(callback, ms);
    };

})()
},
```
