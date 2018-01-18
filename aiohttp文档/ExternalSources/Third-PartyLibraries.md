# 第三方扩展包

aiohttp不仅仅是一个能发起HTTP请求和创建WEB服务器的库。同时还为广大包提供基础支持。    
本页面列出了这些使用aiohttp为基础的工具。    
你可以随时向我们提供你写的库，如果可以的话请写一个PR到 https://github.com/aio-libs/aiohttp/.
*  :smile:为什么你会想把你自己的牛X代码提供给我们呢？
*  因为这样能提高曝光率啊~。

# 官方支持的包
本列表列出了由aio-libs官方团队所创建维护的包。主页是：https://github.com/aio-libs

## aiohttp扩展
* aiohttp-session   是一个为aiohttp.web提供会话功能的扩展包。
* aiohttp-debugtoolbar 是一个为aiohttp.web提供调试工具栏的扩展包。
* aiohttp-security 是一个为aiohttp.web提供许可和验证功能的扩展包。
* aiohttp-devtools 是一个为aiohttp.web应用提供开发工具包的扩展包。
* aiohttp-cors 是一个为aiohttp提供CORS（跨域资源共享）支持的扩展包。
* aiohttp-sse 是一个为aiohttp提供服务器推送事件的扩展包。
* pytest-aiohttp 是一个为aiohttp提供pytest插件的扩展包。
* aiohttp-mako 是一个为aiohttp.web提供Mako模板支持的扩展包。
* aiohttp-jinja2 是一个为aiohttp.web提供Jinja2模板支持的扩展包。

## 数据库相关
* aiopg 可以以异步方式处理PostgreSQL的扩展包。
* aiomysql 可以以异步方式处理MySql的扩展包。
* aioredis 可以以异步方式处理Redis的扩展包。

## 其他工具
* aiodocker 基于asyncio和aiohttp的Python Docker API客户端。
* aiobotocore 使用aiohttp对botocore提供异步支持的库。

# 很赞的第三方库
这些库不在aio-libs中，不过写的很赞，强烈推荐使用~。   
* uvloop 使用它可以让asyncio运行得飞快，基于`libuv`。
强烈推荐使用它来代替标准asyncio。

## 数据库相关
* asyncpg 另一个可以以异步方式处理PostgreSQL的扩展包。它比aiopg快很多，不过它并不是开箱即用的——它的API有点怪。不管怎样请务必看一看，真滴非常快。

# 其他工具
这些包是基于aiohttp的，但不在上述两种类别中。    
至于好不好用，全凭君定——因为我们也不知道。

请现在这里添加你的库的介绍，之后在更改其状态。
* aiohttp-cache 是一个为aiohttp服务器提供缓存系统的扩展包。
* aiocache 是一个为多个后端提供异步缓存的扩展包。（未知框架）
* gain 是一个基于异步的爬虫框架。
* aiohttp-swagger 是关于aiohttp服务器使用Swagger的API文档。
* aiohttp-swaggerify 是一个为aiohttp终端自动生成Swagger 2.0定义的扩展包。
* aiohttp-validate 是一个帮助你确认你的API是否有效（JSON形式）的扩展包。
* raven-aiohttp raven-python版aiohttp。
* webargs 一个可以让解析HTTP请求参数变得容易的包。已被多个流行web框架内置。其中包括 Flask, Django, Bottle, Tornado, Pyramid, webapp2, Falcon和aiohttp。
* aioauth-client 为aiohttp提供OAuth客户端的扩展包。
* aiohttpretty 使用aiohttp制作的httpretty（一个可以伪造HTTP请求的包）的包。
* aioresponse 帮助你制造虚假web请求的包。
* aiohttp-transmute 是一个为aiohttp提供transmute（未知含义）。
* aiohttp_apiset 一个使用swagger规范创建路由的扩展包。
* aiohttp-login 是一个为aiohttp应用提供注册和授权功能的扩展包。
* aiohttp_utils 一个方便创建aiohttp.web应用的扩展包。
* aiohttpproxy 一个简单aiohttp HTTP代理工具。
* aiohttp_traversal 一个将基础路由转换给aiohttp.web的扩展包。
* aiohttp_autoreload 一个让aiohttp服务器在源码更改时可以实时重载的扩展包。
* gidgethub 异步Github API。
* aiohttp_jrpc aiohttp版 JSON-RPC服务。
* fbemissary 一个提供Facebook Messenger平台的功能的bot框架，基于asyncio和aiohttp。
* aioslacker 一个asyncio懒包装后的扩展包。
* aioreloader 一个可以将tornado重新加载到aiohttp上的扩展包。
* aiohttp_babel 为aiohttp提供Babel本地化支持的扩展包。
* python-mocket 一个套接字mock框架 - 支持所有种类的套接字应用，包括web客户端。
* aioraft 异步RAFT算法（可逆加成-断裂链转移聚合算法），基于aiohttp。
* home-assistant 开元中心自动化平台，基于Python 3。
* discord.py Discord客户端。
* async-v20 是一个为OANDA的v20API提供异步FOREX客户端扩展包。Python 3.6 +。

