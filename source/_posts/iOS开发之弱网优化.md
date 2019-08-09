---
title: iOS开发之弱网优化
date: 2019-08-05 18:42:44
tags:
	- iOS 
	- 弱网
---

首先我们要区分网络状态和弱网络状态的区别。

##### 网络状态

苹果监测网络状态，我们通常使用Reachablity框架或着使用AFNetworking中AFNetworkReachabilityManager来监测当前网络是否可用，是在WAN还是WiFi下。但在这里我可以告诉大家，通过以上方法无法检测实际的网络状态，它只是检测的是本地连接状态。在如下场景下用以上方法行不通的：
 1.现在很流行的公用wifi，需要网页鉴权，鉴权之前无法上网，但本地连接已经建立；
 2.存在了本地网络连接，但信号很差，实际无法连接到服务器；
 3.iOS连接的路由设备本身没有连接外网。
 在上面的情形下，使用上面的两种方法可能仍然返回网络可用。这里推荐一个框架 [RealReachability](https://github.com/dustturtle/RealReachability)。题外话就不啰嗦了。

##### 弱网络状态

首先可以肯定的是有网的，有流量产生的。我们可以使用Ping值来检测当前网络是否为弱网。
 *ping* 值是指，从PC对网络服务器发送数据到接收到服务器反馈数据的时间。也就是我们APP数据请求过去，再回来，所需要的时间，就是我们常说的ping值。大家可以查看下面的文章来了解[什么是PING值，PING值的计算方法](https://blog.csdn.net/wawa1203/article/details/53186365
)。
 苹果自己提供了 [SimplePing](https://developer.apple.com/library/archive/samplecode/SimplePing/Introduction/Intro.html#//apple_ref/doc/uid/DTS10000716) 封装了 ping 的功能，它利用 resolve host，create socket(send & recv data), 解析 ICMP 包验证 checksum 等实现了 ping 功能。并且支持 iPv4 和 iPv6。
 这样我们就可以在正式环境中，通过ping值来准确获取当前网络状况了。



###  弱网状态如何优化

#### 接口设计优化

* 慢查询监控

* 多查询优化

* 常用API cache，对于常用的接口进行缓存处理

* 多接口合并：所谓的多接口合并，也就是某个页面内请求过多，也可以考虑做一定的请求合并

#### 数据压缩处理

* 对数据进行gzip压缩

* 精简数据格式，如JSON代替XML，WebP代替其他图片格式

* 针对不同的设备不同的网络返回不同的数据格式

  2/3G使用低清晰度图片——>下发300X240，精度为80的图片
  4G普通清晰度图片——>下发600X480，精度为80的图片
  WiFi高清晰度图片（最好根据网速来判断，wifi也有慢的）——>下发600X480，精度为100的图片

#### 数据缓存

对首页及特定一级页面进行数据缓存，在一定的有效时间内再次请求可以直接从缓存读取数据，也可避免空白页出现影响体验

#### 界面优化

* 针对弱网（移动网络），不自动加载图片
* 界面先反馈，请求延迟





## Reference

[iOS网络深度优化总结](https://www.jianshu.com/p/a470ab485e39)



最近对网络优化进行了一些研究，好些都没有去实践，所以做一个整理，以后慢慢研究

### HTTP2.0

**HTTP2.0新特性**

- 二进制分帧
- 首部压缩
- 多路复用
- 服务器推送
- 请求优先级

[HTTP/2 新特性浅析](https://links.jianshu.com/go?to=http%3A%2F%2Fio.upyun.com%2F2015%2F05%2F13%2Fhttp2%2F)     
 [HTTP2.0原理详细分析](https://links.jianshu.com/go?to=http%3A%2F%2Fblog.csdn.net%2Fzhuyiquan%2Farticle%2Fdetails%2F69257126)    
 [什么是HTTP2.0协议：HTTP2.0协议详解](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.evssl.cn%2Fev-ssl-news2%2Fhttp2-0.html)   
 [HTTP 2.0 协议详解](https://links.jianshu.com/go?to=http%3A%2F%2Fblog.csdn.net%2Fzqjflash%2Farticle%2Fdetails%2F50179235)   
 [HTTP/2 头部压缩技术介绍](https://links.jianshu.com/go?to=https%3A%2F%2Fimququ.com%2Fpost%2Fheader-compression-in-http2.html)   
 [HTTP/2笔记之帧](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.blogjava.net%2Fyongboy%2Farchive%2F2015%2F03%2F20%2F423655.html)   
 [HTTP 1.1学习笔记](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fsyfwhu%2Fp%2F6116277.html)

### 网络深度优化的点

- NSCache缓存、Last-Modified、ETag
- 失败重发、缓存请求有网发送
- DNS解析
- 数据压缩：protobuf，WebP
- 弱网：2G、3G、4G、wifi下设置不同的超时时间
- TCP对头阻塞：GOOGLE提出QUIC协议，相当于在UDP协议之上再定义一套可靠传输协议

**NSCache缓存、Last-Modified、ETag**     
 [iOS网络缓存扫盲篇--使用两行代码就能完成80%的缓存需求](https://www.jianshu.com/p/fb5aaeac06ef)

**失败重发、缓存请求有网发送**    
 [iOS网络模块优化（失败重发、缓存请求有网发送）](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.cnblogs.com%2Fziyi--caolu%2Fp%2F8176331.html)

**DNS解析**    
 [HTTPDNS 在 iOS 中的实践](https://links.jianshu.com/go?to=http%3A%2F%2Fnathanli.cn%2F2016%2F12%2F20%2Fhttpdns-%E5%9C%A8-ios-%E4%B8%AD%E7%9A%84%E5%AE%9E%E8%B7%B5%2F)   
 [APP端的网络优化（DNS优化，HTTP优化）](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.cnblogs.com%2Fziyi--caolu%2Fp%2F8058577.html)    
 [iOS网络请求优化之DNS映射](https://links.jianshu.com/go?to=http%3A%2F%2Fmrpeak.cn%2Fios%2F2016%2F01%2F22%2Fdnsmapping)    
 [可能是最全的iOS端HttpDns集成方案](https://www.jianshu.com/p/cd4c1bf1fd5f)     
 [NSURLProtocol 配hosts（内含例子）](https://links.jianshu.com/go?to=https%3A%2F%2F09jianfeng.github.io%2F2016%2F05%2F05%2FNSURLProtocol%E7%BB%99%E8%AF%B7%E6%B1%82%E5%8A%A0host%2F)     
 [Swift - 拦截Alamofire的网络请求（缓存请求结果，从缓存中读取数据）](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.hangge.com%2Fblog%2Fcache%2Fdetail_1171.html)      
 [移动解析HTTPDNS在App开发中实践总结](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.iosfly.com%2F2016%2F12%2F03%2FHTTPDNS%2F)     
 [AFNetworking 原作者都无法解决的问题: 如何使用ip直接访问https网站?](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000004359232%3Fspm%3Da2c4g.11186623.2.4.qAGHxt)

**弱网优化**  
 [海量之道系列文章之弱联网优化 （一）](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1005365)  
 [海量之道系列文章之弱联网优化 （二）](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1005367)  
 [海量之道系列文章之弱联网优化 （三）](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1005368)  
 [海量之道系列文章之弱联网优化 （四）](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1005370)   
 [海量之道系列文章之弱联网优化 （五）](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1005371)  
 [海量之道系列文章之弱联网优化 （六）](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1005372)   
 [海量之道系列文章之弱联网优化 （七）](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1005373)  
 [移动端IM开发者必读(一)：通俗易懂，理解移动网络的“弱”和“慢”](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.52im.net%2Fthread-1587-1-1.html)   
 [移动端IM开发者必读(二)：史上最全移动弱网络优化方法总结](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.52im.net%2Fthread-1588-1-1.html)     
 [弱网下移动端网络连接处理策略](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000006733978)     
 [iOS开发——实时监控网速](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fyyt-hehe-yyt%2Fp%2F5954009.html)

**深度优化概述**       
 [携程App的网络性能优化实践](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.infoq.com%2Fcn%2Farticles%2Fhow-ctrip-improves-app-networking-performance%2F%23)    
 [移动 APP 网络优化概述](https://links.jianshu.com/go?to=http%3A%2F%2Fblog.cnbang.net%2Ftech%2F3531%2F)      
 [深度优化iOS网络模块](https://links.jianshu.com/go?to=http%3A%2F%2Fmrpeak.cn%2Fblog%2Fios-network%2F)
 [iOS网络请求优化](https://links.jianshu.com/go?to=https%3A%2F%2Frenchao0711.github.io%2F2017%2F08%2F29%2FiOS%E7%BD%91%E7%BB%9C%E4%BC%98%E5%8C%96%2F)        
 [《携程移动APP架构优化之旅》-陈浩然](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.docin.com%2Fp-1399719859.html)    
 [移动端网络常见问题及优化对策](https://links.jianshu.com/go?to=http%3A%2F%2Fios.jobbole.com%2F93110%2F)       
 [美团点评移动网络优化实践](https://links.jianshu.com/go?to=https%3A%2F%2Ftech.meituan.com%2FSharkSDK.html)   
 [无线性能优化：域名收敛](https://links.jianshu.com/go?to=http%3A%2F%2Ftaobaofed.org%2Fblog%2F2015%2F12%2F16%2Fh5-performance-optimization-and-domain-convergence%2F%3Futm_source%3Dtuicool)
 [iOS网络优化](https://www.jianshu.com/p/54e93303f0d7)      
 [携程移动端架构演进与优化之路](https://links.jianshu.com/go?to=http%3A%2F%2Fsacc.it168.com%2FPPT2016%2F%E4%B8%9316-4-%E5%8D%97%E5%BF%97%E6%96%87.pdf)   
 [App的网络测试中性能优化方案](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000010001767)   
 [URLSession如何动态控制并发数？](https://www.jianshu.com/p/473a55102716)  
 [传输速度优化方案](https://links.jianshu.com/go?to=http%3A%2F%2Fdzpqzb.com%2F2017%2F10%2F17%2F%E4%BC%A0%E8%BE%93%E9%80%9F%E5%BA%A6%E4%BC%98%E5%8C%96%E6%96%B9%E6%A1%88%2F)    
 [58 同城 iOS 客户端网络框架的演进之路](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fbyeweiyang%2Farticle%2Fdetails%2F80128027)  
 [IM 即时通讯技术在多应用场景下的技术实现，以及性能调优（iOS视角）](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FChenYilong%2FiOSBlog%2Fissues%2F6)  
 [网络请求优化之取消请求](https://www.jianshu.com/p/20f6172524d6)  
 [谈谈 iOS 网络层设计](https://www.jianshu.com/p/fe0dd50d0af1)
 [百度App网络深度优化系列《一》DNS优化](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUxMzk2ODI1NQ%3D%3D%26mid%3D2247483654%26idx%3D1%26sn%3D4cac066619fef001a1aed3616ad862af%26chksm%3Df94c5016ce3bd900562243c721ae95b5715fdd2f068748216d8f5a79a960164b6bff85844337%26mpshare%3D1%26scene%3D23%26srcid%3D%23rd)  
 [百度App网络深度优化系列《二》连接优化](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUxMzk2ODI1NQ%3D%3D%26mid%3D2247483700%26idx%3D1%26sn%3D84b5807b69f680504157b507eaa6c382%26chksm%3Df94c5024ce3bd932fb41da6343832dae327a26be7f9223a3569068967f7dda8e127b8055ae97%26mpshare%3D1%26scene%3D23%26srcid%3D%23rd)
 [新 Uber 司机端是如何克服网络延迟问题](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5c7ce6b7e51d453bf07b3508)
 [网络优化](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5cbe9133f265da038733a0db)



[百度 App 网络深度优化系列（一）：DNS 优化](https://www.infoq.cn/article/3QZ0o9Nmv*O0LoEPVRkN)

[百度 App 网络深度优化系列《三》弱网优化](https://www.infoq.cn/article/pQmLUECekW*DsymqbGvy)