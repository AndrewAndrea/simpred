> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [articles.zsxq.com](https://articles.zsxq.com/id_skxjqd8a71ep.html)

> 之前的 wasm 转 c 调用文章中，其中的工作核心其实就是需要补 js 的导入函数，也可以说叫做补环境。

之前的 wasm 转 c 调用文章中，其中的工作核心其实就是需要补 js 的导入函数，也可以说叫做补环境。导入的类型分为四种导入表，导入内存，导入函数和导入全局变量，各种导入类型之前都已经介绍完了。除了导入函数这种不难以被忽略的内容外，还有一个非常容易被忽略的是预设动态内存基址，最后两篇讲的就是需要预设动态内存基址的案例

相关文章

[1. 某网站字幕加密的 wasm 分析](https://www.52pojie.cn/thread-1461335-1-1.html)

[2. 调用 monalisa.wasm 获取字幕解密 key](https://github.com/592767809/xhlove-share/blob/master/iqiyi/%E8%B0%83%E7%94%A8monalisa.wasm%E8%8E%B7%E5%8F%96%E5%AD%97%E5%B9%95%E8%A7%A3%E5%AF%86key.md)

新知识点

1. 预设动态内存基址

在这个案例的 js 中，有一段这样的代码

```
avascript
var DYNAMIC_BASE = 6065008, DYNAMICTOP_PTR = 821968;
HEAP32[DYNAMICTOP_PTR >> 2] = DYNAMIC_BASE;


```

这里就是在 wasm 中预设动态内存基址，然后一般情况下会把这个基址作为导入全局变量，但是在第一个案例中没有。

在 CLion 中，这个案例的导入操作和之前是一模一样

```
static wasm_rt_memory_t w2c_memory;
static wasm_rt_table_t w2c___indirect_function_table;


u32 aZ_nZ_iii(u32 t1, u32 t2) {
    return 0;
};

u32 aZ_oZ_iii(u32 t1, u32 t2) {
    return 0;
};

u32 aZ_lZ_iiii(u32 dest, u32 src, u32 num){
    memcpy(w2c_memory.data + dest, w2c_memory.data + src, num);
    return dest;
};


u32 (*Z_aZ_aZ_iiii)(u32, u32, u32) = NULL;
u32 (*Z_aZ_bZ_iiiii)(u32, u32, u32, u32) = NULL;
u32 (*Z_aZ_cZ_ii)(u32) = NULL;
u32 (*Z_aZ_dZ_iiii)(u32, u32, u32) = NULL;
u32 (*Z_aZ_eZ_iiii)(u32, u32, u32) = NULL;
u32 (*Z_aZ_fZ_ii)(u32) = NULL;
u32 (*Z_aZ_gZ_ii)(u32) = NULL;
u32 (*Z_aZ_hZ_iv)(void) = NULL;
u32 (*Z_aZ_iZ_ii)(u32) = NULL;
void (*Z_aZ_jZ_vi)(u32) = NULL;
u32 (*Z_aZ_kZ_iiiiii)(u32, u32, u32, u32, u32) = NULL;
u32 (*Z_aZ_lZ_iiii)(u32, u32, u32) = aZ_lZ_iiii;
u32 (*Z_aZ_mZ_ii)(u32) = NULL;
u32 (*Z_aZ_nZ_iii)(u32, u32) = aZ_nZ_iii;
u32 (*Z_aZ_oZ_iii)(u32, u32) = aZ_oZ_iii;
u32 (*Z_aZ_pZ_iiiii)(u32, u32, u32, u32) = NULL;
u32 (*Z_aZ_qZ_ii)(u32) = NULL;
u32 (*Z_aZ_rZ_ii)(u32) = NULL;
wasm_rt_memory_t (*Z_aZ_memory) = &w2c_memory;
wasm_rt_table_t (*Z_aZ_table) = &w2c___indirect_function_table;


```

其中预设动态内存基址我是放在内存初始化的函数

![](https://article-images.zsxq.com/FrmHIqaJkCOHSZ0SGXRloOtBOQ7U)

这里的 821968u 就是 js 中 DYNAMIC_BASE 变量的值，4 代表是长度，那么上面的【0x70, 0x8b, 0x5c, 0x00】是怎么来的呢？实际是 DYNAMIC_BASE 的值 6065008 的小端序，如果不知道怎么计算，可以在 js 中打印出来

```
avascript
console.log(HEAPU8.slice(DYNAMICTOP_PTR, DYNAMICTOP_PTR + 4));


```

就可以得到结果

```
avascript
Uint8Array(4) [ 112, 139, 92, 0 ]


```

这里把 10 进制数直接写进去也可以，我按照统一的表示，就转成 16 进制

然后就可以开始编写 c 的导出函数了

```
#include <stdio.h>
#include <stdlib.h>
#include "iqcom.c"

extern void init_wasm(void);
extern char* get_key(char*);

void init_wasm(){
    init_func_types();
    init_globals();
    init_memory();
    init_table();
    init_exports();
}

char* get_key(char* license){
    w2c_s();
    u32 ctx = w2c_D();

    int license_len = (int)strlen(license);
    u32 license_ptr = w2c_J( license_len + 1);
    memcpy(w2c_memory.data + license_ptr, license, license_len);

    u32 z0_ptr = w2c_J(2);
    memcpy(w2c_memory.data + z0_ptr, "0", 1);

    w2c_F(ctx, license_ptr, license_len, z0_ptr);

    char* out_str = (char *)malloc(17);
    out_str[16] = 0;

    memcpy(out_str, w2c_memory.data + ctx + 4, 16);

    w2c_x(license_ptr);
    w2c_x(z0_ptr);

    return out_str;
}



```

编译为 dll

```
ash
"D:/MinGW64/bin/gcc" -shared -Os -s -o iqcom.dll main.c wasm-rt-impl.c


```

在 python 中调用

```
ython
    dll = ctypes.windll.LoadLibrary('iqcom.dll')
    print(dll)
    dll.init_wasm()
    dll.get_key.argtypes = [ctypes.c_char_p]
    dll.get_key.restype = ctypes.c_char_p
    t = dll.get_key(ctypes.c_char_p('AA4ACgMAAAAAAAAAAAQCDwACATADEAAnAgAgeyWysVa0GpbmCNvd+S1tsL6yp/j2tbA14sqW1ppgepYCAAAAAxEANwEAMDCtrqLHyZQ7p8RX3ih4NIqLWR1zCfu3mMFlxC2kiPgHmxZY7I/KYq4pMkH3rZQsqgEAAgD/EgAkAQAAIGSSUL7C0qWJp/LIkKoS12QYws1e0z/CewNJaaqktC3z'.encode()))
    print(t)
    print(len(t))
    print(t.hex())


```

这样就成功得到字幕解密的 key

![](https://article-images.zsxq.com/FvLe5zocbbeAdCXcqKdFdbJxsl6Q)

接着来下一篇，这是一个 ts 文件，解密算法在 wasm 中

新知识点

1. 字节集转 16 进制字符串

这个案例的 js 中，也有一段类似的代码

```
avascript
var DYNAMIC_BASE = 5350544, DYNAMICTOP_PTR = 107392;
HEAP32[DYNAMICTOP_PTR >> 2] = DYNAMIC_BASE;


```

也是有预设动态内存基址的，并且这个 DYNAMICTOP_PTR 也被作为全局变量进行导入。一般来说，这个预设动态内存基址都是一个定值，所以即使是一个变量，也可以写成一个定值的。

这个案例的导入函数比较多，有 170 个，不过绝大部分都是可以设置为 NULL 的。导入部分都是一样的写法

```
u32 envZ_invoke_iiZ_iii(u32 e, u32 t){
	return w2c_dynCall_ii(e, t);
}

u32 envZ_invoke_iiiZ_iiii(u32 e, u32 t, u32 i){
	return w2c_dynCall_iii(e, t, i);
}

u32 envZ_invoke_iiiiZ_iiiii(u32 e, u32 t, u32 i, u32 r) {
	return w2c_dynCall_iiii(e, t, i, r);
}

u32 envZ_invoke_iiiiiZ_iiiiii(u32 e, u32 t, u32 i, u32 r, u32 n){
	return w2c_dynCall_iiiii(e, t, i, r, n);
}

u32 envZ_invoke_iiiiiiZ_iiiiiii(u32 e, u32 t, u32 i, u32 r, u32 n, u32 a){
	return w2c_dynCall_iiiiii(e, t, i, r, n, a);
}

u32 envZ_invoke_iiiiiiiZ_iiiiiiii(u32 e, u32 t, u32 i, u32 r, u32 n, u32 a, u32 o){
	return w2c_dynCall_iiiiiii(e, t, i, r, n, a, o);
}

void envZ_invoke_viZ_vii(u32 e, u32 t){
	w2c_dynCall_vi(e, t);
}

void envZ_invoke_viiZ_viii(u32 e, u32 t, u32 i){
	w2c_dynCall_vii(e, t, i);
}

void envZ_invoke_viiiZ_viiii(u32 e, u32 t, u32 i, u32 r){
	w2c_dynCall_viii(e, t, i, r);
}

void envZ_invoke_viiiiZ_viiiii(u32 e, u32 t, u32 i, u32 r, u32 n){
	w2c_dynCall_viiii(e, t, i, r, n);
}

void envZ_invoke_viiiiiZ_viiiiii(u32 e, u32 t, u32 i, u32 r, u32 n, u32 a){
	w2c_dynCall_viiiii(e, t, i, r, n, a);
}

void envZ_invoke_viiiiiiZ_viiiiiii(u32 e, u32 t, u32 i, u32 r, u32 n, u32 a, u32 o){
	w2c_dynCall_viiiiii(e, t, i, r, n, a, o);
}

void envZ_invoke_viiiiiiiZ_viiiiiiii(u32 e, u32 t, u32 i, u32 r, u32 n, u32 a, u32 o, u32 u){
	w2c_dynCall_viiiiiii(e, t, i, r, n, a, o, u);
}

void envZ_invoke_viiiiiiiiZ_viiiiiiiii(u32 e, u32 t, u32 i, u32 r, u32 n, u32 a, u32 o, u32 u, u32 s){
	w2c_dynCall_viiiiiiii(e, t, i, r, n, a, o, u, s);
}

void envZ____buildEnvironmentZ_vi(u32 e){
	
}

u32 envZ____cxa_uncaught_exceptionZ_iv(void){
	return 0;
}

u32 envZ__emscripten_get_heap_sizeZ_iv(void){
	return 134217728;
}

u32 envZ__emscripten_memcpy_bigZ_iiii(u32 dest, u32 src, u32 num){
    memcpy(w2c_memory.data + dest, w2c_memory.data + src, num);
	return dest;
}

u32 envZ__gettimeofdayZ_iii(u32 e, u32 t){
	return 0;
}

u32 envZ__timeZ_ii(u32 e){
	return 0;
}

void envZ_setTempRet0Z_vi(u32 e){
	
}

u32 envZ_getTempRet0Z_iv(void){
	return 0;
}

u32 envZ_invoke_iiiijiiZ_iiiiiiiii(u32 e, u32 t, u32 i, u32 r, u32 n, u32 a, u32 o, u32 u){
	return w2c_dynCall_iiiijii(e, t, i, r, n, a, o);
}

u32 envZ_invoke_jiZ_iii(u32 e, u32 t){
	return w2c_dynCall_ji(e, t);
}

u32 envZ_invoke_jijZ_iiiii(u32 e, u32 t, u32 i, u32 r){
	return w2c_dynCall_jij(e, t, i);
}

void envZ_invoke_viiijZ_viiiiii(u32 e, u32 t, u32 i, u32 r, u32 n, u32 a){
	w2c_dynCall_viiij(e, t, i, r, n);
}


/* import: 'env' 'getTempRet0' */
u32 (*Z_envZ_getTempRet0Z_iv)(void) = envZ_getTempRet0Z_iv;
/* import: 'env' 'abortStackOverflow' */
void (*Z_envZ_abortStackOverflowZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_diii' */
void (*Z_envZ_nullFunc_diiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_fiii' */
void (*Z_envZ_nullFunc_fiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_i' */
void (*Z_envZ_nullFunc_iZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_ii' */
void (*Z_envZ_nullFunc_iiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iidi' */
void (*Z_envZ_nullFunc_iidiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iii' */
void (*Z_envZ_nullFunc_iiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiii' */
void (*Z_envZ_nullFunc_iiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiii' */
void (*Z_envZ_nullFunc_iiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiid' */
void (*Z_envZ_nullFunc_iiiiidZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiii' */
void (*Z_envZ_nullFunc_iiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiiid' */
void (*Z_envZ_nullFunc_iiiiiidZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiiii' */
void (*Z_envZ_nullFunc_iiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiiiii' */
void (*Z_envZ_nullFunc_iiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiiiiii' */
void (*Z_envZ_nullFunc_iiiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiiiiiiii' */
void (*Z_envZ_nullFunc_iiiiiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiiiiiiiii' */
void (*Z_envZ_nullFunc_iiiiiiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiiiiiiiiii' */
void (*Z_envZ_nullFunc_iiiiiiiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiiij' */
void (*Z_envZ_nullFunc_iiiiijZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiij' */
void (*Z_envZ_nullFunc_iiiijZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iiiijii' */
void (*Z_envZ_nullFunc_iiiijiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_iij' */
void (*Z_envZ_nullFunc_iijZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_ji' */
void (*Z_envZ_nullFunc_jiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_jii' */
void (*Z_envZ_nullFunc_jiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_jiiii' */
void (*Z_envZ_nullFunc_jiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_jiiijj' */
void (*Z_envZ_nullFunc_jiiijjZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_jij' */
void (*Z_envZ_nullFunc_jijZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_v' */
void (*Z_envZ_nullFunc_vZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_vi' */
void (*Z_envZ_nullFunc_viZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_vif' */
void (*Z_envZ_nullFunc_vifZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_vii' */
void (*Z_envZ_nullFunc_viiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viii' */
void (*Z_envZ_nullFunc_viiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiifiii' */
void (*Z_envZ_nullFunc_viiifiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiii' */
void (*Z_envZ_nullFunc_viiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiiii' */
void (*Z_envZ_nullFunc_viiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiiiii' */
void (*Z_envZ_nullFunc_viiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiiiiii' */
void (*Z_envZ_nullFunc_viiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiiiiiii' */
void (*Z_envZ_nullFunc_viiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiiiiiiii' */
void (*Z_envZ_nullFunc_viiiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiiiiiiiii' */
void (*Z_envZ_nullFunc_viiiiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiiiiiiiiiiiiii' */
void (*Z_envZ_nullFunc_viiiiiiiiiiiiiiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viiij' */
void (*Z_envZ_nullFunc_viiijZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viij' */
void (*Z_envZ_nullFunc_viijZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viijii' */
void (*Z_envZ_nullFunc_viijiiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_viijji' */
void (*Z_envZ_nullFunc_viijjiZ_vi)(u32) = NULL;
/* import: 'env' 'nullFunc_vijji' */
void (*Z_envZ_nullFunc_vijjiZ_vi)(u32) = NULL;
/* import: 'env' 'invoke_diii' */
f64 (*Z_envZ_invoke_diiiZ_diiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_fiii' */
f64 (*Z_envZ_invoke_fiiiZ_diiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_i' */
u32 (*Z_envZ_invoke_iZ_ii)(u32) = NULL;
/* import: 'env' 'invoke_ii' */
u32 (*Z_envZ_invoke_iiZ_iii)(u32, u32) = envZ_invoke_iiZ_iii;
/* import: 'env' 'invoke_iidi' */
u32 (*Z_envZ_invoke_iidiZ_iiidi)(u32, u32, f64, u32) = NULL;
/* import: 'env' 'invoke_iii' */
u32 (*Z_envZ_invoke_iiiZ_iiii)(u32, u32, u32) = envZ_invoke_iiiZ_iiii;
/* import: 'env' 'invoke_iiii' */
u32 (*Z_envZ_invoke_iiiiZ_iiiii)(u32, u32, u32, u32) = envZ_invoke_iiiiZ_iiiii;
/* import: 'env' 'invoke_iiiii' */
u32 (*Z_envZ_invoke_iiiiiZ_iiiiii)(u32, u32, u32, u32, u32) = envZ_invoke_iiiiiZ_iiiiii;
/* import: 'env' 'invoke_iiiiii' */
u32 (*Z_envZ_invoke_iiiiiiZ_iiiiiii)(u32, u32, u32, u32, u32, u32) = envZ_invoke_iiiiiiZ_iiiiiii;
/* import: 'env' 'invoke_iiiiiii' */
u32 (*Z_envZ_invoke_iiiiiiiZ_iiiiiiii)(u32, u32, u32, u32, u32, u32, u32) = envZ_invoke_iiiiiiiZ_iiiiiiii;
/* import: 'env' 'invoke_iiiiiiii' */
u32 (*Z_envZ_invoke_iiiiiiiiZ_iiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_iiiiiiiii' */
u32 (*Z_envZ_invoke_iiiiiiiiiZ_iiiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_iiiiiiiiiii' */
u32 (*Z_envZ_invoke_iiiiiiiiiiiZ_iiiiiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_iiiiiiiiiiii' */
u32 (*Z_envZ_invoke_iiiiiiiiiiiiZ_iiiiiiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_iiiiiiiiiiiii' */
u32 (*Z_envZ_invoke_iiiiiiiiiiiiiZ_iiiiiiiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_v' */
void (*Z_envZ_invoke_vZ_vi)(u32) = NULL;
/* import: 'env' 'invoke_vi' */
void (*Z_envZ_invoke_viZ_vii)(u32, u32) = envZ_invoke_viZ_vii;
/* import: 'env' 'invoke_vif' */
void (*Z_envZ_invoke_vifZ_viid)(u32, u32, f64) = NULL;
/* import: 'env' 'invoke_vii' */
void (*Z_envZ_invoke_viiZ_viii)(u32, u32, u32) = envZ_invoke_viiZ_viii;
/* import: 'env' 'invoke_viii' */
void (*Z_envZ_invoke_viiiZ_viiii)(u32, u32, u32, u32) = envZ_invoke_viiiZ_viiii;
/* import: 'env' 'invoke_viiifiii' */
void (*Z_envZ_invoke_viiifiiiZ_viiiidiii)(u32, u32, u32, u32, f64, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_viiii' */
void (*Z_envZ_invoke_viiiiZ_viiiii)(u32, u32, u32, u32, u32) = envZ_invoke_viiiiZ_viiiii;
/* import: 'env' 'invoke_viiiii' */
void (*Z_envZ_invoke_viiiiiZ_viiiiii)(u32, u32, u32, u32, u32, u32) = envZ_invoke_viiiiiZ_viiiiii;
/* import: 'env' 'invoke_viiiiii' */
void (*Z_envZ_invoke_viiiiiiZ_viiiiiii)(u32, u32, u32, u32, u32, u32, u32) = envZ_invoke_viiiiiiZ_viiiiiii;
/* import: 'env' 'invoke_viiiiiii' */
void (*Z_envZ_invoke_viiiiiiiZ_viiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32) = envZ_invoke_viiiiiiiZ_viiiiiiii;
/* import: 'env' 'invoke_viiiiiiii' */
void (*Z_envZ_invoke_viiiiiiiiZ_viiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32, u32) = envZ_invoke_viiiiiiiiZ_viiiiiiiii;
/* import: 'env' 'invoke_viiiiiiiii' */
void (*Z_envZ_invoke_viiiiiiiiiZ_viiiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_viiiiiiiiii' */
void (*Z_envZ_invoke_viiiiiiiiiiZ_viiiiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_viiiiiiiiiiiiiii' */
void (*Z_envZ_invoke_viiiiiiiiiiiiiiiZ_viiiiiiiiiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' '_ConfigStorage_get' */
void (*Z_envZ__ConfigStorage_getZ_vi)(u32) = NULL;
/* import: 'env' '_ConfigStorage_put' */
void (*Z_envZ__ConfigStorage_putZ_vii)(u32, u32) = NULL;
/* import: 'env' '_DataStorage_save' */
void (*Z_envZ__DataStorage_saveZ_vi)(u32) = NULL;
/* import: 'env' '_Notifier_tri' */
void (*Z_envZ__Notifier_triZ_viii)(u32, u32, u32) = NULL;
/* import: 'env' '_NuRequest_cancel' */
void (*Z_envZ__NuRequest_cancelZ_vi)(u32) = NULL;
/* import: 'env' '_NuRequest_create' */
u32 (*Z_envZ__NuRequest_createZ_iiii)(u32, u32, u32) = NULL;
/* import: 'env' '_NuRequest_start' */
void (*Z_envZ__NuRequest_startZ_vi)(u32) = NULL;
/* import: 'env' '_WebRTC_close' */
void (*Z_envZ__WebRTC_closeZ_vi)(u32) = NULL;
/* import: 'env' '_WebRTC_delete' */
void (*Z_envZ__WebRTC_deleteZ_vi)(u32) = NULL;
/* import: 'env' '_WebRTC_method' */
void (*Z_envZ__WebRTC_methodZ_viii)(u32, u32, u32) = NULL;
/* import: 'env' '_WebRTC_new' */
u32 (*Z_envZ__WebRTC_newZ_iiii)(u32, u32, u32) = NULL;
/* import: 'env' '_WebRTC_send' */
void (*Z_envZ__WebRTC_sendZ_viii)(u32, u32, u32) = NULL;
/* import: 'env' '_WrapReference_new' */
u32 (*Z_envZ__WrapReference_newZ_iii)(u32, u32) = NULL;
/* import: 'env' '_WrapReference_noty' */
void (*Z_envZ__WrapReference_notyZ_viii)(u32, u32, u32) = NULL;
/* import: 'env' '___assert_fail' */
void (*Z_envZ____assert_failZ_viiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' '___buildEnvironment' */
void (*Z_envZ____buildEnvironmentZ_vi)(u32) = envZ____buildEnvironmentZ_vi;
/* import: 'env' '___cxa_allocate_exception' */
u32 (*Z_envZ____cxa_allocate_exceptionZ_ii)(u32) = NULL;
/* import: 'env' '___cxa_begin_catch' */
u32 (*Z_envZ____cxa_begin_catchZ_ii)(u32) = NULL;
/* import: 'env' '___cxa_call_unexpected' */
void (*Z_envZ____cxa_call_unexpectedZ_vi)(u32) = NULL;
/* import: 'env' '___cxa_end_catch' */
void (*Z_envZ____cxa_end_catchZ_vv)(void) = NULL;
/* import: 'env' '___cxa_find_matching_catch_2' */
u32 (*Z_envZ____cxa_find_matching_catch_2Z_iv)(void) = NULL;
/* import: 'env' '___cxa_find_matching_catch_3' */
u32 (*Z_envZ____cxa_find_matching_catch_3Z_ii)(u32) = NULL;
/* import: 'env' '___cxa_find_matching_catch_4' */
u32 (*Z_envZ____cxa_find_matching_catch_4Z_iii)(u32, u32) = NULL;
/* import: 'env' '___cxa_free_exception' */
void (*Z_envZ____cxa_free_exceptionZ_vi)(u32) = NULL;
/* import: 'env' '___cxa_pure_virtual' */
void (*Z_envZ____cxa_pure_virtualZ_vv)(void) = NULL;
/* import: 'env' '___cxa_rethrow' */
void (*Z_envZ____cxa_rethrowZ_vv)(void) = NULL;
/* import: 'env' '___cxa_throw' */
void (*Z_envZ____cxa_throwZ_viii)(u32, u32, u32) = NULL;
/* import: 'env' '___cxa_uncaught_exception' */
u32 (*Z_envZ____cxa_uncaught_exceptionZ_iv)(void) = envZ____cxa_uncaught_exceptionZ_iv;
/* import: 'env' '___lock' */
void (*Z_envZ____lockZ_vi)(u32) = NULL;
/* import: 'env' '___map_file' */
u32 (*Z_envZ____map_fileZ_iii)(u32, u32) = NULL;
/* import: 'env' '___resumeException' */
void (*Z_envZ____resumeExceptionZ_vi)(u32) = NULL;
/* import: 'env' '___setErrNo' */
void (*Z_envZ____setErrNoZ_vi)(u32) = NULL;
/* import: 'env' '___syscall10' */
u32 (*Z_envZ____syscall10Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall140' */
u32 (*Z_envZ____syscall140Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall145' */
u32 (*Z_envZ____syscall145Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall146' */
u32 (*Z_envZ____syscall146Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall195' */
u32 (*Z_envZ____syscall195Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall220' */
u32 (*Z_envZ____syscall220Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall221' */
u32 (*Z_envZ____syscall221Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall33' */
u32 (*Z_envZ____syscall33Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall39' */
u32 (*Z_envZ____syscall39Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall5' */
u32 (*Z_envZ____syscall5Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall54' */
u32 (*Z_envZ____syscall54Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall6' */
u32 (*Z_envZ____syscall6Z_iii)(u32, u32) = NULL;
/* import: 'env' '___syscall91' */
u32 (*Z_envZ____syscall91Z_iii)(u32, u32) = NULL;
/* import: 'env' '___unlock' */
void (*Z_envZ____unlockZ_vi)(u32) = NULL;
/* import: 'env' '_abort' */
void (*Z_envZ__abortZ_vv)(void) = NULL;
/* import: 'env' '_clock_gettime' */
u32 (*Z_envZ__clock_gettimeZ_iii)(u32, u32) = NULL;
/* import: 'env' '_emscripten_get_heap_size' */
u32 (*Z_envZ__emscripten_get_heap_sizeZ_iv)(void) = envZ__emscripten_get_heap_sizeZ_iv;
/* import: 'env' '_emscripten_memcpy_big' */
u32 (*Z_envZ__emscripten_memcpy_bigZ_iiii)(u32, u32, u32) = envZ__emscripten_memcpy_bigZ_iiii;
/* import: 'env' '_emscripten_resize_heap' */
u32 (*Z_envZ__emscripten_resize_heapZ_ii)(u32) = NULL;
/* import: 'env' '_emscripten_websocket_get_ready_state' */
u32 (*Z_envZ__emscripten_websocket_get_ready_stateZ_iii)(u32, u32) = NULL;
/* import: 'env' '_emscripten_websocket_is_supported' */
u32 (*Z_envZ__emscripten_websocket_is_supportedZ_iv)(void) = NULL;
/* import: 'env' '_emscripten_websocket_new' */
u32 (*Z_envZ__emscripten_websocket_newZ_ii)(u32) = NULL;
/* import: 'env' '_emscripten_websocket_send_binary' */
u32 (*Z_envZ__emscripten_websocket_send_binaryZ_iiii)(u32, u32, u32) = NULL;
/* import: 'env' '_emscripten_websocket_send_utf8_text' */
u32 (*Z_envZ__emscripten_websocket_send_utf8_textZ_iii)(u32, u32) = NULL;
/* import: 'env' '_emscripten_websocket_set_onclose_callback_on_thread' */
u32 (*Z_envZ__emscripten_websocket_set_onclose_callback_on_threadZ_iiiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' '_emscripten_websocket_set_onerror_callback_on_thread' */
u32 (*Z_envZ__emscripten_websocket_set_onerror_callback_on_threadZ_iiiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' '_emscripten_websocket_set_onmessage_callback_on_thread' */
u32 (*Z_envZ__emscripten_websocket_set_onmessage_callback_on_threadZ_iiiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' '_emscripten_websocket_set_onopen_callback_on_thread' */
u32 (*Z_envZ__emscripten_websocket_set_onopen_callback_on_threadZ_iiiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' '_getenv' */
u32 (*Z_envZ__getenvZ_ii)(u32) = NULL;
/* import: 'env' '_gettimeofday' */
u32 (*Z_envZ__gettimeofdayZ_iii)(u32, u32) = envZ__gettimeofdayZ_iii;
/* import: 'env' '_llvm_eh_typeid_for' */
u32 (*Z_envZ__llvm_eh_typeid_forZ_ii)(u32) = NULL;
/* import: 'env' '_llvm_stackrestore' */
void (*Z_envZ__llvm_stackrestoreZ_vi)(u32) = NULL;
/* import: 'env' '_llvm_stacksave' */
u32 (*Z_envZ__llvm_stacksaveZ_iv)(void) = NULL;
/* import: 'env' '_llvm_trap' */
void (*Z_envZ__llvm_trapZ_vv)(void) = NULL;
/* import: 'env' '_localtime_r' */
u32 (*Z_envZ__localtime_rZ_iii)(u32, u32) = NULL;
/* import: 'env' '_mktime' */
u32 (*Z_envZ__mktimeZ_ii)(u32) = NULL;
/* import: 'env' '_nup2p_index_callback' */
void (*Z_envZ__nup2p_index_callbackZ_viii)(u32, u32, u32) = NULL;
/* import: 'env' '_nup2p_list_callback' */
void (*Z_envZ__nup2p_list_callbackZ_vii)(u32, u32) = NULL;
/* import: 'env' '_nup2p_read_data_complete' */
void (*Z_envZ__nup2p_read_data_completeZ_viiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' '_pthread_cond_wait' */
u32 (*Z_envZ__pthread_cond_waitZ_iii)(u32, u32) = NULL;
/* import: 'env' '_strftime_l' */
u32 (*Z_envZ__strftime_lZ_iiiiii)(u32, u32, u32, u32, u32) = NULL;
/* import: 'env' '_time' */
u32 (*Z_envZ__timeZ_ii)(u32) = envZ__timeZ_ii;
/* import: 'env' 'abortOnCannotGrowMemory' */
u32 (*Z_envZ_abortOnCannotGrowMemoryZ_ii)(u32) = NULL;
/* import: 'env' 'setTempRet0' */
void (*Z_envZ_setTempRet0Z_vi)(u32) = envZ_setTempRet0Z_vi;
/* import: 'env' 'invoke_iiiiij' */
u32 (*Z_envZ_invoke_iiiiijZ_iiiiiiii)(u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_iiiijii' */
u32 (*Z_envZ_invoke_iiiijiiZ_iiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32) = envZ_invoke_iiiijiiZ_iiiiiiiii;
/* import: 'env' 'invoke_iij' */
u32 (*Z_envZ_invoke_iijZ_iiiii)(u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_ji' */
u32 (*Z_envZ_invoke_jiZ_iii)(u32, u32) = envZ_invoke_jiZ_iii;
/* import: 'env' 'invoke_jii' */
u32 (*Z_envZ_invoke_jiiZ_iiii)(u32, u32, u32) = NULL;
/* import: 'env' 'invoke_jiiii' */
u32 (*Z_envZ_invoke_jiiiiZ_iiiiii)(u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_jiiijj' */
u32 (*Z_envZ_invoke_jiiijjZ_iiiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_jij' */
u32 (*Z_envZ_invoke_jijZ_iiiii)(u32, u32, u32, u32) = envZ_invoke_jijZ_iiiii;
/* import: 'env' 'invoke_viiij' */
void (*Z_envZ_invoke_viiijZ_viiiiii)(u32, u32, u32, u32, u32, u32) = envZ_invoke_viiijZ_viiiiii;
/* import: 'env' 'invoke_viij' */
void (*Z_envZ_invoke_viijZ_viiiii)(u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_viijji' */
void (*Z_envZ_invoke_viijjiZ_viiiiiiii)(u32, u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' 'invoke_vijji' */
void (*Z_envZ_invoke_vijjiZ_viiiiiii)(u32, u32, u32, u32, u32, u32, u32) = NULL;
/* import: 'env' '__table_base' */
u32 (*Z_envZ___table_baseZ_i) = NULL;
/* import: 'env' 'DYNAMICTOP_PTR' */
u32 (*Z_envZ_DYNAMICTOP_PTRZ_i) = NULL;
/* import: 'global' 'NaN' */
f64 (*Z_globalZ_NaNZ_d) = NULL;
/* import: 'global' 'Infinity' */
f64 (*Z_globalZ_InfinityZ_d) = NULL;
/* import: 'env' 'memory' */
wasm_rt_memory_t (*Z_envZ_memory) = &w2c_memory;
/* import: 'env' 'table' */
wasm_rt_table_t (*Z_envZ_table) = &w2c_table;



```

主要还是看看预设动态内存基址的部分

![](https://article-images.zsxq.com/FhUdb-5eSwP16N2GvDVu0ZLLXygc)

设置的方法和上一个案例是类似的，只需要修改偏移值和 4 个字节的值就可以。但是这里返回的是一个完整的 ts 字节，那么就会出现中间有 0 字节的情况，如果按照常规的使用 char * 类型，那么就会在出现 0 字节的时候被截断，不能返回完整的 ts，所以这里需要多一步把 ts 字节转为 16 进制再返回，那么在 ts 中间就肯定不会出现 0 字节了

```
#include <stdio.h>
#include <stdlib.h>
#include "u5kk.c"

extern void init_wasm(void);
extern int set_m3u8(char*, int);
extern char* decrypt(char*, int, int, int);

void init_wasm(){
    init_func_types();
    init_globals();
    init_memory();
    init_table();
    init_exports();
    w2c___GLOBAL__I_000261();
    w2c___GLOBAL__I_000262();
    w2c___GLOBAL__I_000263();
    w2c___GLOBAL__I_000275();
    w2c___GLOBAL__I_000276();
    w2c___GLOBAL__I_000277();
    w2c___GLOBAL__I_000300();
    w2c___GLOBAL__I_000301();
    w2c___GLOBAL__I_000310();
    w2c___GLOBAL__I_000311();
    w2c____cxx_global_var_init_25();
    w2c___GLOBAL__sub_I_nup2p_cpp();
    w2c___GLOBAL__sub_I_em_web_socket_cpp();
    w2c___GLOBAL__sub_I_em_web_rtc_cpp();
    w2c___GLOBAL__sub_I_em_request_cpp();
    w2c___GLOBAL__sub_I_em_config_storage_cpp();
    w2c___GLOBAL__sub_I_bit64_cpp();
    w2c___GLOBAL__sub_I_tracker_cpp();
    w2c___GLOBAL__sub_I_packer_cpp();
    w2c___GLOBAL__sub_I_phoenix_socket_cpp();
    w2c___GLOBAL__sub_I_wrap_cpp();
    w2c___ZNSt3__210shared_ptrIN2nu8__holderEE18__enable_weak_thisEz();
    w2c___ZNSt3__210shared_ptrIN2nu8__holderEE18__enable_weak_thisEz();
    w2c___ZNSt3__210shared_ptrIN2nu8__holderEE18__enable_weak_thisEz();
    w2c___ZNSt3__210shared_ptrIN2nu8__holderEE18__enable_weak_thisEz();
    w2c___ZNSt3__210shared_ptrIN2nu8__holderEE18__enable_weak_thisEz();
    w2c____emscripten_environ_constructor();
}

int set_m3u8(char* m3u8, int m3u8_len){
    int n2 = w2c_stackAlloc(m3u8_len);
    memcpy(w2c_memory.data + n2, m3u8, m3u8_len);
    return w2c__nup2p_set_m3u8_file(n2);
}

int min(int a, int b){
    if (a < b){
        return a;
    }else{
        return b;
    }
}

void ByteToHexStr(char* source, char* dest, int sourceLen)
{
    char highByte, lowByte;
    for (int i = 0; i < sourceLen; i++)
    {
        int temp = (int)source[i];
        if (temp < 0){
            temp += 256;
        }
        highByte = temp >> 4;
        lowByte = source[i] & 0x0f ;
        highByte += 0x30;
        if (highByte > 0x39)
            dest[i * 2] = highByte + 0x07;
        else
            dest[i * 2] = highByte;
        lowByte += 0x30;
        if (lowByte > 0x39)
            dest[i * 2 + 1] = lowByte + 0x07;
        else
            dest[i * 2 + 1] = lowByte;
    }
}

char* decrypt(char* ts, int ts_len, int m3u8_ptr, int m3u8_len){
    int mem = w2c__malloc(ts_len);
    for (int a = 0; a < ts_len;) {
        int o = min(ts_len - a, 131072);
        memcpy(w2c_memory.data + mem, ts + a, o);
        w2c__nup2p_process_segment(m3u8_ptr, mem, m3u8_len - 1 + a, o);
        memcpy(ts + a, w2c_memory.data + mem, o);
        a += o;
    }
    w2c__free(mem);

    char* out_ts_hex = (char *)malloc(ts_len * 2 + 1);
    ByteToHexStr(ts, out_ts_hex, ts_len);
    out_ts_hex[ts_len * 2 + 1] = 0;
    return out_ts_hex;
}


```

编译为 dll

```
ash
"D:/MinGW64/bin/gcc" -shared -Os -o u5kk.dll main.c wasm-rt-impl.c


```

python 进行调用

```
ython
    dll = ctypes.windll.LoadLibrary('u5kk.dll')
    print(dll)
    dll.init_wasm()
    with open('index.m3u8', 'rb') as f:
        m3u8_file = f.read()
    dll.set_m3u8.argtypes = [ctypes.c_char_p, ctypes.c_int]
    e = dll.set_m3u8(ctypes.c_char_p(m3u8_file), len(m3u8_file))
    print(e)
    with open('0.ts', 'rb') as f:
        ts_file = f.read()
    dll.decrypt.argtypes = [ctypes.c_char_p, ctypes.c_int, ctypes.c_int, ctypes.c_int]
    dll.decrypt.restype = ctypes.c_char_p
    t = dll.decrypt(ctypes.c_char_p(ts_file), len(ts_file), e, len(m3u8_file))
    with open('1.ts', 'wb') as f:
        f.write(bytes.fromhex(t.decode()))


```

因为在 c 中我们把 ts 转换为 16 进制，所以在 python 中需要多一步 bytes.fromhex 把 16 进制的转回字节，最终视频正常播放