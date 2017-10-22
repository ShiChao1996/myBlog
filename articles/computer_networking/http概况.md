# web 和 http 
### 1、http 概况
web的应用层传输协议是**超文本传输协议**（HyperText Transfer Protocol，http），它是web的核心。
http由两个程序实现：一个客户端程序和一个服务器程序，他们分别运行在不同的端系统，通过交换 `http` 报文进行会话。

http 使用 tcp 作为它的支撑运输协议（不是UDP）。http客户端首先发起一个与服务端的TCP连接，一旦连接建立，该浏览器就可以和服务器进程通过**套接字**访问TCP。
一旦用户向他的套接字接口发送了一个请求报文，该报文就脱离了客户端控制并进入TCP的控制

注意：服务端向客户发送请求的报文，但不保存任何关于该客户的状态信息。也就是说，HTTP是一个**无状态连接协议**

### 2、非持续性连接和持续性连接
可以通过请求头里的 `Connection` 参数为`keep-alive`来使TCP保持持续性链接。
##### 非持续性的HTTP
TCP 连接在服务器发送一个对象后关闭，该链接并不会为其他对象而持续下来。
##### 持续性连接
TCP 连接在服务器响应后保持该TCP连接打开。一般来说，一条链接经过一定时间间隔（可配置的超时间隔）仍未被使用，HTTP服务将关闭该连接。

### 3、报文格式
##### HTTP报文请求
下面是一个典型的 HTTP 报文请求：
```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```
该报文是用普通的`ASCII`文本书写的。第一行叫请求行（request line），剩下的叫首部行（header line）。
请求行有三个字段：请求方法、url、HTTP版本
![](http://img2.shangxueba.com/img/ku/20140828/13/9F5BE2A9270E4408D7C9F7D38AA28C19.png)

##### 响应报文
响应报文有三个部分：初始状态行（status line）、首部行、实体体（entity body）
![](https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3234511298,4292588505&fm=27&gp=0.jpg)

### 4、用户与服务器的交互：cookie
前面提到HTTP是无状态的，无法记录用户信息。但是通常一个web站点需要能够识别用户，希望将内容与用户身份联系起来，为此，HTTP使用了cookie。

cookie 技术有四个组件：
* 在HTTP响应报文中的一个cookie首部行
* 在HTTP请求报文中的一个cookie首部行
* 用户端系统中有一个cookie文件，由用户的浏览器进行管理
* 位于web站点的一个后端数据库
![](http://img.mp.itc.cn/upload/20160702/b014306d961742ea89c74c86f8d14e9f_th.png)

### web 缓存
web 缓存器也叫代理服务器，能够代表初始服务器来满足HTTP请求的网络实体。web服务器有自己的磁盘储存空间，并保存最近请求过的对象的副本。
![](http://www.2cto.com/uploadfile/Collfiles/20160407/20160407093112298.png)

web缓存器能大大减少对客户请求的响应时间，也能减少一个机构的接入链路到因特网的通信量。

另外，通过使用内容分发网络（Content Distribution Network, CDN）,wen缓存器正在因特网中发挥着越来越重要的作用。

### 条件 GET 方法
尽管高速缓存能够减少用户感受到的响应时间，但也引入了一个新的问题，即存放在缓存其中的对象副本可能是旧的，不是最新版本。需要一种机制，允许缓存器证实它的对象是最新的 —— 条件GET。
满足以下两点要求：
* 请求报文使用 GET 方法
* 请求报文中包含一个 “If-Modified-Since” 首部行

浏览器在想服务器发送请求的时候，由web缓存器代理请求web服务器，服务器发送响应给缓存器，缓存器将响应对象保留副本，并记录最后修改日期，再发送给浏览器。再有相同请求到达web缓存器时，web缓存器发送一个条件请求执行最新检查，例如：
```
GET /some/route HTTP/1.1
Host: www.whatever.com
If-Modified-Since: Wed, 7 Sep 2011 09:23:22
```
服务器可能的响应：
```
HTTP/1.1 304 NOT Modified
Date: Wed, 5 Oct 2011 09:23:22
Server: Apache/1.3.0 (Unix)
(empty entity body)
```
服务器的响应告知缓存器之前的副本有没有过期，如果没有，缓存器则返回副本给浏览器。如果服务器判断已过期，则直接返回新的响应对象。