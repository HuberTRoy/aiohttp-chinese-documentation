# 常见问题汇总

* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#有没有提供像flask一样的approute装饰器的计划">有没有提供像Flask一样的`@app.route`装饰器的计划？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#aiohttp有没有flask中的蓝图或django中的app的概念呢">aiohttp有没有Flask中的蓝图或Django中的App的概念呢？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#如何创建一个可以缓存url的给定前缀的路由">如何创建一个可以缓存url的给定前缀的路由？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#我要把数据库连接放在哪才可以让处理器访问到它">我要把数据库连接放在哪才可以让处理器访问到它？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#为什么最低版本只支持到python-342">为什么最低版本只支持到Python 3.4.2？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#如何让中间件可以存储数据以便让web-handler使用">如何让中间件可以存储数据以便让web-handler使用？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#如何并行地接收来自不同源的事件">如何并行地接收来自不同源的事件？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#如何以编程的方式在服务器端关闭websocket">如何以编程的方式在服务器端关闭websocket？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#如何从特定ip地址上发起请求">如何从特定IP地址上发起请求？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#如果是隐式循环要怎么用aiohttp的测试功能呢">如果是隐式循环要怎么用aiohttp的测试功能呢？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#api创建和废弃政策是什么">API创建和废弃政策是什么？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#如何让整个应用程序都使用gzip压缩">如何让整个应用程序都使用gzip压缩？</a>
* <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/FrequentlyAskedQuestions.md#在web服务器中如何管理clientsession">在web服务器汇总如何管理ClientSession？</a>

# 有没有提供像Flask一样的`@app.route`装饰器的计划？
这种方法有几项问题:
* 最大的问题是“导入时会有副作用”。
* 路由匹配是有序的，这样做的话在导入时很难保证顺序。
* 在大部分大型应用中，都表示在某文件中写路由表比这样好很多。
所以，基于以上原因，我们就没有提供这个功能。不过如果你真的很想用这个功能，继承下`web.Application`然后自己写一个吧~。

# aiohttp有没有Flask中的蓝图或Django中的App的概念呢？
如果你计划写一个大型应用程序，你可以考虑使用嵌套应用。它的功能就像Flask的蓝图和Django的应用一样。
使用嵌套应用你可以为主应用程序添加子应用。

请看: <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/ServerUsage.md#嵌套应用">嵌套应用</a>

# 如何创建一个可以缓存url的给定前缀的路由？
尝试下这样做:
```
app.router.add_route('*', '/path/to/{tail:.+}', sink_handler)
```
第一个参数星号（\*）表示可以是任意方法（GET, POST, OPTIONS等等），第二个参数则是指定的前缀，第三个就是处理器了。

# 我要把数据库连接放在哪才可以让处理器访问到它？
aiohttp.web.Application对象支持字典接口（dict），在这里面你可以存储数据库连接或其他任何你想在不同处理器间共享的资源。请看例子:
```
async def go(request):
    db = request.app['db']
    cursor = await db.cursor()
    await cursor.execute('SELECT 42')
    # ...
    return web.Response(status=200, text='ok')


async def init_app(loop):
    app = Application(loop=loop)
    db = await create_connection(user='user', password='123')
    app['db'] = db
    app.router.add_get('/', go)
    return app
```

# 为什么最低版本只支持到Python 3.4.2？
在aiohttp 还是v0.18.0时，我们是支持Python 3.3 - 3.4.1的。    
最主要的原因是 `object.__del__()`方法了，自Python3.4.1时它才可以完整地工作，这也正是我们想要的——可以很方便的关闭资源。    
当前适用于Python3.3, 3.4.0版本的是v0.17.4。    
当然，这应该对于大多数适用aiohttp的用户来说不是个问题（例子中的Ubuntu 14.04.3 LTS的Python版本是3.4.3呢！），不过依赖于aiohttp的包应该考虑下这个问题了，是只使用低版本aiohttp呢还是一起抛弃Python3.3呢。
在aiohttp v1.0.0时我们就抛弃了Python 3.4.1转而要求3.4.2 +。原因是: loop.is_closed自3.4.2才出现。
最后，在如今的2016年夏这更不应该是个问题了，主流都已经是Python 3.5啦。

# 如何让中间件可以存储数据以便让web-handler使用？
`aiohttp.web.Request`与`aiohttp.web.Application`一样都支持字典接口（dict）。
只需要将数据放到里面即可：
```
async def handler(request):
    request['unique_key'] = data
```
请看 https://github.com/aio-libs/aiohttp_session 的代码，`aiohttp_session.get_session(request)`方法使用`SESSION_KEY`来保存请求的特定会话信息。

# 如何并行地接收来自不同源的事件？
比如我们现在有两个事件：
1. 某一终端用户的WebSocket事件。

2. Redis PubSub从应用的其他地方接受信息并要通过websocket发送给其他用户的事件。
并行地调用`aiohttp.web.WebSocketResponse.receive()`是不行的，同一时间只有一个任务可以执行websocket读操作。
不过其他任务可以使用相同的websocket对象发送数据：

```
async def handler(request):

    ws = web.WebSocketResponse()
    await ws.prepare(request)
    task = request.app.loop.create_task(
        read_subscription(ws,
                          request.app['redis']))
    try:
        async for msg in ws:
            # handle incoming messages
            # use ws.send_str() to send data back
            ...

    finally:
        task.cancel()

async def read_subscription(ws, redis):
    channel, = await redis.subscribe('channel:1')

    try:
        async for msg in channel.iter():
            answer = process message(msg)
            ws.send_str(answer)
    finally:
        await redis.unsubscribe('channel:1')
```

# 如何以编程的方式在服务器端关闭websocket？
比如我们现在有两个终端：
1.  `/echo` 一个用于以某种方式验证用户真实性的回显websocket。
2. `/logout`  用于关闭某一用户的打开的所有websocket连接。
一种简单的解决方法是在`aiohttp.web.Application`实例中为某一用户持续存储websocket响应，并在`/logout_user`处理器中执行`aiohttp.web.WebSocketResponse.close()`。

```
async def echo_handler(request):

    ws = web.WebSocketResponse()
    user_id = authenticate_user(request)
    await ws.prepare(request)
    request.app['websockets'][user_id].add(ws)
    try:
        async for msg in ws:
            ws.send_str(msg.data)
    finally:
        request.app['websockets'][user_id].remove(ws)

    return ws


async def logout_handler(request):

    user_id = authenticate_user(request)

    ws_closers = [ws.close() for ws in request.app['websockets'][user_id] if not ws.closed]

    # Watch out, this will keep us from returing the response until all are closed
    ws_closers and await asyncio.gather(*ws_closers)

    return web.Response(text='OK')


def main():
    loop = asyncio.get_event_loop()
    app = web.Application(loop=loop)
    app.router.add_route('GET', '/echo', echo_handler)
    app.router.add_route('POST', '/logout', logout_handler)
    app['websockets'] = defaultdict(set)
    web.run_app(app, host='localhost', port=8080)
```

# 如何从特定IP地址上发起请求？
如果你的系统上有多个IP接口，你可以选择其中一个来绑定到本地socket:

```
conn = aiohttp.TCPConnector(local_addr=('127.0.0.1', 0), loop=loop)
async with aiohttp.ClientSession(connector=conn) as session:
    ...
```

### 扩展
请看 <a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/ClientReference.md#tcpconnector">`aiohttp.TCPConnector`及其`local_addr`参数</a>。

#  如果是隐式循环要怎么用aiohttp的测试功能呢？
传递显式的loop是推荐方式。但有时如果你有一个嵌套多层而且写的不好的服务时，这几乎是不可能的任务。    
这里推荐一种基于**猴子补丁**的技术方式，要依赖于`aioes`，具体方式是注射一个loop进去。这样你只需要让`AioESService`在其自己的循环中就行（##非常不确定此句的正确性##）。例子如下:

```
import pytest

from unittest.mock import patch, MagicMock

from main import AioESService, create_app

class TestAcceptance:

    async def test_get(self, test_client, loop):
        with patch("main.AioESService", MagicMock(
                side_effect=lambda *args, **kwargs: AioESService(*args,
                                                                 **kwargs,
                                                                 loop=loop))):
            client = await test_client(create_app)
            resp = await client.get("/")
            assert resp.status == 200
```

注意我们为`AioESService`打了补丁，但是要额外传入一个显式`loop`（你需要自己加载loop fixture）。
最后需要测试的代码（你需要一个本地elasticsearch实例来运行）：
```
import asyncio

from aioes import Elasticsearch
from aiohttp import web


class AioESService:

    def __init__(self, loop=None):
        self.es = Elasticsearch(["127.0.0.1:9200"], loop=loop)

    async def get_info(self):
        return await self.es.info()


class MyService:

    def __init__(self):
        self.aioes_service = AioESService()

    async def get_es_info(self):
        return await self.aioes_service.get_info()


async def hello_aioes(request):
    my_service = MyService()
    cluster_info = await my_service.get_es_info()
    return web.Response(text="{}".format(cluster_info))


def create_app(loop=None):

    app = web.Application(loop=loop)
    app.router.add_route('GET', '/', hello_aioes)
    return app


if __name__ == "__main__":
    web.run_app(create_app())
```
全部测试文件：

```
from unittest.mock import patch, MagicMock

from main import AioESService, create_app


class TestAioESService:

    async def test_get_info(self, loop):
        cluster_info = await AioESService("random_arg", loop=loop).get_info()
        assert isinstance(cluster_info, dict)


class TestAcceptance:

    async def test_get(self, test_client, loop):
        with patch("main.AioESService", MagicMock(
                side_effect=lambda *args, **kwargs: AioESService(*args,
                                                                 **kwargs,
                                                                 loop=loop))):
            client = await test_client(create_app)
            resp = await client.get("/")
            assert resp.status == 200
```

注意我们要怎么用side_effect功能来注射一个loop到`AioESService.__init__`中的。 \*args和\*\*kwargs是必须的，是为了将调用时传入的参数都传递出去。

# API创建和废弃政策是什么？
aiohttp尽量不会破坏现存的用户代码。    
过时的属性和方法会在文档中被标记为已废弃，并且在使用时也会抛出`DeprecationWarning`警告。    
废弃周期通常为一年半。     
过了废弃周期后废弃的代码会被删除。    
如果有新功能或bug修复强制我们打破规则我们也会打破的（比如合适地客户端cookies支持让我们打破了向后兼容政策两次！）。     
所有向后不兼容的改变都会在CHANGES章节明确指出。     

# 如何让整个应用程序都使用gzip压缩？
这是不可能的。选择在哪不用压缩和用哪种压缩方式是一个非常棘手的问题。    
如果你需要全局压缩——写一个自定义中间件吧。或在NGINX中开启（:smirk: 你正在部署反向代理是不是）。

# 在web服务器中如何管理ClientSession？
aiohttp.ClientSession在服务器的生命周期中只应该被创建一次，因为可以更好地利用连接池。           
Session在内部保存cookies。如果你不需要cookies使用aiohttp.DummyCookieJar就行。如果你需要在不同的http调用中使用不同的cookies，但在逻辑链的处理时使用的是同一个aiohttp.TCPConnector，那把own_connector设置为False。




