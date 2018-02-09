为Python提供异步HTTP 客户端/服务端编程，基于<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-asyncio">asyncio(Python用于支持异步编程的标准库)</a>。

## 核心功能:

* 同时支持<a href="https://aiohttp.readthedocs.io/en/stable/client.html#aiohttp-client">客户端使用</a>和<a href="https://aiohttp.readthedocs.io/en/stable/web.html#aiohttp-web">服务端使用</a>。
* 同时支持<a href="https://aiohttp.readthedocs.io/en/stable/web.html#aiohttp-web-websockets">服务端WebSockets组件</a>和<a href="https://aiohttp.readthedocs.io/en/stable/client.html#aiohttp-client-websockets">客户端WebSockets组件</a>，开箱即用呦。
* web服务器具有<a href="https://aiohttp.readthedocs.io/en/stable/web.html#aiohttp-web-middlewares">中间件</a>，<a href="https://aiohttp.readthedocs.io/en/stable/web.html#aiohttp-web-signals">信号组件</a>和可插拔路由。

## aiohttp库安装:
`$ pip install aiohttp`

你可能还想安装更快的<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-cchardet">cchardet</a>库来代替<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-chardet">chardet</a>进行解码:
`$ pip install cchardet`

对于更快的客户端API DNS解析方案，<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-aiodns">aiodns</a>是个很好的选择，极力推荐:
`$ pip install aiodns`

## 快速开始:
客户端例子:
```
import aiohttp
import asyncio
import async_timeout

async def fetch(session, url):
    with async_timeout.timeout(10):
        async with session.get(url) as response:
            return await response.text()

async def main():
    async with aiohttp.ClientSession() as session:
        html = await fetch(session, 'http://python.org')
        print(html)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

服务端例子:
```
from aiohttp import web

async def handle(request):
    name = request.match_info.get('name', "Anonymous")
    text = "Hello, " + name
    return web.Response(text=text)

app = web.Application()
app.router.add_get('/', handle)
app.router.add_get('/{name}', handle)

web.run_app(app)
```

### 注意:
> 这篇文档的所有例子都是利用 *async/await* 语法来完成的，此语法介绍请看<a href="https://www.python.org/dev/peps/pep-0492">PEP 492</a>，此语法仅Python 3.5+有效。
> 如果你使用的是Python 3.4, 请将await替换成yield from，将async 替换成带有 @corotine装饰器的def. 比如:
> ```
> async def coro(...):
>     ret = await f()
> ```
> 应替换为:
> ```
> @asyncio.coroutine
> def coro(...):
>    ret = yield from f()
> ```

## 服务端指南:
<a href="https://aiohttp.readthedocs.io/en/stable/tutorial.html#aiohttp-tutorial">Polls tutorial</a>

## 源码:

该项目托管在<a href="https://github.com/aio-libs/aiohttp">Github</a>.

如果你发现了一个bug或有一些改善的建议请随时<a href="https://github.com/aio-libs/aiohttp/issues">提交</a>。

该库使用<a href="https://travis-ci.org/aio-libs/aiohttp">Travis</a>进行持续集成。

## 程序依赖:
* Python 3.4.2+

* *chardet*

* *multidict*

* *async_timeout*

* *yarl*

* 可选更快的<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-cchardet">cchardet</a>代替<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-chardet">chardet</a>。

可通过下面命令的安装:

`$ pip install cchardet`

可选<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-aiodns">aiodns</a>进行DNS快速解析。极力推荐。
`$ pip install aiodns`

## 交流渠道:
*aio-libs* 谷歌交流群: https://groups.google.com/forum/#!forum/aio-libs

随时在这里交流你的问题和想法。

*gitter 聊天* https://gitter.im/aio-libs/Lobby

我们还支持<a href="https://stackoverflow.com/questions/tagged/aiohttp">Stack Overflow</a>. 在你的问题上添加aiohttp标签即可。

## 贡献
请在写一个PR前阅读下<a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/Contributing.md">贡献须知</a>。

## 作者和授权
`aiohttp` 大部分由 Nikolay Kim 和 Andrew Svetlov编写.

使用 Apache 2 授权并可随意使用。

随时在<a href="https://github.com/aio-libs/aiohttp">GitHub</a>上提交PR来改善此项目。

## 对后续不再兼容的更改所采用的策略
一般的更改aiohttp 保持向后兼容.

在废弃某些公开API(方法，类，函数参数等.)后仍保证可以使用这些被废弃的API至少一年半的时间直到某新版本完全弃用。

所有废弃的东西都会反映在文档中并给出**已废弃**提示。

有时我们会因一些必须要做的理由而打破某些我们定的规则。大多数原因是有只能通过修改主要API解决的BUG出现，但我们会尽可能不让这种事情发生。

## 目录:
打开此<a href="https://aiohttp.readthedocs.io/en/stable/toc.html#mastertoc">链接</a>看完整的目录。