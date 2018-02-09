# 底层服务器
这一节介绍了aiohttp.web的基础底层服务器。

# 抽象基础
有时候用户不需要更高级的封装，像是 application，routers和signals。
只是需要一个支持异步调用并且是接受请求返回响应对象的东西。
在aiohttp.web.Server类中有介绍过一个服务协议工厂——`asyncio.AbstractEventLoop.create_server()`，并可以将数据流桥接到web处理器以及反馈结果。
底层web处理器应该接收单个`BaseRequest`参数并且执行下列中的其中一个:
1. 返回一个包含HTTP响应体的响应对象。
2. 创建一个`StreamResponse`对象，然后可以调用`StreamResponse.prepare()`发送头信息，调用`StreamResponse.write() / StreamResponse.drain()`发送数据块，最后结束响应。
3. 抛出`HTTPException`派生的异常（看<a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/ServerUsage.md#异常">Exception</a>部分）。
4. 使用`WebSocketResponse`发起/处理Web-Socket连接。

# 运行基础底层服务器

请看下列代码:
```
import asyncio
from aiohttp import web


async def handler(request):
    return web.Response(text="OK")


async def main(loop):
    server = web.Server(handler)
    await loop.create_server(server, "127.0.0.1", 8080)
    print("======= Serving on http://127.0.0.1:8080/ ======")

    # pause here for very long time by serving HTTP requests and
    # waiting for keyboard interruption
    await asyncio.sleep(100*3600)


loop = asyncio.get_event_loop()

try:
    loop.run_until_complete(main(loop))
except KeyboardInterrupt:
    pass
loop.close()
```

这样我们有了一个返回"OK"标准响应的处理器。

这个处理器经由服务器调用。调用`loop.create_server`创建的网络交流通道，随后可以访问http://127.0.0.1:8080/来查看。
这个处理器可以接受所有的请求: 不论`GET, POST, Web-Socket`都可以，无论哪一个路径的访问也都同样由其处理。
不过也同样很基础: 无论如何处理器都只返回`200 OK`。实际生活中所产生的状态要复杂的多。

