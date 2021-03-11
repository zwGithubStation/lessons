##以下为极客时间的两日专题课程-《网络协议集训班》的内容总结 共15课时  

重要程度星级：  
S:非常重要  
A:比较重要  
N:一般重要  
P:了解即可  

# 课时一 在浏览器上输入URL到主页显示，经历了哪些步骤(S)  
URL的基本介绍(协议、域名、端口、资源路径等等，以及域名的进一步介绍),可参考：  
https://developer.mozilla.org/zh-CN/docs/Learn/Common_questions/What_is_a_URL  
https://www.verisign.com/en_US/website-presence/online/what-is-a-url/index.xhtml   
![Image text](https://raw.githubusercontent.com/zwGithubStation/books/master/Network_Protocal_two_days/pic/what_is_URL.png)

**1.1 浏览器发起HTTP请求的逻辑**  
1.1.1 浏览器输入URL或者点击链接URL后，第一步是**域名解析为IP地址** 
关于域名系统(DNS)的原理，可参考：  
https://draveness.me/dns-coredns/  
关于DNS主要使用UDP的历史渊源，可参考：  
https://draveness.me/whys-the-design-dns-udp-tcp/  
当然各级大致有自己的缓存，比如浏览器本身就可以缓存了域名对应的IP地址  
该流程对应PPT中的下图部分：  
![Image text](https://raw.githubusercontent.com/zwGithubStation/books/master/Network_Protocal_two_days/pic/what_is_URL.png)  

1.1.2 http请求转化为传入IP地址的一个TCP连接  
通过IP地址，建立与服务的连接，服务可以是一个的负载均衡入口，其后台有一个水平扩展的集群(各个server的数据/代码/程序一致，提供同样的服务能力)，根据具体的请求类型，可以在server上直行下盘的数据库操作、内存上的读写缓存操作、亦或是kafka队列操作(mySQL/redis/kafka属于在TCP之上的协议)  
当然，图片、文件、css请求等也可能通过CDN访问用户就近的server，其连接server的过程同质于上述流程  
该流程对应PPT中的下图部分：  
![Image text](https://raw.githubusercontent.com/zwGithubStation/books/master/Network_Protocal_two_days/pic/what_is_URL.png)  

1.1.3 当然一个URL请求大概率不会只触发一个服务请求/建立一个TCP连接，这与http请求的DOM树划分对应，如果DOM树分解产生多个各类请求(html/js/css等)，则会触发多个B.1.2介绍的服务请求；最终各个请求结果在浏览器上组装呈现。  
http请求的DOM树划分、请求结果的合并呈现，不作为理解重点，了解即可。  
该流程对应PPT中的下图部分：   
![Image text](https://raw.githubusercontent.com/zwGithubStation/books/master/Network_Protocal_two_days/pic/what_is_URL.png)  

**1.2 OSI协议分层体系**  
1.2.1 OSI七层网络协议分层  
![Image text](https://raw.githubusercontent.com/zwGithubStation/books/master/Network_Protocal_two_days/pic/what_is_URL.png)   

1.2.2 重要的5层  
![Image text](https://raw.githubusercontent.com/zwGithubStation/books/master/Network_Protocal_two_days/pic/what_is_URL.png)    

1.2.3 负载均衡所在的层级 7层负载均衡、4层负载均衡的概念  
图如B.2.1所示  
四层负载均衡工作在UDP/TCP上，不会去解析上层的诸如tsl/ssl(不编译openssl库)、http协议的内容，只会建立两个TCP连接(负载均衡与客户端、负载均衡与服务器)或两个UDP session.  
LVS、iptables/ipvs可以工作在网络层或传输层，区别是网络层就只有一个TCP连接/UDP session(客户端与服务器)，而传输层就存在两个TCP连接/UDP session(负载均衡与客户端、负载均衡与服务器)，当然他们都不会解析上层的诸如tsl/ssl(不编译openssl库)、http协议的内容  
如图Nginx、HAProxy、Envoy可以实现四层负载均衡,也可以实现七层负载均衡。 工作在七层则可以解析tsl/ssl、http协议，将http协议直接解析为cgi(Common Gateway Interface,可以理解为http请求最终在server上所触发的动作，可以是python脚本，也可以是C/C++程序)，抑或是解析转换为redis操作。 e.g. python的uWSGI  

1.2.4 http、TSL/SSL、TCP、IP诸协议的工作流程图见课件PPT，不再赘述  
http:客户端发请求，服务端做响应; req/res都包含头+资源  
TLS/SSL:公开的消息通道中进行加密传输. c和s先协商出只有双方才知道的消息秘钥，后面传输的实际内容不解密无法看到  
TCP: 三次握手; 数据分割多消息传输,可靠传输  
IP: 路由器寻址最优路径, icmp等辅助实现传输   


**1.3 通过抓包缩小问题边界**  
抓包可以通过层层的以太网帧头部信息(以太网层的MAC头，IP层的IP头，TCP层的tcp头，HTTP层的header)剥离，获取信息    
关键路径：  
API Gateway、WAF防火墙等；  
各层对于包信息的修改：  
IP路由器：修改IP层信息的TTL等；  
TCP(四层)反向代理设备(e.g.F5):新建一条TCP连接(反向代理与server，新链上的包信息都有区别)  
HTTP(七层)反向代理设备：新的HTTP请求(反向代理与server，新请求可能是短连接，而client的http请求可能是keepalive长连接)   

** 课时综述：**   
1.OSI将协议分层后，可以让软件实现层面解耦合，降低了系统复杂性，允许各层协议独立演进。   
2.浏览器依据DOM树及渲染规则，会发出多个HTTP请求。  
3.DNS协议可以将域名转换为IP地址。  
4.OSI协议分层只是参考概念，许多实际的协议无法与其一一对应。  
5.IP协议保障了主机与主机间的报文可达。  
6.传输层协议提供了进程与进程间的报文可达。  
7.TCP协议提供了可靠的字节流传输。  
8.TLS/SSL协议提供了机密性、完整性及通讯双方的身份鉴别。  

# 课时二 如何抓包分析网络报文   
**2.1 HTTP/1协议的语法格式**  
HTTP的协议格式、基于ABNF严格描述的HTTP协议格式：  
该部分的基本内容不再详述，可在其他课程中再做详细了解(e.g. 透视HTTP协议课程)  
**2.2 浏览器的Network抓包面板**   
同上，需使用时再做细致了解  
**2.3 GUI中Wireshark抓包工具的用法**   
BPF表达式的使用不做细化了解。 本节同上，需使用时再做细致了解  
**2.4 shell中的TCPDump抓包工具的用法**   
本节同上，需使用时再做细致了解    

# 课时三 RESTful API该如何设计  
**3.1 Restful API的设计原则**   
RESTful原则的细致了解，需要其他材料的辅助深化学习   
关于非侵入式/侵入式的服务治理，PPT上的总结比较重要  

**3.2 HTTP常见方法的含义**  
幂等方法的概念  
常见方法的介绍见PPT  
WEBDAV基本方法介绍见PPT，webdav的细致了解需要其他材料的辅助深化学习  
**3.3 HTTP常见响应码的含义**  
1xx、2xx、3xx、4xx、5xx这些错误码含义，基础见PPT，细致了解需要其他材料的辅助深化学习  
**3.4 HTTP常见头部的含义**  
七层Proxy转发HTTP消息流程：  
![Image text](https://raw.githubusercontent.com/zwGithubStation/books/master/Network_Protocal_two_days/pic/what_is_URL.png)   

如何传递客户端IP地址的问题见PPT标注  
长连接相较短连接的优势及使用场景，见PPT标注  
理解HTTP方法/响应码是设计符合RESTful架构API的基础，也使服务的可扩展性更好，也是k8s这样的第三方开源工具能更好管理PASS服务的基础  

# 课时四 哪些HTTP头部配合浏览器增强了Web安全   
暂时不做涉及  

# 课时五 WAF防火墙究竟是怎样工作的？  
暂时不做涉及  
WAF防火墙的三种工作方式见PPT标注  

# 课时六 HTTP/1协议该如何优化性能？  
**6.1 HTTP缓存的工作原理**  

**6.2 设置缓存的过期时间**  
**6.3 解决缓存不一致的乐观锁**  
**6.4 设置依赖资源间的过期时间**  
**6.5 开启多线程下载及断点续传功能**  
**6.6 HTTP资源的压缩传输**  


# 课时七 升级到HTTP2协议有哪些好处？  
**7.1 HTTP2的多路复用功能**  
**7.2 基于Protobuf、HTTP2的gRPC框架**   

# 课时八 下一代的HTTP3协议有哪些新特性？  
**8.1 HTTP2的队头阻塞问题**    

**8.2 HTTP3的连接迁移功能**    
**8.3 HTTP3有序字符流的实现**    
**8.4 QPACK对HTTP头部的压缩**    


# 课时九 TLS/SSL协议使用了哪些密码学知识？   
暂时不做涉及  

# 课时十 如何优化TLS/SSL协议的性能、安全性？   
暂时不做涉及  

# 课时十一 为何TCP连接要经由三次握手才能建立？   
**11.1 TCP协议的发展历史**  
**11.2 TCP连接的同步要素**  
**11.3 TCP状态机的变迁流程**  


# 课时十二 如何优化TCP协议的性能？  
**12.1 建立连接时的缓冲区队列**  
**12.2 关闭连接时的参数调优**  
**12.3 滑动窗口的工作原理**  
**12.4 TCP传输时的缓冲区调优**  


# 课时十三 TCP的自动分包会引入哪些性能问题？  
**13.1 MSS与MTU的区别**  
**13.2 丢包驱动的拥塞控制算法**  
**13.3 初始拥塞窗口的调整**  
**13.4 测量驱动的拥塞控制算法**  
**13.5 TCP字符流对消息边界的影响**   

# 课时十四 为什么必须使用UDP协议发送广播报文？  
**14.1 UDP与DNS协议**  
**14.2 IP多播功能**  
**14.3 IP地址的CIDR分段法**  
**14.4 内网、公网间的NAT端口转发**  


# 课时十五 如何抓包分析K8S容器网络中的报文？     
**15.1 ARP协议与MAC地址**  
**15.2 K8S容器网络架构**  
**15.3 容器网络的优缺点**  
**15.4 vxlan overlay网络**  



#问题集合  
1.socket的定位   
每一个tcp连接附一个数字，即socket句柄     
socket是内核给应用层提供的接口，在把IP/TCP层完成解封包、有序化等等工作后，提供给python/c++等等的接口.  
2.七层代理和四层代理  
七层都解析http协议，四层代理不解析http协议；解析则可做协议转换  
四层的负载均衡的方式有限，基于ip/端口，七层可以基于http头部、URL等做  
四层七层都可能解析TLS协议  
WAF大多工作于7层  
3.自定义一个应用层协议的主要关注点  
基于TCP还是UDP,每个请求是否大于1500字节(较小UDP更好，tcp为你提供token有序化服务，quic是基于UDP的自主有序化的勇士)，是否有很强的实效性(不能重发如视频用UDP)  
如果一个TCP连接上跑多个请求，需明确请求之间如何做区分(ascii[http]还是二进制[mysql、redis、http2、gRPC])  
协议如何暴露给整个监控体系，e.g. head/body体系保证监控/日志等体系可以为之服务  
4.HTTP2使用虚拟stream可以大大提升传输速度的关键  
浏览器使用http1协议发送请求，并发多个tcp连接，每个连接上一次发送一个请求，收到res后继续发送，中间有等待时间;   
http2协议单个tcp连接，软件层可以一直进行消息发送，最大程度使用了网络带宽   
