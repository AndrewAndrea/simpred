> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_44504978/article/details/117790722?spm=1001.2014.3001.5502)

​**声明：本文只作学习研究，禁止用于非法用途，否则后果自负，如有侵权，请告知删除，谢谢！**

前言：我解释一下 hook 是什么玩意  
hook 的原意是钩子。  
我理解替换或者拦截原有方法去修改和处理。  
1. 怎么去替换原来的方法  
这里我自己写一个方法去替换。实例

```
function myfunction(x,y){
if(x>y){
return "是原来的方法";
}else{
return "方法参数被修改了";
}
myfunction(2,1)//原来方法的执行结果
"是原来的方法"
var xxx=myfunction //这里开始替换原来的方法，吧myfunction辅助给xxx
myfunction=function (x,y)//这里再修改原来的myfunction这方法，一样的给他传两个值，在后面的函数里面去修改这两个值
{
	var x=5,y=6;//这个位置就是把xy值改了，覆盖原来的方法的值
return xxx (x,y) //这个再把修改过后的值传给原来的myfunction方法。


}

myfunction(1,2) //这里是hook过后的方法执行结果，和原来执行传的值一样，但是返回值却变了
"方法参数被修改了"


```

上面讲这一种就是 hook 替换原来的方法去改变它的执行结果，传值一样但是获取的结果不一样。

还有两个方法其实我个人感觉都差不多的，这两种都是拦截  
1.Object.defineProperty  
2.Porxy  
使用方法比较简单，太多人说这两个方法了我就不说了，这里我重点讲的一个，刚刚说了一种替换。这里我再讲一个实例。还是自己写的代码  
这里讲这么去 hook 内置的函数，eval，这个函数是现在很多混淆的都爱用的。把这个 hook 对我们分析代码帮助很大，

```
eval("function ccc(){return 22;};ccc()")
//我这里比较简单，传的都是明文，要是在混淆里面传的肯定都是混淆的变量

```

首先我们要在代码执行之前在浏览器控制台打上我们的 hook 代码

```
var sss=eval; //把eval赋值给sss，sss是自己建的全局变量
eval=function (x){//一样的处理方法，重写这个方法
	debugger; //这里发现调用就让他断下来
console.log(x)//这里把值打印出来
return sss(x)//返回原来的eval方法
}

```

下面我截图跑一遍你们看看效果  
![](https://img-blog.csdnimg.cn/20210610205119273.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUwNDk3OA==,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20210610205119264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUwNDk3OA==,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20210610205133884.png#pic_center)  
其他方法也是一样，其他自己脑补吧。欢迎关注我的公众号哦，谢谢观看。  
![](https://img-blog.csdnimg.cn/20210206005427264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUwNDk3OA==,size_16,color_FFFFFF,t_70#pic_center)