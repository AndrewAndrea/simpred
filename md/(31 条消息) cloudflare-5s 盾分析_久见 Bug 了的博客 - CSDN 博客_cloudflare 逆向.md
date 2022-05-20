> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Ig_thehao/article/details/122966953)

### 文章目录

*   [前言](#_8)
*   [一、流程分析](#_17)
*   *   [第一个请求](#_26)
    *   [第二个请求](#_33)
    *   [第三个请求](#_51)
    *   [第四个请求](#_58)
    *   [第五次请求](#_118)
*   [二、注意事项](#_135)
*   [总结](#_146)

前言
==

5s 盾逆向解析，参考链接  
[https://mp.weixin.qq.com/s/efloBirboVfH2hK3cNoU5A](https://mp.weixin.qq.com/s/efloBirboVfH2hK3cNoU5A)  
[https://mp.weixin.qq.com/s/Bv8v7kbjX5NWWzdwCnwFHg](https://mp.weixin.qq.com/s/Bv8v7kbjX5NWWzdwCnwFHg)  
着重感谢 wlb 和小林哥的帮助  
本文主要对以上链接的分析补充

一、流程分析
======

目标网站：

```
aHR0cHM6Ly9hZG1pbmRhc2hib2FyZC5tZXRhbGlua2VyZC5jb20vYXBpL3YxL21lbWJlcnM=

```

画个图，更清晰  
![](https://img-blog.csdnimg.cn/47ee0917fd5f4e18b261347eaba9f0ca.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LmF6KeBQnVn5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)

第一个请求
-----

流程上面都分析过了，这里记录一下自己实际逆向的流程，  
首先不带 cookie 进去就是一个 503 的请求，该请求返回俩自执行函数，第一个是_cf_chl_opt，这个很重要  
![](https://img-blog.csdnimg.cn/22f6d91b039c4e2baf2ff5a3baf3dcdf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LmF6KeBQnVn5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)  
第二个自执行函数是一些 dom 元素的操作  
![](https://img-blog.csdnimg.cn/6821a4bca22d4fc8b277d6465fdbb368.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LmF6KeBQnVn5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)  
然后拿着_cf_chl_opt 中的 cRay，拼接 url 请求下一个接口

第二个请求
-----

这个接口返回之后唯一一段加密 js，长这样

```
window._cf_chl_opt.uaSR = false;
~function (x, w, t, h, g, f, e, d, c, b) {
    if (b = 'replace,RVEge,prog,number,xdmKp,ODazn,FMBYS,wouLN,XSbWZ,cRrwX,dxLpd,OJpFq,hasOwnProperty,ODUko,cursor,sCURp, - ,_cf_chl_enter,AMyIl,dEgTT,zCjOc,hxiTC,szjNT,SkzCz,xfJCK,getTime,jzaDV,inncP,sGRQJ,Izubc,IvxwG,DYiSO,SmVhY,lastIndex,cf_chl_,Content-type,QRFPm,POST,cVrSh,chCAS,Please click here to continue: ,NKhUt,YBYLu,ckNxt,RdClu,JjSGs,SQQmZ,wlcVX,nGobG,XKszC,tBibs,clUuL,getUTCFullYear,hcXel,JODgR,hsGyF,no-cookie-warning,dZiTr,fHukQ,XDMsi,cFPWv,foecj,CwcMO,aGmHC,cAALT,wlxWa,ontimeout,SYPbO,cBcmF,removeEventListener,VQlue,Iohki,VjGtr,JtQkC,oaxKm,mousemove,HPNlx,TbEHX,wFqLI,IDCLk,XrpAj,LzDQH,SEfpg,slice,cPLYw,Column: ,RKxed,YRsvx,AYFKY,HtPMT,push,black,Function,WjOaF,valueOf,JSON.stringify,5|0|2|3|1|4,0|2|3|4|1,wLcJO,eOVcv,DroXX,cmXTZ,MrWAJ,gmylq,NWEDf,jhomh,ZyfbL,open,UxQhn,cf-content,treqP,3|16|8|10|15|12|14|5|4|1|13|6|0|7|9|11|2,FesCd,YwHnA,UBgUt,alert,chC,type,cRay,QhvJc,call,FLkcD,MnTGv,cpVPs,JnqJn,onclick,hCKOM,LKZvr,HBBzy,DzCvs,VfOnS,mSYUU,FdspD,cf-please-wait,EmAnM,BMgul,object,lnTAv,%2b,HcnFp,bXYiy,3|0|2|4|1,split,This browser is not supported.,parse,timeout,boolean,_cf_chl_done_ran,AMdLM,kfWMn,zyNgz,flow/ov,Clvpm,xxjLN,application/x-www-form-urlencoded,zcWFT,submit,min,jfgkd,GVhEG,wEQWu,KTnkX,bSYhh,function,navigator,zSjAE,ntvrH,ZOXnP,dOCUQ,nhono,<div class="cf-content"><p style="background-color: #de5052; border-color: #521010; color: #fff;" class="cf-alert cf-alert-error">This web property is not accessible via this address.</p></div>,pVkJW,click,apBoL,lHSRF,vZawK,OHaWl,gyrXO,cTTimeMs,WBPBi,data-translate,msg,kqtWF,expires=,voLrG,SHA256,TMaui,beacon/ov,floor,setTimeout,GYRbH,pCKLw,jHYiX,XMLHttpRequest,SGhjH,qLJiu,mLQEw,iJsSw,UGdih,_cf_chl_ctx,complete,jc-spinner-allow-5-secs,iiVWM,setTime,lOsWf,cf_chl_prog,PxgtF,iGKQX,_cf_atob,cached-challenge-warning,jlFOY,oofBv,location,nHmgR,UlOTh,hZjBT,I am human!,getUTCSeconds,CTTlh,prototype,CxUhr,WYNYh,cHash,iViXT,location-mismatch-warning,oMOrQ,null,protocol,vSnAo,ybTVb,EtwWu,DOMContentLoaded,pCstw,TCZIG,HsNTY,cozTE,1|4|3|2|0,rebzA,WHbxJ,NaYvV,interactive,Ayryc,tPnLe,0|2|4|1|3,apply,document,eqdxb,onerror,zuJgi,YckJl,VoiNr,zeALr,prQpu,charAt,Qxjpy,getUTCMonth,display,CLoiJ,pjhll,mdyIS,ZigwX,cNounce,NsXAM,RSpPQ,dJlvG,suEHk,reload,pointerover,ZMdPq,ISxfx,PXYmO,JJQZV,attachEvent,AdtWT,span,TNdve,JqQaJ,cf_chl_rc_ni,=; Max-Age=-99999999;,responseText,ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=,gZKqW,cRq,getElementById,ZssvZ,OjdLJ,elmvc,kVDGN,UpmHK,tUUWC,JZOmt,gdmwH,passive,toJSON,Line: ,vAZoJ,ZeKST,MaXrp,yuNaf,qyXqV,hostname,_cf_chl_done,JULnK,CneYc,<div class="jc-content"><p style="background-color: #de5052; border-color: #521010; color: #fff;" class="jc-alert jc-alert-error">该质询页面被意外缓存，不再可用。</p></div>,hgWOV,[object Array],QuLPh,LWlpb,send,IZiRD,ocAYq,gmCAq,innerText,FqJsf,indexOf,innerHTML,PtuQb,rKipU,PEvUU,mkCHf,wygPQ,RmIYm,jsgSm,<div class="cf-content"><p style="background-color: #de5052; border-color: #521010; color: #fff;" class="cf-alert cf-alert-error">This challenge page was accidentally cached and is no longer available.</p></div>,/0.7832973745134263:1645147211:927be1b6cbf90c0ab7c8b1c1929ab5998691f3f79f9f0a3f7374a129ab774318/,_cf_chl_opt,appendChild,substring,LbhLB,toLowerCase,mScZc,script error,URL: ,color,join,OgxoB,AVibj,xOwqR,TpnJZ,mXIvU,wyDAY,input,PJRPj,NxrWj,TaPba,htzOl,QJDcz,[[[ERROR]]]:,ipAnh,BldNm,zgbBq,THMFK,Message: ,jc-content,cookie,%E8%AF%B7%E7%82%B9%E5%87%BB%E8%BF%99%E9%87%8C%E7%BB%A7%E7%BB%AD,imKvr,GVqmi,tfGTC,console,qjNHP,TSRkO,toUTCString,RRhXk,ActiveXObject,0|1|6|4|2|5|3,readyState,zyLqB,test,atob,error code: 1020,vfwTU,VmTdS,pow,loaded,DgGrK,NZAyS,faKWc,xIkFB,zssAO,onreadystatechange,gqTaD,CF-Challenge,chmWb,RHciH,readystatechange,RlkdV,dZbMF,qOzgI,vsVqY,ruebp,stringify,TPAoN,DHOhZ,value,nXUWe,challenge-form,bUXtG,addEventListener,GkQTV,LQckD,cType,cookieEnabled,length,keydown,QHYRd,Math,createElement,getUTCDate,kCdUu,EVFBa,/cdn-cgi/challenge-platform/,fromCharCode,uPqsz,QaYyx,parseInt,JSON.parse,cvId,vsGrz,kZLJP,PbFyE,rHHfu,UvQbr,eiuVN,GRMwY,srTfC,Error object: ,setAttribute,style,wssSF,aEWuc,wMMoP,CygPg,jc-please-wait,cVzlx,MlMEk,YWtPq,toString,href,touchstart,RpkrP,PwCHh,MzDfG,JwsPo,UOHCE,sendRequest,getUTCMinutes,charCodeAt,;path=/,nCvyh,VQgmD,dWlzJ,status,JdxvN,GbSbV,JSON,getUTCHours,HsyTW,iCqlf,PknVD,zITJd,wHWhJ,VJilg,Date,xPXle,VURlA,none,cagdY,ekytz,block,XpehQ,cf-spinner-allow-5-secs,kJATM,aPeZJ,uedyM,string,xQyLl,now,dEcDw,0000,wNmqr,oGeOL,WiHPI,cLt,BvKNv,chReq,pointermove,setRequestHeader,isrcr,UDhPD,qhIRN,qBohS,<div class="jc-content"><p style="background-color: #de5052; border-color: #521010; color: #fff;" class="jc-alert jc-alert-error">该网站资源无法通过此地址访问。</p></div>,qlAps,jojto,CpKpt,$jvQ3hNwXo4aP5M6KtTZiOxyqgp9E7I0frBbkUCRneWGL1c8uHVFAmJ-z2DSs+lYd,0123456789abcdef,kwGev,Microsoft.XMLHTTP,IhCFr,alEmG'.split(','), function (a, c, d) {
        d = function (e) {
            for (; --e; a.push(a.shift())) ;
        }, d(++c)
    }(b, 177), c = function (a, d, e) {
        return a = a - 0, e = b[a], e
    }, d = this || self, e = d[c('0x44')], f = [], g = function (y, F, E, D, D, C, B, A, z) {....

```

大概 1000 行，ast 之后大概 700 行，执行这个 js 之后会生成一个参数，  
长这样

```
v_6df5acf6ce099446=HbfaSaBarazaqT8lc8LauSJxa6Qicb3PF8$yZNaob3a8Gb87yaCbzqlfc4808c38t8MXpaC8rJ8yrY8c%2b$w8t6fvLPJYY8ewpzi8OwaY1icMCY6tzS8P6JvQr3Bw8-8Pw8haYb8gacHz8oe+U8ZYK8cDSuJbWg38F8BA$YTw8BJPVLGLTgf858o78ml9QaxL5gJP28ow8tlY8PnE8xeaC7JPyusjwU3PB7UNwVr+08PB7EzVK6hz5Z1AMrsjzuhw8OsCua8G8fJPsQ6VHacV9acXhQwnuThzwv2-rvnXMuJ$hLx1ixKXLERcKJ$ZLm-r3huTY-eRLQDOJJgev8Mv+e$gOR9zduaq8c1nliI$bRFrIeSGy1qleyVFLxMxHcDB75dzAmTcmi$YO9$4FTS3wweZwh$onxbb$O889b6b85rgtgJr1htcHHPZ9JD8Py-MaFq38h9QR71GRMSGQpE9dOh$nOZwSow1TSshiYY7phYmd3OoEiwOfVYiJa9RfnPbhwMIw1RV7BBxVlg8BB0jX1z4Qo$9Wf9l$h6NltX1z+wzbCBCQX8QT$ICfvHbvZ9qX34WJuHA4dGMRb85fI8MafAaYOu9bJod6ZztrALsCHg6ZBPJCgN7JEmOY-fJuFz5PaC5rlj6BPrv2eHdGHiq5qwv5TlQSZp-td19A3-+EB8c5czmO1Bdg9JSOJPcQCg5Bx-+Cq8uJEBFlfuPH8P17M0YEgSTcF8JFBrJ$hZYriRBMu3Yn77iGPL9AJydd$ImB-6C$TjEuwEl6JaiM3PZClQuPaCP9yrLShBn6aRYjZMEh7r3lZ9uJyonZM1xfOzhor-3bY75$lZi7JFo1ZzgX9Fi4grHzNTcjdr8im3a$IOzVPIgMFSIf61qJ5S9y3CPbsavMwyPfAZdELzF2Bg398iYQb8it94gnrivoxYX5Zah$sPoWfQXxY+AcSZaRZi7ixFn9Mg$JZi3Y05$fsgu3FdrA6y8audcY1-8GZeuJMw7Z$MZeQaQYnOiM$bCSbecud5cS5PTdBZaEPo96UY1L3ud6-3uonuXAY48MEXYJ$NXrtgH$TcPAoeC9yywZzSo3udJTXH8KGyBPFsqZMmw0-3QwEM3GdacJtw1u$gongQCsaAfCo1W6ho7ZMoTbLSeq3JaMaigJzgr8M5BYd$BX7PLg8a6aGozMzh6aOi4m1fSUPIBX$d3CJPhe5$-hqdiMq85$0PdZ$IBcW6LXogMxM8dXwPa58bowo8zrXJ$VorZMQq1OMjoEm4FZCLgMoeMzldOFzVB7f65caLS4wEmNb6PBe9Z49l4cY+PlZCgMo$Irgbt7ZMVdaHzsZgL94xr7An5zR$4Bw+PcZg5afZ7-S9ynfAscaj$1ywW64dBx0Io7YlMQdCMvPIucfYagMa4EuH7$MEaSJilfQoOZzhPT-SE$a8MbFeg+zo3rSLTcc$4ZY9fuBBzafcwt$MdovHh8al6zZ4+j4cXPQLZgQZSdUJaicatgJyeJFydEc8to-6oyPBbJfPhAP4ZqrzbF3uMHhiZJhMaOMPB7Oi58n$QhYjFH8XJLdBPv5Q9bJMh8wXxYY6a0ayd1qzT888UzZLaqatB88

```

第三个请求
-----

同时会生成 url，和 cookie，带上上面的参数请求这个 url 会得到一个大概 10w 的数据  
长这样  
![](https://img-blog.csdnimg.cn/e9f14b54ff014e37ace870c49653cb80.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LmF6KeBQnVn5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)  
然后呢，新玩法来了，我之前也是没遇见过，同一份 js，生成不同的次数，根据每次请求得到的数据，生成不同的新的 js，这次返回的数据生成的 js（`第一次`）长这样  
![](https://img-blog.csdnimg.cn/92dfe2c13e7f48c29f16a67620d0c9e0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LmF6KeBQnVn5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)  
这里是最大的困难点，一大波的环境检测，非常恶心，并且这一份 js 也是动态的检测环境，补吧，那就

第四个请求
-----

还是一样，得到一个 post 参数，和第三次请求的参数名一致，但是长度长很多了，带上这个参数请求之后得到一段加密数据，大概长度 2500-5000，不在区间的，基本是环境检测没通过，返回参数和第三次请求长一个样，加入 js 之后，生成的 js 长这样  
![](https://img-blog.csdnimg.cn/8a15c72caea345a59617ff6a5a756789.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LmF6KeBQnVn5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)  
其中有最开始第一次请求中的 form 表单中的 input 标签赋值操作（这里的 ast 没写哈哈）

```
window._cf_chl_done = function () {
        window[_[5]][window[_[5]][_[3 * 2]]].done_entered = _[_[_[0]], 7];
        setCookie(_[0x8], _[3 * 3] + window._cf_chl_ctx.chC, 1);
        var vcEl = document.getElementById(_[10]);
        var answerEl = document.getElementById(_[11]);
        var formEl = document.getElementById(_[3 * 4]);
        var inputEl = document.createElement(_[13]);
        inputEl.setAttribute(_[14], _[3 * 5]);
        inputEl.setAttribute(_[0x10], _[17]);
        inputEl.setAttribute(_[3 * 6], _[19]);
        document.getElementById(_[3 * 4]).appendChild(inputEl);
        var redirecting = document.getElementById(_[20]);
        if (!redirecting) {
            redirecting = document.getElementById(_[3 * 7]);
        }
        if (redirecting) {
            redirecting.style.display = _[22];
        }
        var pleaseWait = document.getElementById(_[23]);
        if (!pleaseWait) {
            pleaseWait = document.getElementById(_[0x18]);
        }
        if (pleaseWait) {
            pleaseWait.style.display = _[25];
        }
        var allowSecs = document.getElementById(_[26]);
        if (!allowSecs) {
            allowSecs = document.getElementById(_[3 * 9]);
        }
        if (allowSecs) {
            allowSecs.style.display = _[25];
        }
        vcEl.setAttribute(_[3 * 6], _[28]);
        answerEl.setAttribute(_[3 * 6], _[29]);
        eraseCookie(_[0x8]);
        eraseCookie(_[3 * 10]);
        if (window._cf_chl_opt.cOgUQuery === undefined) {
            window._cf_chl_opt.cOgUQuery = location.search;
        }
        if (window._cf_chl_opt.cOgUHash === undefined) {
            window._cf_chl_opt.cOgUHash = location.hash;
        }
        if (window.history && window.history.replaceState) {
            if (window._cf_chl_opt.uaSR) {
                formEl.action = location.pathname + window._cf_chl_opt.cOgUQuery;
                history.replaceState(null, null, window._cf_chl_opt.cUPMDTk + window._cf_chl_opt.cOgUHash);
            } else {
            }
        } else {
        }
        formEl.action += window._cf_chl_opt.cOgUHash;
        formEl.submit();
        window._cf_chl_done_ran = true;
    };

```

第五次请求
-----

最后拿到这些数据和最开始第一次请求返回的参数得到 post 请求的 data，长这样

```
{
	"md":"eviIy4_OxvBKgYKB3LRXKUNZV_lcE4LBSSzNOb77Afg-1645072000-0-AVnCxJI1vecTb2IrqnWlE_hTHuun9VVyG2NOwMUFDhfD1i5UPSQcnQ73h7vpKQO5DIiB3nTFvJ42PBg3EUNR_onDwKqJLFQbsYuiaLst0KsCJ6ih2h_BiZgmyEVERhRgGsyiiKRTiyeJ3Km6DicHD-mMg-QND7farSdH1jkPJljcCxwS7eAEC5Rnz9-7fIFS-muioyEAJJEsr4Yc0SUVByLig7suxFfaxyutQK-wGelhwkTBWsqKtte7y_EC3HjWPlBFOmBb7nENlXNhGtQ8k3IrifvgC1G_SuHoAIZJVQFv2CuaBNDFl9bnGw6BKimPrPlVHNRsOhNT8TWeKS5LfkICEoWbXzYsedB1i2KZH1XuBbjI6GvKM7ws0yxySF-6JRGFbIGxyoHY39uBwnFTsKp8juJ6Tc2SoWcW9NECKERX7aNVa21-ADfQUZiB00qTSRa2fRt2O_dvfOmDHgqjW37CKiNzH1zsZ27jMF1vDLmOFf7GfVcemzMXER8SIKSFOnmVDxIiW4rl-vfJ6koMRvAkW2N8gVo5oM9qWT4OD3te",
	"r":"TINRLph6PQUsGYHTF9srMhhD0KZx2VG6jp9ooa5lAVw-1645072000-0-AQE/8aXzKr8y73dzLoU/ZQyhnNn+wmF8cqtUdSqDMrTpO+5ibUDXTKqegYk5ss5+SvQyMKwLCDrUQ/n16IdSi6g0jJMbBH7Q1f9CUofI5//6UOCFuseamCTa8of7E5Xa3f8iT1jEWyDP7GnlSy7nWVfuTN3St9E5Q2eKPtaSGUSTKEN4b7J1hrLeeO6LMHuVFvog+pLz8jbZCryHXp1WHe8E8Tl0Ob5mrumCjfddx5OeCPBrmJeKdb8o/ku4Er7nv4HdmpBZY9zWbtzxyPRvdBuWMwIzlTeSlkgVTCaPgUp8s2I7HXtZ3MK3lDPmksMu+KU2yeb/4c/lQ7DHUXRW5mipEPhqPDjh2LvY3swyJf1QKVLix012kbtXy177HLXDvoPpjJ+67AkzmSiT7D6BJH0NWZm5h5JV4DET02MLWL0nx41LHFf+8icYO0uqLxvQOPzS1qUFNq+LYDwShSBfuOMtObFkMABzj1HA/2+OT99qlGe+Fmq0SGTHQUSGW4ovogOFksWY7+D/oOexjG/MpFmjVk0w26gYyc7IAXw4isP1HL98JRl3YNR9OMYGyJ6BK/ZkyJn1/RyEEnrpbehhH1mRLjE7Cc5y3F2aUnX2Hjs0wIFu93T09DlxSbSBp+PjlN+r88DNUHT3Ll4cLUMiQP6V5nFRrNX8nqGWVb9md7udxo9R+qqmR1VQg8VQAmawCIucpAjbkfyJlTTs4mIbcMQjqdyZ7DzQOe+1xohiol5Su3QQobAdJe0dmjNm0EhJmPT5MnRGBWJruM49TMDxDm/+kqmE9PUgcFzFzyDXnLqyYGakBPiRtej6B9DyFxvXoN9w9pa+yaAwTb8f6IBvH0gkVgt7K1Ay+HQtwoOcyg1VwPA9BpxbQ8dHE2iY+QHzwrP8wBZvSnY/Qtzy9MF5XoYrkaKuFz8XlgCOsjDlWQNTvqaLd2ubPVLfr05ZNe2OkvOuZGRcecnSHvgOJsQw6sjylgnqES4cWivO6GN54YcLvnjtgRikegV1yrf8B38jRq8CW97d81tu1eyfeWHq6MjRs54KXgmSdNjaNQ2uY1+1RnXGsjnq6X1PxUMagHDqe35UsLjT48wHfe3A7M7d0WNFDcGSsuUzFVO3q85NQuOQ4xi+rSMDOUkzw6CQkoJQkJW0cHfASTaiggcDqVcfRkDL4Q07zHVc+32g6NfoQZh3k0+cjKvrS79FYTgEx/8tU8BQySxlduLiKW/QBmdsiwN684VpF3m4cssmXRN+/qm9Q51uOGFmpYae0HSEpgvDHzGQT+3NfcTUZN7NVIciwUqWPleyCBjkiNMp0tCJzNPE06kB0iaYpjeXM+z/mGPyu0CJNFGsujdHqcCCZ0n4FAD1eBEXQzCKflhcwgB8QEAKms6C1Y/Rmb2NdapgRnTcDR1dE+S6o8W7wdvdVjjw3eazUgJai9khbNyrwiQNXZGbC6r/WkyeZmZaWUxUF2eX4ePqh8yO9NyG/9x0Ve/K5nGyya6VKQhlgIeqRL1KjNvvSnn5fE3qfjXbc3o9YogmGTAXQIjjipoZG08aPylo4m2ODXTQm/FEuwnpMLQQc2qU4cRMwCe+VO+5rS0E5mfwmtvb99x3q4R2FzfWa2V0npNJqJziGjdN65ss0C/wUmyQsydgdbOvB6QcNaqMZWOMGAsSdPQ1LFToxvftHVB0/iFVjuTID/GT9LqYVyZe9pk0lAlKwE84zKgHGmSuN/ksl0ptzWbky8QOJBLUfqPl5VFJeNMccsIlTWZN+zIi4Hmxn3LNCDyg9BdA46IUtvtFupy/9lC7iv+VDxRIbb/cXdnz5FXZfMGob4xppDTIbsQvI0yo7QIQj8Vh15Pll1TqFD5AxYdwEHXvb32K4LLkdOhYrv5a/DtFBhPei5JE/93TlK/sgXm20Arn6+GqLk722tvhO/x7T5DhkX6cAcWMrh6cJ+RCeC2H/BINuFsBzqkMjir4/eu/mDMAHQSkquvuVmpX0H0FSNgZbBRyBJRXdKfZsp/oepEj0DXxT6tqsD2RFINaNyNL6ZP7w/5k6S0wQAfYaABTuNMifBDxL4JNEPtCj0bOprVXEGirO7q6dbx2u1zZtKJQ3VUfMRRHmWnZ00S4z+0z9cZGGuW3BTlzysfcfOSi87SjmveotlVpjMXDa0c5bgKNOQvzeiwB1N3zYxKcnfyj+2l4vLrD868Hv2E75eM5Y9GjBYaEsNyO14WNgqAyXsu53IXPlt9dT/HM/kN6hLG6PC1fT0a7VrBovEXE0b3ibhSjl+CpHoLcb453NCbkJzDdSc0ndLFZKnDAnM7soN5wKQsg/cyafV3BPtAV0Yrckc6bdEg7Z5nBglzgL4miLEJZK5t5B4h+K1s3+mUb+oUJ/GngHF8LQBhTnAy6fT0oFcZpaw6qySGl08DKR+eLxbcWmmpAltduZRFgVAt+bLXEw23aIcohDMMMC6lmqnk5rm9Nfnm1pmsq17xvbCb5vBhrAz6SrGKXu6SgHnQVYfhdl8W3yJvTv28NzAWViXrttoeFZaE7jaXURoUsZR4RfXYpYckrBOyEeubvxeTJMY3mcMj8iNhBI83mDkG+hf/Y0xSS9LT3qxB+L2/KGv3O4jDy0uiz4zbR6t/NePKamhApq9N1VBA0zRR1NXDcDdMArRBHyhFwESPq/pK1P5fID/n0Xys2zG9BDuFbjf2RRKXwlH0SALr4zdYyovmU7KeIMU94vyO4pJz934UhW4+mgQiuXQl8iEpb//W24XD9Rljjk97S91I9IMnZt5XjSmdFzCmaMyOHuynIKLR/N0FCqcVLG2Sh86osXPHlgPO4nT+5q2QJ0lMfy12EdJzC0OBaYSaBa1wgxiEsXoSa",
	"jschl_vc":"255ec208ddfbb1da78f57c2db512976e",
	"pass":"1645072001.406-7d4c8sgNOF",
	"jschl_answer":"JWflpIpDaJBN-13-6dec4242683b96c9",
	"cf_ch_verify":"plat"
}

```

md，r，pass 都是第一次请求中 form 表单的 input 标签 name 对应的 value  
剩下的就是第四次请求得到的参数，  
最后，拿到这些参数请求主页，状态码 302，得到 set-cookie：`cf_clearance`  
![](https://img-blog.csdnimg.cn/271449aa98ad4022a659ee8bbf53ac31.png)

二、注意事项
======

这里就是本文的精髓所在了，有些坑给你们说了，

1.  `代码不过夜`：第二次请求中的 js，会有一个时间检测，也就是`window._cf_chl_opt.cRq.t`为 base64 的时间戳，该时间与当前时间差值大于 12 就走错误分支了，长这样![](https://img-blog.csdnimg.cn/545bf3a3d53a4d90b219b4ba9750efac.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LmF6KeBQnVn5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)
2.  在第四次请求的参数生成时，还请求了一张图片，参数里还需要该图片的长和宽，需要注意一下，
3.  剩下的就是自己踩吧，全是环境检测，像上面参考链接中的 cookie 这种，也是个大坑

就这样吧，不想写了

总结
==

这东西整个流程还挺有意思的，算是我第一个搞的比较有难度的产品吧，最后感谢 wlb，感谢小林哥的帮助，当然主要还是我厉害 (●ˇ∀ˇ●)