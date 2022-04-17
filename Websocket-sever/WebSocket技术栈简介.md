# WebSocket 服务器原理简介

> 参考文档：
>
> wsgi详解：https://blog.csdn.net/li_101357/article/details/52748323
>
> wsgi协议介绍（萌新版）：https://blog.csdn.net/j163you/article/details/80919360
>
> 廖雪峰web编程讲解：https://www.liaoxuefeng.com/wiki/897692888725344/923057027806560
>

## WSGI是什么？

WSGI是一种规范，它定义了使用python编写的web app(django)与web server（uWSGI）之间接口格式，实现web app与web server间的解耦。

WSGI 没有官方的实现, 因为WSGI更像一个协议. 只要遵照这些协议,WSGI应用(Application)都可以在任何服务器(Server)上运行

WSGI实质：WSGI是一种描述web服务器（如nginx，uWSGI等服务器）如何与web应用程序（如用Django、Flask框架写的程序）通信的规范、协议。

## 为什么需要web协议：

1）不同的框架有不同的开发方式，但是无论如何，开发出的应用程序都要和服务器程序配合，才能为用户提供服务。

2） 这样，服务器程序就需要为不同的框架提供不同的支持,只有支持它的服务器才能被开发出的应用使用，显然这是不可行的。

3）**web协议本质：**就是定义了Web服务器和Web应用程序或框架之间的一种简单而通用的接口规范。

## Web协议介绍

### Web协议出现顺序： CGI -> FCGI -> WSGI -> uwsgi

1. CGI： 最早的协议

2. FCGI： 比CGI快

3. WSGI： Python专用的协议
 4. uwsgi： 比FCGI和WSGI都快，是uWSGI项目自有的协议，主要特征是采用二进制来存储数据，
  议都是使用字符串，所以在存储空间和解析速度上，都优于字符串型协议.

## uwsgi是什么？

它是一个二进制协议，可以携带任何类型的数据。
一个uwsgi分组的头4个字节描述了这个分组包含的数据类型。
uwsgi是一种线路协议而不是通信协议，在此常用于在uWSGI服务器与其他网络服务器的数据通信；

### uWSGI是什么？（web服务器 和nginx类似）

什么是uWSGI： uWSGI是一个全功能的HTTP服务器，实现了WSGI协议、uwsgi协议、http协议等。

### uWSGI作用

**它要做的就是把HTTP协议转化成语言支持的网络协议，比如把HTTP协议转化成WSGI协议，让Python可以直接使用。

### uWSGI特点

轻量级，易部署，性能比nginx差很多

> 注：如果架构是Nginx+uWSGI+APP，uWSGI是一个中间件
>
> 　　如果架构是uWSGI+APP，uWSGI是一个服务器
>

### WSGI / uwsgi / uWSGI 这三个概念的区分

> WSGI是一种通信协议。通常用户django框架和uWSGI服务器之间通信。（如果说的更细致的话就是用来和Python WSGI model）
> uwsgi是一种线路协议而不是通信协议，在此常用于在uWSGI服务器与其他网络服务器的数据通信。
> 而uWSGI是实现了uwsgi和WSGI等协议的Web服务器

## nginx是什么？

1. Nginx是一个Web服务器,其中的HTTP服务器功能和uWSGI功能很类似
 2. 但是Nginx还可以用作更多用途，比如最常用的反向代理、负载均衡、拦截攻击等，而且性能极高

## django是什么？

1. Django是一个Web框架，框架的作用在于处理request和 reponse，其他的不是框架所关心的内容。

2. 所以如何部署Django不是Django所需要关心的。

### 请求处理整体流程

1. nginx接收到浏览器发送过来的http请求，将包进行解析，分析url
2. **静态文件请求：**就直接访问用户给nginx配置的静态文件目录，直接返回用户请求的静态文件
3. **动态接口请求：**那么nginx就将请求转发给uWSGI，最后到达django处理

### 各模块作用

1. **nginx：**是对外的服务器，外部浏览器通过url访问nginx，nginx主要处理静态请求

2. **uWSGI：**是对内的服务器，主要用来处理动态请求

3. **uwsgi：**是一种web协议，接收到请求之后将包进行处理，处理成wsgi可以接受的格式，并发给wsgi

4. **WSGI：**是python专用的web协议，根据请求调用应用程序（django）的某个文件，某个文件的某个函数

5. **django：**是真正干活的，查询数据等资源，把处理的结果再次返回给WSGI， WSGI 将返回值进行打包，打包成uwsgi能够接收的格式

6. **uwsgi**接收wsgi发送的请求，并转发给nginx,nginx最终将返回值返回给浏览器

### Django + uWSGI方案

1. 没有nginx而只有uwsgi的服务器，则是Internet请求直接由uwsgi处理，并反馈到web项目中。

2. nginx可以实现安全过滤，防DDOS等保护安全的操作，并且如果配置了多台服务器，nginx可以保证服务器的负载相对均衡。

3. 而uwWSGI则是一个web服务器，实现了WSGI协议(Web Server Gateway Interface)，http协议等，它可以接收和处理请求，发出响应等。所以只用uwsgi也是可以的。

###nginx和uWSGI特点

####1）nginx的作用

1.反向代理，可以拦截一些web攻击，保护后端的web服务器

2.负载均衡，根据轮询算法，分配请求到多节点web服务器

3.缓存静态资源，加快访问速度，释放web服务器的内存占用，专项专用

####  2）uWSGI的适用

1.单节点服务器的简易部署

2.轻量级，好部署
