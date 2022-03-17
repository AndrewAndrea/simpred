> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43411585/article/details/123160733)

### 目录

*   *   *   *   [一、案例分析](#_1)
            *   [二、案例解析](#_7)

#### 一、案例[分析](https://so.csdn.net/so/search?q=%E5%88%86%E6%9E%90&spm=1001.2101.3001.7020)

*   如图弹幕信息，弹幕信息是 protobuf 序列化后的数据  
    ![](https://img-blog.csdnimg.cn/1655c782b73e4ce9b80f22ffd25f37ac.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![](https://img-blog.csdnimg.cn/25c035fade46440eb8c8a662e7444ab8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   响应头也有明显特征：`content-type: application/protobuffer`  
    ![](https://img-blog.csdnimg.cn/ca1f83f2881e4dadb145008041f825f4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 二、案例解析

*   关于 protobuf 案例的更详细介绍看[这篇文章](https://blog.csdn.net/weixin_43411585/article/details/123151047?spm=1001.2014.3001.5501)
*   这里直接安装`pip install blackboxprotobuf`，先用 fiddler 抓包，然后选择响应当中的十六进制字节黑色字体右击存为. bin 文件，然后按如下脚本解析  
    ![](https://img-blog.csdnimg.cn/f13941b51dbf4ccf869c9fffd0eddce3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![](https://img-blog.csdnimg.cn/42586cc4c2d6431096b79c519fb2057f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   如果是响应请求的话，就按如下脚本解析即可
    
    ```
    import requests
    import blackboxprotobuf
    response = requests.get("", headers={}, timeout=10)
    deserialize_data, message_type = blackboxprotobuf.protobuf_to_json(response.content)
    print(f"原始数据: {deserialize_data}")
    print(f"消息类型: {message_type}") 
    
    ```
    
*   相关文章推荐：[文章 1](https://blog.csdn.net/qq_36443294/article/details/119679639)，[文章 2](https://blog.csdn.net/didadidadada/article/details/119836515?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_aa&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_aa&utm_relevant_index=2)，[文章 3](https://blog.csdn.net/loveljsheng/article/details/120631773?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&utm_relevant_index=2)