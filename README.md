RxFace
=====================


## Overview

这是一个人脸识别的简单 Demo, 使用了 [FacePlusPlus](http://www.faceplusplus.com.cn/) 的接口。他们的`/detection/detect` 人脸识别接口可以使用普通的 `get` 也可以用 `post` 传递图片二进制流的形式。其中 `post` 的时候遇到了相当多的坑，下面会提。

该 demo 的网络请求库使用了 [Retrofit](https://github.com/square/retrofit) 并集成了 [OkHttp](https://github.com/square/okhttp)，使用 [RxJava](https://github.com/ReactiveX/RxJava) 进行封装，方便以流的形式处理网络回调以及图片处理，View 的注入框架用了 [ButterKnife](https://github.com/JakeWharton/butterknife)，图片加载使用 [Glide](https://github.com/bumptech/glide)。


## Difficult point

当直接使用 `get` 通过传图片 Url 拿到人脸识别数据的话是相当简单的，如下请求链接只要使用 `Retrofit` 的 `get` 请求的 `@QueryMap` 传递参数即可

`http://apicn.faceplusplus.com/v2/detection/detect?api_key=7cd1e10dc037bbe9e6db2813d6127475&api_secret=gruCjvStG159LCJutENBt6yzeLK_5ggX&url=http://imglife.gmw.cn/attachement/jpg/site2/20111014/002564a5d7d21002188831.jpg`

主要存在的困难点是，当获取本地图片，再使用 `post` 传二进制图片数据时，`post` 要使用 `MultipartTypedOutput`，可参考 [stackoverflow](http://stackoverflow.com/questions/25249042/retrofit-multiple-images-attached-in-one-multipart-request/25260556#25260556) 的回答。然而，这样并没有结束，根据 FacePlusPlus 提供的 SDK Sample 里的 Httpurlconnection 的得到的请求头是这样的：

 `[Content-Disposition: form-data; name="api_key", Content-Type: text/plain; charset=US-ASCII, Content-Transfer-Encoding: 8bit]`
 
 `[Content-Disposition: form-data; name="img"; filename="NoName", Content-Type: application/octet-stream, Content-Transfer-Encoding: binary]`
 
 而使用 `Retrofit` 默认实现的话，得到的String参数的头是这样的
 ```java
Content-Disposition: form-data; name="api_key"
Content-Type: text/plain; charset=UTF-8
Content-Length: 32
Content-Transfer-Encoding: 8bit
 ``` 
 
 所以需要重写三个类:`CustomMultipartTypedOutput` 增加 boundary 的设置, `AsciiTypeString` 编码格式改为 US-ASCII, `CustomTypedByteArray` 设置其 fileName 为 "NoName"。
 
 同时需要注意的是在设置 `RestAdapter` 的 header 时，其 boundary 一定要和 `CustomMultipartTypedOutput` 的 boundary 相同，否则服务端无法匹配的！（这个地方，一时没注意，被整了一个多小时才发现！！） 
 


## Preview

![image_screen](/images/image_screen.png)

![movie_screen](/images/movie_screen.gif)


## More about me

* [MrFu-傅圆的个人博客](http://mrfu.me/)



## Acknowledgments

* [Glide](https://github.com/bumptech/glide) -Glide
* [Retrofit](https://github.com/square/retrofit) - Retrofit
* [OkHttp](https://github.com/square/okhttp) - OkHttp
* [RxJava](https://github.com/ReactiveX/RxJava) - RxJava
* [ButterKnife](https://github.com/JakeWharton/butterknife) - ButterKnife



License
============

    Copyright 2015 MrFu

	Licensed under the Apache License, Version 2.0 (the "License");
	you may not use this file except in compliance with the License.
	You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

	Unless required by applicable law or agreed to in writing, software
	distributed under the License is distributed on an "AS IS" BASIS,
	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	See the License for the specific language governing permissions and
	limitations under the License.