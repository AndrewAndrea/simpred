> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/zzU-hI9lSgNdPuYzQv-dUg)

**关注它，不迷路。**

 ![](http://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR0ibiaRA80DQAxzIicVWCTicOOws8h74IkXGmOTHOmGDFnxr9j6prDmyRu78pZNoPfSuoFqUSXXUm9meg/0?wx_fmt=png) ** 菜鸟学 Python 编程 ** Python 编程学习，JS 逆向分析。 127 篇原创内容  公众号

*   本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！
    

一. 插件地址
-------

```
https://github.com/cilame/v_jstools

```

安装到浏览器步骤请参考这篇文章:  

[工具推荐 | 一款爬虫界的神兵利器，值得拥有](http://mp.weixin.qq.com/s?__biz=MzAwNTY1OTg0MQ==&mid=2647563166&idx=1&sn=6330407705c77366581d74be50a3088c&chksm=83237630b454ff2697105b3a3418c8aebb525b200d461a6db8a43e87dabfd4ab601758d83fd3&scene=21#wechat_redirect)  

二. 插件配置
-------

‍如图，点击 打开配置按钮:  

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1ibDv3URxdELOibnXy4osRjW4E4mTibARC7db3cb1NXhQ3L5crcdRfp9Y6XjPCX5f8dL7yCUTeGzs4Q/640?wx_fmt=png)

按图进行如下设置:  

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1ibDv3URxdELOibnXy4osRjWNfQxGWpVOqSdTI8hLibSGicyGgYZbTdsrYkj0XpW1lMDEDzVorKBQszQ/640?wx_fmt=png)

三. 实战案例
-------

```
'aHR0cHM6Ly93d3cuemhpaHUuY29tL3NlYXJjaD90eXBlPWNvbnRlbnQmcT1weXRob24='

```

来看看我们需要的加密参数:  

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1ibDv3URxdELOibnXy4osRjWF7myttt0QkrZud5I0dHXwebfczKUNMKsicMbib9YUBYZgZokjYoNgebQ/640?wx_fmt=png)

等网站加载完毕后，我们再点开这个插件，选择 **生成临时环境** 命令:  

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1ibDv3URxdELOibnXy4osRjWMP7iaw7ssh6iaA5nDJEhj1gXVkDvPkzd7iaicHBvIavgRUrMc0vgKU2njQ/640?wx_fmt=png)

点击后，**会有个弹框提示已复制到剪切板**。这样，临时环境就帮我们生成好了，新建一个 js 文件 v.js, 将生成的环境复制过来进来并保存。  

我们再将**这个参数加密的代码也复制进来，保存到一起**，具体请参考这篇文章: [某乎请求头签名算法分析](http://mp.weixin.qq.com/s?__biz=MzAwNTY1OTg0MQ==&mid=2647563340&idx=1&sn=229487d7b232f882b990637c33e06c48&chksm=832371e2b454f8f44389ed8fda8c7916554c5df3ecde638242cdd88ba78d9a92cbacb26ae8ef&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1ibDv3URxdELOibnXy4osRjWtibaEP5VTicD0icE6MhtGYyb95tYz07GfzIfltr5aLdrxT3C5JK55YEBA/640?wx_fmt=png)

在加密函数处打上断点，记录实参和结果。  

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1ibDv3URxdELOibnXy4osRjW5dRzcf9wPeVdiajdGb9b6QRcfx7l43FFNogt3Lvvrnaynic4ia4Kk1jQg/640?wx_fmt=png)

实参: '7861d84e110af123f2fc8c05ff38601f'；

结果: 'a8O0QQuy28xYcR28f0NBkQe0kXtYUuY0mLxq66L0S7Yp'

在 v.js 中构造好实参，并打印结果:  

```
console.log(b('7861d84e110af123f2fc8c05ff38601f'));

```

保存后运行:  

![](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR1ibDv3URxdELOibnXy4osRjWLkHlrjVAAban88R2are3juU1rErnFSlzsqC0YUzdNmIabiaK66iaOu3w/640?wx_fmt=png)

经过仔细比对后可以发现，和网页上面的结果是一模一样的。非常的 nice。  

基本就靠一些 CV 操作，就把结果弄出来了，可以说非常的简单。

如果你不想打印上面的日志，可以将这个函数进行改写:  

```
var v_console_log = function(){{}}

```

这个插件试用了几天，已经解决了很多网站的加密，几乎可以说 js 逆向有手就行，当然，你需要一些 js 逆向分析的功底。毕竟它只是帮你补环境。  

**注意，这个并不是万能的，因此有效网站如果不行的话，可能需要想其他的办法了哈。**  

今天的内容就介绍到这里，感谢阅读。