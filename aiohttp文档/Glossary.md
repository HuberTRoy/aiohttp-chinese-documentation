# 相关名词释义
**aiodns**      
&ensp;&ensp;&ensp; 异步DNS解决方案    
&ensp;&ensp;&ensp; https://pypi.python.org/pypi/aiodns

**asyncio**      
&ensp;&ensp;&ensp; 写单线程多并发代码的协程库，通过套接字和其他资源进行多路复用的I/O访问，运行网络客户端和服务器 ，还有其他相关内容。      
&ensp;&ensp;&ensp; 相关参考信息请看 PEP 3156      
&ensp;&ensp;&ensp; https://pypi.python.org/pypi/asyncio/ 

**callable**     
&ensp;&ensp;&ensp; 任何可调用的对象。可以使用`callable()`进行检测是不是一个callable对象。     

**cchardet**      
&ensp;&ensp;&ensp; `cChardet`是一个高速的通用编码发现引擎。     
https://pypi.python.org/pypi/cchardet/    

**chardet**     
通用编码发现引擎。      
https://pypi.python.org/pypi/chardet/      

**gunicorn**       
Gunicorn  “绿色独角兽”是一个UNIX下的Python WSGI HTTP 服务器。         
http://gunicorn.org/     

**IDNA**       
应用程序中的国际化域名（IDNA）是为了编码互联网域名的一个行业标准，基于特定的语言模板和字母，如阿拉伯语，汉语，西里尔文，泰米尔文，希伯来文或以拉丁字母为基础的变音字符还有像法语一样的连字字符。这些写作系统经由电脑编码成多字节Unicode字符。国际化的域名以ASCII的形式保存到域名系统中，使用Punycode编码互转。    

**keep-alive**     
一个可以让HTTP客户端和服务器间保持通讯的技术。一旦进行连接并发送完一次响应后并不关闭连接，而是保持打开状态以使用同一个套接字发送下一次请求。     
此技术可以让通讯变得飞快，因为无需为每次请求都建立一遍连接。    

**nginx**     
Nginx [engine x] 是一个HTTP和反向代理服务器，邮件代理服务器和通用 TCP/UDP代理服务器。     
https://nginx.org/en/     

**percent-encoding**       
是一个在URL中编码信息的机制，当URL中有不合适的字符时一般会进行编码。     

**requoting**      
对不合理的字符进行百分号编码（percent-encoding）或编码后的字符进行解码。     

**resource**     
一个映射HTTP路径的概念，每个资源（resource）相当于一个URI。       
可以有一个独立的名称。       
基于不同的HTTP方法，会包含不同的路由。

**route**      
资源（resource）的一部分，资源（resource）的路径和HTTP方法组成路由（route）。       

**web-handler**     
用于返回HTTP响应的终端。      

**websocket**      
是一个在同一个TCP连接上进行全双工通讯的协议。WebSocket协议由IETF标准化写入RFC 6455。      

**yarl**      
一个操作URL对象的库。    

https://pypi.python.org/pypi/yarl


