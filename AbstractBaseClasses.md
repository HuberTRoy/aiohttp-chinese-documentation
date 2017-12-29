# 抽象基类

## 抽象的路由类

aiohttp使用抽象类来管理web接口。
aiohttp.web中大部分类都不打算是可以继承的，只有几个是可以继承的。
aiohttp.web建立在这几个概念之上: 应用（application），路由（router），请求（request）和响应（response）。
路由（router）是一个可插拔的部分: 用户可以从头开始创建一个新的路由库，不过其他的部分必须与这个新路由无缝契合才行。
AbstractRouter只有一个必须（覆盖）的方法： AbstractRouter.resolve()协程方法。同时必须返回AbstractMatchInfo实例对象。
如果所请求的URL的处理器发现AbstractMatchInfo.handler()所返回的是web处理器，那AbstractMatchInfo.http_exception则为None。
否则的话AbstractMatchInfo.http_exception会是一个HTTPException实例，如404: NotFound 或 405: Method Not Allowed。如果调用AbstractMatchInfo.handler()则会抛出http_exception。

*class aiohttp.abc.AbstractRouter*
&ensp;&ensp;&ensp;此类是一个抽象路由类，可以指定aiohttp.web.Application中的router参数来传入此类的实例，使用aiohttp.web.Application.router来查看返回.
&ensp;&ensp;&ensp;*coroutine **resolve**(request)*  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;URL处理方法。该方法是一个抽象方法，需要被继承的类所覆盖。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;request - 用于处理请求的aiohttp.web.Request实例。在处理时aiohttp.web.Request.match_info会被设置为None。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**返回:**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回AbstractMathcInfo实例对象。


*class aiohttp.abc.AbstractMatchInfo*
&ensp;&ensp;&ensp;抽象的匹配信息，由AbstractRouter.resolve()返回。
&ensp;&ensp;&ensp;**http_exception**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;如果没有匹配到信息则是aiohttp.web.HTTPException否则是None。
&ensp;&ensp;&ensp;*coroutine **handler**(request)*
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;执行web处理器程序的抽象方法。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;request - 用于处理请求的aiohttp.web.Request实例。在处理时aiohttp.web.Request.match_info会被设置为None。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**返回:**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回aiohttp.web.StreamResponse或其派生类。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**可能会抛出的异常:**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;所抛出的异常类型是aiohttp.web.HTTPException。
&ensp;&ensp;&ensp;*coroutine expect_handler(request)*
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;用户处理100-continue的抽象方法。

## 抽象的基于类的视图类
aiohttp提供抽象类 AbstractView来对基于类的视图进行支持，而且还是一个awaitable对象（可以应用在await Cls()或yield from Cls()中，同时还有一个request属性。）

*class aiohttp.AbstarctView*
&ensp;&ensp;&ensp; 一个用于部署所有基于类的视图的基础类。
&ensp;&ensp;&ensp; \_\_iter\_\_和\_\_await\_\_方法需要被覆盖。
&ensp;&ensp;&ensp; **request**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;用于处理请求的aiohttp.web.Request实例。

## 抽象的Cookies Jar类

*class aiohttp.abc.AbstractCookieJar*
&ensp;&ensp;&ensp;此类所生成的cookie jar实例对象可以作为ClientSession.cookie_jar所需要的对象。
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
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;cookies - 接受collection.abc.Mapping对象（如dict, SimpleCookie等）或由服务器响应返回的可迭代cookies键值对。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;response_url(str) - 响应该cookies的URL，如果设置为None则该cookies为共享cookies。标准的cookies应该带有服务器URL，只在请求此服务器时发送，共享的话则会向全部的客户端请求都发送。
&ensp;&ensp;&ensp;*filter_cookies(request_url)*
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回jar中可以被此URL接受的cookies和可发送给给定URL的客户端请求的Cookies 头。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;request_url (str) - 想要查询cookies的请求URL。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**返回:**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回包含过滤后的cookies的http.cookies.SimpleCookie对象。

## 抽象访问日志
*class aiohttp.abc.AbstractAccessLogger*
&ensp;&ensp;&ensp; 所有RequestHandler中实现access_logger的基础类。
&ensp;&ensp;&ensp; log方法需要被覆盖。
&ensp;&ensp;&ensp;*log(request, response, time)*
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;**参数:**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;request - aiohttp.web.Request 对象。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;reponse - aiohttp.web.Response 对象。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;time (float) - 响应该请求的时间戳。