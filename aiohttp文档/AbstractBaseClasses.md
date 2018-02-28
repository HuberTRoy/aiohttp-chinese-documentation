# 抽象基类

## 抽象路由类

`aiohttp`使用抽象类来管理web接口。  
`aiohttp.web`中大部分类都不打算是可以继承的，只有几个是可以继承的。  
`aiohttp.web`建立在这几个概念之上: 应用（application），路由（router），请求（request）和响应（response）。  
路由（router）是一个可插拔的部分: 用户可以从头开始创建一个新的路由库，不过其他的部分必须与这个新路由无缝契合才行。  
AbstractRouter只有一个必须（覆盖）的方法： `AbstractRouter.resolve()`协程方法。同时必须返回`AbstractMatchInfo`实例对象。  
如果所请求的URL的处理器发现`AbstractMatchInfo.handler()`所返回的是web处理器，那`AbstractMatchInfo.http_exception`则为None。  
否则的话`AbstractMatchInfo.http_exceptio`n会是一个`HTTPException`实例，如`404: NotFound` 或 `405: Method Not Allowed`。如果调用`AbstractMatchInfo.handler()`则会抛出`http_exception`。  

*class aiohttp.abc.AbstractRouter*   
&ensp;&ensp;&ensp;此类是一个抽象路由类，可以指定`aiohttp.web.Application`中的router参数来传入此类的实例，使用aiohttp.web.Application.router来查看返回.  
&ensp;&ensp;&ensp;*coroutine **resolve**(request)*    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;URL处理方法。该方法是一个抽象方法，需要被继承的类所覆盖。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;request - 用于处理请求的`aiohttp.web.Request`实例。在处理时`aiohttp.web.Request.match_info`会被设置为None。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**返回:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回`AbstractMathcInfo`实例对象。  


*class aiohttp.abc.AbstractMatchInfo*   
&ensp;&ensp;&ensp;抽象的匹配信息，由`AbstractRouter.resolve()`返回。   
&ensp;&ensp;&ensp;**http_exception**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;如果没有匹配到信息则是`aiohttp.web.HTTPException`否则是None。  
&ensp;&ensp;&ensp;*coroutine **handler**(request)*   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;执行web处理器程序的抽象方法。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;request - 用于处理请求的`aiohttp.web.Request`实例。在处理时`aiohttp.web.Request.match_info`会被设置为None。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**返回:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回`aiohttp.web.StreamResponse`或其派生类。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**可能会抛出的异常:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;所抛出的异常类型是`aiohttp.web.HTTPException`。  
&ensp;&ensp;&ensp;*coroutine expect_handler(request)*   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;用户处理100-continue的抽象方法。  
 
## 抽象类基础视图
`aiohttp`提供抽象类 `AbstractView`来对基于类的视图进行支持，而且还是一个`awaitable`对象（可以应用在`await Cls()`或`yield from Cls()`中，同时还有一个`request`属性。）  
  
*class aiohttp.AbstarctView*   
&ensp;&ensp;&ensp; 一个用于部署所有基于类的视图的基础类。  
&ensp;&ensp;&ensp; \_\_iter\_\_和\_\_await\_\_方法需要被覆盖。  
&ensp;&ensp;&ensp; **request**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;用于处理请求的`aiohttp.web.Request`实例。  

## 抽象Cookies Jar类

*class aiohttp.abc.AbstractCookieJar*   
&ensp;&ensp;&ensp;此类所生成的cookie jar实例对象可以作为`ClientSession.cookie_jar`所需要的对象。  
&ensp;&ensp;&ensp;jar中包含用于存储内部cookie数据的Morsel组件。  
&ensp;&ensp;&ensp;提供一个统计所存储的cookies的API:  
```
len(session.cookie_jar)
```
&ensp;&ensp;&ensp;jar也可以被迭代:  
```
for cookie in session.cookie_jar:
    print(cookie.key)
    print(cookie["domain"])
```
&ensp;&ensp;&ensp;此类用于存储cookies，同时提供**collection.abc.Iterable**和**collections.abc.Sized**的功能。  
&ensp;&ensp;&ensp;*update_cookies(cookies, response_url=None)*   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;更新由服务器返回的Set-Cookie头所设置的cookies。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;cookies - 接受`collection.abc.Mapping`对象（如dict, SimpleCookie等）或由服务器响应返回的可迭代cookies键值对。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;response_url(str) -   响应该cookies的URL，如果设置为None则该cookies为共享cookies。标准的cookies应该带有服务器URL，只在请求此服务器时发送，共享的话则会向全部的客户端请求都发送。  
&ensp;&ensp;&ensp;*filter_cookies(request_url)*   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回jar中可以被此URL接受的cookies和可发送给给定URL的客户端请求的Cookies 头。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;request_url (str) - 想要查询cookies的请求URL。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**返回:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回包含过滤后的cookies的`http.cookies.SimpleCookie`对象。  

## 抽象访问日志
*class aiohttp.abc.AbstractAccessLogger*   
&ensp;&ensp;&ensp; 所有`RequestHandler`中实现`access_logger`的基础类。  
&ensp;&ensp;&ensp; log方法需要被覆盖。  
&ensp;&ensp;&ensp;*log(request, response, time)*   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;request - `aiohttp.web.Request` 对象。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;reponse - `aiohttp.web.Response` 对象。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;time (float) - 响应该请求的时间戳。  