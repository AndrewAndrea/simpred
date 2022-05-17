> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zhuoyue360.com](http://www.zhuoyue360.com/jsnx/67.html)

> 事先声明：本篇文章仅作为学术研究和安全交流使用，切勿用于商业用途。如因违反规定产生任何法律纠纷，本人概不负责。如果本篇文章影响到官方利益，请添加文末的作者微信号告知，本人会在第一时间将文章删除。...

**事先声明：**

**本篇文章仅作为学术研究和安全交流使用，切勿用于商业用途。如因违反规定产生任何法律纠纷，本人概不负责。如果本篇文章影响到官方利益，请添加文末的作者微信号告知，本人会在第一时间将文章删除。**

**【本文共计 5225 个字，阅读大约需要 11 分钟】**

**写在前面：**

爬虫界大山，这是其中一座。如果你认为我用的比喻有点言过其实，那么不妨看看下面的截图。之后你就会对我说的话深以为然，甚至还会觉得，它的难度被我严重低估了。

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-1639448442707111.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-1639448442707111.webp "图片")

2020 年 12 月 13 日，差不多就是去年的这个时候。我在沈阳远程面试了一家坐落于大连的互联网公司。公司的技术总监为了测试我的技术能力，就把一个名为 “中国海关进口商品查询网” 的站点，作为面试题目，**限期三天之内抓取 1w 条数据，开出的薪资是 8k**。当时有过大约两年工作经验的我，很自命不凡，一番折腾下，终于用了整整三十天的时间，才知道它的名字：**R 数第 5 代加密**。从此我心中也萌生出了一个至今都无法消除的疑问：他妈的，耍我玩呢？

转眼已经过去了一年，我一直无法对这件事释怀。同时，也把研究这项加密技术当成了我业余生活中势在必行的一件事，甚至列入有生之年系列。但好在，随着我在研究上的慢慢深入和知识上的层层储备，我终于在 2021 年 12 月 6 号 21 点 32 分整突破这项技术瓶颈，当时的心情和海因斯突破百米纪录心情一样：“原来那扇门是虚掩着的！”

故事要留给过去，但成长要用于分享。所以接下来，我将会把时间的指针拨回一年之前，以一个初学者的身份出现在 R 数面前。从对它的一无所知到的豁然开朗，从对它的闻风丧胆再到泰然自若。这是一个分析的过程，更像是一个渡劫的经历。但往往这种经历会伴随着一个不一样的开端，所以我们要

**0x01 先从无限 debugger 说起**

分析的网站是什么，对于我们无关紧要。因为 R 数是一个系列，而这个系列具体表现在于：你在打开**网络****分析工具**后立马就会引起你的警觉

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425241.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425241.webp "图片")

上述代码我们暂时不太了解它意义，但唯一能让你感受到不明觉厉的地方是，即使你释放断点，0.1s 过后它又会自动执行了这段代码，反反复复，永无止境。如果解决不了这个问题，那么调试工作就会难以为继。所以，必要时，我们要采取一些强硬的手段来对付它

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425242.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425242.webp "图片")

使用 **chrome** 断点工具 Never pause here （永不再此处停止）即可解决这一问题，但这也不过是权宜之计，因为只要你刷新网站，就会惊奇的发现，代码也会发生变化！而且每一次都**极度的混淆（也可以称之为动态混淆）**

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425253.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425253.webp "图片")

这些代码难以理解起来比较难，我们大可不必避重就轻，非要把它研究个所以然来。别忘了我们初心和目的：**抓取数据**

**0x02 分析请求**

这个环节对于我们必不可少，所以在解决掉 **debugger** 问题后，我们来看下它每一次接口请求的数据

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425254.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425254.webp "图片")

可以看到它的链接中携带一个 **MmEwMD** 的签名，同时它的 **cookie** 信息也看上去不是那么友善

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425255.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425255.webp "图片")

而且最让人不解的是，每一次请求，它的 **cookie** 竟然也会跟着变化！正常而言 **cookie** 是服务端与客户端建立唯一的会话标识。如果 **cookie** 发生变化就意味着服务端不会保存你的登录状态！带着这个疑问，我们看一下 **chrome** 的 **Application** 来更好的证实这一结论

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425256.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425256.webp "图片")

可以看到里面有很多关于 **cookie** 信息。这里有一个知识点，是 **CSDN** 山东大葱哥教我的：我们辨别 **cookie** 来源时，可以看 **httpOnly** 这一栏，如果有 √ 的是来自于服务端，如果没有 √ 的话是本地生成的。由此，我们结论得到证实：本地 **cookie** 确实存在变化。同时，我们也能想到让它变化的方法

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425257.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425257.webp "图片")

直观点来讲，官方**肯定有过给上述方法重新赋值的过程，而且还是存在一定的定时性**，至于赋的值如何生成，以及它定时的规律是什么，我们会在后续章节研究，**现在主要深挖它的特点**

**0x03 为什么是 202？**

出于好奇，我们尝试把它的 **cookie** 信息写死，然后用 **python** 程序对它的地址实现一次请求

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425258.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425258.webp "图片")

然后发现了一个更为惊奇的事情：它的**响应结果**和**响应状态**跟官方截然不同

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425259.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-16394484425259.webp "图片")

官方的响应状态是：**200**，而我们本地的响应状态是：**202**

为什么？

**0x03 抓包分析 cookie**

至此，我们可以总结出 R 数的一部分特点

1.  无限 **debugger** 会对调试造成干扰

1.  极强代码混淆会让你彻底丧失对它的阅读兴趣

1.  **cookie** 存在动态刷新机制而且每次请求都各不相同

1.  眼见并不一定为实，比如同样的请求会在两种不同的途径下呈现不同的结果

为了尽快的找出内部的玄机，我们可以借助 **fiddler** 进行抓包，**要相信你看到的所有不可思议的东西，都是服务端在始作俑者**。所以，我们不应该放过服务端所留下的任何线索。对了，抓包之前要把 **cookie** 清掉（这点很重要）

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252610.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252610.webp "图片")

通过抓包可以看到，同一个链接地址它一下子请求了两遍，但竟然出现了两个不同结果。下图是第一遍请求，我们所能获取到的信息

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252611.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252611.webp "图片")

在第二遍请求中，我们可以获取到这些信息。鉴于对请求的描述太蹩脚，我们就姑且把第一次遍请求叫做：初始化请求。第二遍请求叫做：列表页请求。

[![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "图片")

这里可能有点乱，我们先梳理一下：初始化请求 -> 获取服务端 **cookie** + 本地生成的 **cookie** -> 列表页请求 -> 获取请求接口的部分 **cookie** 我们可以试着画出个流程图

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252612.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252612.webp "图片")

经过这样分析，我们对 **cookie** 的机制有了新的认识。同时，我们可以把初始化请求时服务端返回 **cookie** 轻松拿到手，携带它去发送列表页请求。但由于携带的 cookie 信息不完整，我们并不能请求成功。所以要想成功的完成列表页请求，我们需要把重心放到——如何生成后半段 **cookie** 上，否则一直会是 **202**

**0x04 被隐藏的代码**

不管你相信与否，请求列表页后半段 **cookie** 的生成方法，就隐藏在初始化请求所返回的代码中。虽然这个链接没有以 **.js** 作为后缀，却包含着海量的 **js** 信息。而这些信息正是生成 **cookie** 的关键所在

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252613.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252613.webp "图片")

为了方便研究，我们可以把这段代码下载到本地，这样一来，就更能看清它的细节

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252614.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252614.webp "图片")

这里面注意两点，第一点：上述文件引用了一个名为 **c.FxJzG50F.dfe1675.js** 的文件，我们也需要把它下载到本地进行引用。第二点：编码 **charset** 需要改成 **UFT-8** 格式，不然的肯定报错

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252615.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252615.webp "图片")

**0x05 eval 的注册机制**

仿照上述操作，我们将下载到本地的代码文件用浏览器打开后，令人不可思议的一幕发生了，本地的代码竟然跟官方的代码出现了一致的格调：无限 **debugger**

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252616.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252616.webp "图片")

同时我们又发现了一个新的疑问：**这些代码从何而来？**无论在线上还是本地，我们都无法捕捉到它的踪迹。就像幽灵一般，没等你感觉到，就凭空出现在了你的面前。嗯... 这里有点故弄玄虚了哈。如果我们想要知道它的来源，其实也不是很难，可以在这份代码最开始的地方打上断点

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252617.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252617.webp "图片")

然后刷新浏览器后，根据它右侧的调用栈信息，对它进行定位追踪。

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252618.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252618.webp "图片")

然后切换到它生成前的一步，就会有一些新的发现：红色箭头所指的变量，是由字符串构建成的代码

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252619.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252619.webp "图片")

而调用它的函数竟然是 **eval**，换句话讲，我们感到疑惑代码都是由字符串拼接而成，然后通过 **eva**l 函数执行后变成了现在的这般形式，妙哉！

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252720.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252720.webp "图片")

**0x06 层级关系**

在对代码的运行机制有过简单了解后，我们就可以把它做为切入点更加深入的去探究它的经过。**同时，也要把耐心和兴趣结合起来，因为只有这样我们才能勇往直前又会乐在其中。**

如果有划分的概念存在，我们将这段代码分为三部分：

1. 赋值层（变量赋值）

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252721.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252721.webp "图片")

2. 应用层（各种函数）

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252722.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252722.webp "图片")

3. VM 层（一个大到令人窒息的死循环）

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252723.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252723.webp "图片")

**0x07 属性盲盒**

鉴于这种层级关系的存在，我们可以先从**赋值层**进行研究。顾名思义，代码在运行之初给很多变量进行了赋值操作，而且这些值基本属于 **javascript** 的内置函数，其作用暂且不明。但有一部分赋值操作，更他娘的让我们一头雾水

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252724.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252724.webp "图片")

当然，如果你想搞明白这件事情，我们可以利用断点来一探究竟

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252725.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252725.webp "图片")

上述这些属性储存在一个大数组中，展开之后，让我们看起来更加全面和具体。与此同时官方是利用索引的方式，对这些属性进行读取和操作。看起来就像一个盲盒一样，我们对它里面的内容一无所知。所以我们暂且把它称之为：**属性盲盒**

**0x08 拆箱**

盲盒里的东西对我们确实极具吸引力，但要想对他逐个分析却不太容易。不过我们可以借助 **indexOf** 索引工具，来对它一些关键的信息进行提取

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252826.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252826.webp "图片")

嗯，通过上述操作得知，那些所谓的 **cookie** 、**debugger** 、**setInterval** 等一些相关方法不是它没有引用，而是换一种方式对我们进行了一次误导。所以我们可以拨开迷雾，通过索引关系对他们逐一定位分析，比如：**cookie**

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252827.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252827.webp "图片")

可以看到与之 **cookie** 相关的共有 13 个结果，我们试着在其中一处打个断点分析一下，这样的话就能更好的去捕捉到运行动态

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252828.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252828.webp "图片")

如上图我们就能定位到 **cookie** 生成的最终位置，然后根据右侧调用栈的信息，我们还可以定位到它的初始位置

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252829.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252829.webp "图片")

由上图可见，**cookie** 是由一个 **VM** 层函数通过加工一个里面全是整型数字的数组而来

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252830.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252830.webp "图片")

而想要追溯到这些整型数字的来源，这个过程真的是太过复杂，这里面我只能介绍一下方法

**0x09 可以借鉴的分析方法**

因为在 **VM** 层有个索引，会决定代码每次进入到哪个流程去运行。根据这种变化特点，我们姑且称它为：**震荡索引**

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252931.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252931.webp "图片")

而 R 数有个特点，就是在流程中，每个分支都比震荡索引值大 1，比如：震荡索引值 = 255，那我们就可以去到 > 266 的流程去寻找流程。还有一点就是在 **VM** 顶层有很多为了未定义的变量，在 **VM** 运行的过程中都会渐渐产生变化

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252932.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252932.webp "图片")

我们也可以通过对他们插装的方式，对他们进行赋值关系进行输出，这样的话也会对我们的分析过程带来莫大的裨益

**0x10 metadata 闭包的缓存**

利用上一章所介绍的方法，我费尽周折对 **cookie** 的加密进行了全方位的跟踪。当然，涉及的流程很多，这里我就先不一一列举了。但我可以介绍一下它加密之前的都取了那些原始数据。一个是本地的 **localStorage** 另一个则是 **metadata** 的拆分区间。

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252933.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252933.webp "图片")

鉴于 **localStorage** 好理解也容易处理。接下来，着重研究一下 **metadata** 是怎么一回事儿

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252934.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252934.webp "图片")

如果细心的话，你会注意到，这个 **metadata** 每一次请求都会发生变化，而这种变化是随机无序且让人倍感迷惑的。虽然我没有探究过它的具体作用，但是有一点可以肯定，它的存在会跟 **cookie** 或者后面 **MmEwMD** 的签名计算都有很大关系。当然了官方还对它做了一个极其重要的操作：**拆分**

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252935.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252935.webp "图片")

如上图所示，我们就可以看到 **metadata** 被拆分后的结果。而且还有一点需要注意的是，这个拆分过程是以函数嵌套形式存在的，也就是闭包模式

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252936.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844252936.webp "图片")

所以遇到这种情况，我们一定要它先运行 **metadata** 才会加入缓存，不然的话你读不到 **metadata** 的拆分值

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253037.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253037.webp "图片")

但有个问题，我们把生成 **cookie** 的相关代码全部扣出来，是不是就可以模拟生成正确的 **cookie** 了？答案：**不能！**

**0x11 配套原则的随机变量**

先来看看我找出来计算 **cookie** 代码的运行结果

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253038.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253038.webp "图片")

虽然我没有验证过 **cookie** 是否有效，但我想说明的问题是，这套代码只适用于当前你停留的初始化页面，如果下一次发生新的初始化请求， **metadata** 被更换后就会出现一些异常

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253039.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253039.webp "图片")

后来，我发现原来是有个随机值决定了 **cookie** 的计算，**注意这个值是服务端返回的，有一定的随机性**。不过话说回来，在我们眼里这个值看似随机，但在服务端或许可能是存在一定的配套机制，只不过我们无从知晓而已

[![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "图片")

**由以上得出的结果，我们证明一件事：即便你计算出了 cookie 加密，但也仅限于当前的使用。一旦页面发生变化，你所有的努力都将付之一炬。**所以对于 **cookie** 的计算，我们能不能换一种方式去进行？或者说，我们能不能模拟出一个浏览器环境，让它自动拉取到 **cookie** ？答案是：可以的！

**0x12 jsdom 的应用**

谈起 **nodejs** 里的这个类库，可谓是集大成之作，也是模拟浏览器环境的不二之选。有兴趣的可以谷歌一下，功能很多我这里不过多的赘述。然后利用这种工具我们可以模拟官方一样的流程，将代码插入到 **jsdom** 模拟的浏览器环境中，让他自动拉取 **cookie**，问题就会迎刃而解，不会在枉费周折。

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253040.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253040.webp "图片")

这里的暂且不提供核心代码，原因是简单：**_* 保命要紧 *_。**但有几个地方特别需要我们注意一下**：**1. 关于 **python** **execjs** 类库 调用 **js** 问题

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253041.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253041.webp "图片")

可能是底层计算机制不一样，使用这种方法，计算的 **cookie** 不正确。

2. 使用 **express** 搭建服务实现不了功能持久化。具体原因是：把代码插入到 **jsdom** 环境后，缓存无法清除，这样第二次再次插入代码时，跟上一次插入的代码会产生冲突，导致什么也获取不到

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253042.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253042.webp "图片")

3. 结合 **node** 的 **minimist** 的命令行交互命令，使用 **python** 的 **os.popen** 直接调用会出现指令太长的问题（毕竟传入的是几千行代码和长度过千的 **metadata**）

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253043.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253043.webp "图片")

后来机缘巧合之下，我找到了这一问题的解决方案：我们可以把要传入的参数变成 **json** 的字符串形式，并用 **base64** 进行编码。最后，通过下面的指令进行传参

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253044.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253044.webp "图片")

当然别忘了，在 **node** 里面写个方法对 **base64** 编码格式的数据进行解码，然后以这种方式，我们确实能够完成列表页的请求并拿到接口请求的 **cookie**

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253145.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253145.webp "图片")

**0x13 签名与请求的寄生关系**

读到这里，或许有人会感到害怕和恐惧：写了将近 **5000** 字，才刚刚把 **202** 的请求状态给解决。那是不是研究它的签名，又得洋洋洒洒写下 **5000** 字？答案：当不会的（**我虽然能写但我会惜墨如金**[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253146.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253146.webp "图片")）！因为只要你认真阅读上述内容，就会断然的发现，其实研究签名跟研究 **202** 请求状态如出一辙。**只不过计算签名的代码是在列表页返回的**

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253147.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253147.webp "图片")

要想搞明白签名的计算方式，我们需要了解一下 **javascript** **hook** 机制。可以最简单的打印为例

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253148.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253148.webp "图片")

如上图可以看到，方法被 **hook** 后每次在调用 **console.log** 函数时，都会加入我的个人签名。同样的问题，我们也会想。它每次请求都会出现一个签名，那是不是官方也用了如法炮制了呢？

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253149.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253149.webp "图片")

确实有了不一样的发现。如果你还看不懂话，可以对比一下正常的

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253150.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253150.webp "图片")

也就是说，它内部的请求方法被重构了。我们可以切入进入去一探究竟

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253151.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253151.webp "图片")

如果加入断点，这个签名就不期而然的出现在我们面前

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253252.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253252.webp "图片")

也就是说每次接口请求都会触发计算签名的函数，然后将计算好的签名挂到请求链接上。下面是生成签名的方法

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253253.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253253.webp "图片")

**0x14 归档整理**

在发现签名的奥秘后，我们就可以继续利用 **jsdom** 来完成签名的计算。但有个问题也会顺势出现。就是说：你每次把动态的代码插入到 **jsdom** 环境中， **XMLHttpRequest** 确实被重构了。但最根本的问题我们还是无法解决：**拿不到签名而且还会抛出请求的异常**

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253254.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253254.webp "图片")

面对这类问题，我提供一个思路：再对 **XMLHttpRequest** 进行重构。把它的请求机制改为返回机制。只能提示到这里，仅此而已

最后附上一张效果图:

[![](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253255.webp)](https://xiaopang-1256027372.cos.ap-shenzhen-fsi.myqcloud.com//640-163944844253255.webp "图片")