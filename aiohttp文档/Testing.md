# 测试
# aiohttp web服务器测试
aiohttp有一个pytest插件可以轻松构建web服务器测试程序，同时该插件还有一个用于测试其他框架（单元测试等）的测试框架包。      
在写测试之前，我想你可能会想读一读如何写一个可测试的服务器程序感兴趣，因为它们之间的作用的相互的。    
在使用之前，我们还需要安装下才行:
```
$ pip install pytest-aiohttp
```
如果你不想安装它，你可以在conftest.py中插入一行 `pytest_plugins='aiohttp.pytest_plugin'`来代替这个包。

# 临时状态说明
该模块是临时的。

对于已经废弃的API，基于向后不兼容政策，aiohttp允许仍可以继续使用一年半的时间。

不过这对aiohttp.test_tools则不适用。

同时，如有一些必要的原因，我们也会不管向后兼容期而做出更改。

# 客户端与服务器端测试程序

aiohttp中的test_utils有一个基于aiohttp的web服务器测试模板。    
其中包含两个部分: 一个是启动测试服务器，然后是向这个服务器发起HTTP请求。
TestServer使用以aiohttp.web.Application为基础的服务器。RawTestServer则使用基于aiohttp.web.WebServer的低级服务器。
发起HTTP请求到服务器你可以创建一个TestClient实例对象。
测试客户端实例使用aiohttp.ClientSession来对常规操作如ws_connect,get,post等进行支持。

#Pytest
pytest-aiohttp插件允许你创建客户端并向你的应用程序发起请求来进行测试。

简易程序如下:
```
from aiohttp import web

async def hello(request):
    return web.Response(text='Hello, world')

async def test_hello(test_client, loop):
    app = web.Application()
    app.router.add_get('/', hello)
    client = await test_client(app)
    resp = await client.get('/')
    assert resp.status == 200
    text = await resp.text()
    assert 'Hello, world' in text
```
同样，它也提供访问app实例的方法，允许测试组件查看app的状态。使用fixture可以创建非常便捷的app测试客户端:
```
import pytest
from aiohttp import web


async def previous(request):
    if request.method == 'POST':
        request.app['value'] = (await request.post())['value']
        return web.Response(body=b'thanks for the data')
    return web.Response(
        body='value: {}'.format(request.app['value']).encode('utf-8'))

@pytest.fixture
def cli(loop, test_client):
    app = web.Application()
    app.router.add_get('/', previous)
    app.router.add_post('/', previous)
    return loop.run_until_complete(test_client(app))

async def test_set_value(cli):
    resp = await cli.post('/', data={'value': 'foo'})
    assert resp.status == 200
    assert await resp.text() == 'thanks for the data'
    assert cli.server.app['value'] == 'foo'

async def test_get_value(cli):
    cli.server.app['value'] = 'bar'
    resp = await cli.get('/')
    assert resp.status == 200
    assert await resp.text() == 'value: bar'
```
Pytest工具箱里有以下fixture:
aiohttp.test_utils.**test_server**(*app, \*\*kwargs*)
&ensp;&ensp;&ensp; 一个创建TestServer的fixture。
```
async def test_f(test_server):
    app = web.Application()
    # 这里填写路由表

    server = await test_server(app)
```
&ensp;&ensp;&ensp;服务器会在测试功能结束后销毁。   
&ensp;&ensp;&ensp;*app*是aiohttp.web.Application组件，用于启动服务器。    
&ensp;&ensp;&ensp;*kwargs*是其他需要传递的参数。    

aiohttp.test_utils.**test_client**(*app, \*\*kwargs*)
aiohttp.test_utils.**test_client**(*server, \*\*kwargs*)
aiohttp.test_utils.**test_client**(*raw_server, \*\*kwargs*)
&ensp;&ensp;&ensp; 一个用户创建访问测试服务的TestClient fixture。    
```
async def test_f(test_client):
    app = web.Application()
    # 这里填写路由表。

    client = await test_client(app)
    resp = await client.get('/')
```
&ensp;&ensp;&ensp; 客户端和响应在测试功能完成后会自动清除。       
&ensp;&ensp;&ensp; 这个fixture可以接收aiohttp.webApplication, aiohttp.test_utils.TestServer或aiohttp.test_utils.RawTestServer实例对象。   
&ensp;&ensp;&ensp; *kwargs*用于接收传递给aiohttp.test_utils.TestClient的参数。   

aiohttp.test_utils.**raw_test_server**(*handler, \*\*kwargs*)
&ensp;&ensp;&ensp; 一个从给定web处理器实例创建RawTestServer的fixture。   
&ensp;&ensp;&ensp; 处理器应是一个可以接受请求并且返回响应的协同程序:    
```
async def test_f(raw_test_server, test_client):

    async def handler(request):
        return web.Response(text="OK")

    raw_server = await raw_test_server(handler)
    client = await test_client(raw_server)
    resp = await client.get('/')
```

# 单元测试
使用标准库里的单元测试/基础单元测试的功能来测试应用程序，提供AioHTTPTestCase类:
```
from aiohttp.test_utils import AioHTTPTestCase, unittest_run_loop
from aiohttp import web

class MyAppTestCase(AioHTTPTestCase):

    async def get_application(self):
        """
        Override the get_app method to return your application.
        """
        return web.Application()

    # the unittest_run_loop decorator can be used in tandem with
    # the AioHTTPTestCase to simplify running
    # tests that are asynchronous
    @unittest_run_loop
    async def test_example(self):
        request = await self.client.request("GET", "/")
        assert request.status == 200
        text = await request.text()
        assert "Hello, world" in text

    # a vanilla example
    def test_example(self):
        async def test_get_route():
            url = root + "/"
            resp = await self.client.request("GET", url, loop=loop)
            assert resp.status == 200
            text = await resp.text()
            assert "Hello, world" in text

        self.loop.run_until_complete(test_get_route())
```
*class aiohttp.test_utils.**AioHTTPTestCase***     
&ensp;&ensp;&ensp;  一个允许使用aiohttp对web应用程序进行单元测试的基础类。    
&ensp;&ensp;&ensp;  该类派生于unittest.TestCase.    
&ensp;&ensp;&ensp;  提供下列功能:        
&ensp;&ensp;&ensp;  **client**       
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp; aiohttp测试客户端（TestClient实例）。    
&ensp;&ensp;&ensp;  **server**     
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp; aiohttp测试服务器（TestServer实例）。    新增于2.3.0版本。     
&ensp;&ensp;&ensp;  **loop**     
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  应用程序和服务器运行的事件循环。    
&ensp;&ensp;&ensp;  **app**     
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  应用程序（aiohttp.web.Application实例），由get_app()返回。     
&ensp;&ensp;&ensp;  **coroutine get_client()**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  该方法可以被覆盖。返回测试中的TestClient对象。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  **返回TestClient实例对象。**    新增于2.3.0版本。     
&ensp;&ensp;&ensp;  **coroutine get_server()**    
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;   该方法可以备覆盖。返回测试中的TestServer对象。    
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  **返回TestServer实例对象。**    新增于2.3.0版本。      
&ensp;&ensp;&ensp;  **coroutine get_application()**     
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;   该方法可以被覆盖。返回用于测试的aiohttp.web.Application对象。    
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;   **返回aiohttp.web.Application实例对象。**    
&ensp;&ensp;&ensp;  **coroutine setUpAsync()**     
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;   默认该方法什么也不做，不过可以被覆盖用于在TestCase的setUp阶段执行异步代码。     新增于2.3.0版本。     
&ensp;&ensp;&ensp;  **coroutine tearDownAsync()**    
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;   默认该方法什么也不做，不过可以被覆盖用于在TestCase的tearDown阶段执行异步代码。   新增于2.3.0版本。    
&ensp;&ensp;&ensp;  **setUp()**   
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  标准测试初始化方法。    
&ensp;&ensp;&ensp;  **tearDown()**   
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  标准测试析构方法。   

### 注意
    TestClient的方法都是异步方法，你必须使用异步方法来执行它的函数。
    使用unittest_run_loop()装饰器可以包装任何一个基础类中的测试方法。
```
class TestA(AioHTTPTestCase):

    @unittest_run_loop
    async def test_f(self):
        resp = await self.client.get('/')
```
**unittest_run_loop**     
&ensp;&ensp;&ensp; 专门用在AioHTTPTestCase的异步方法上的装饰器。     
&ensp;&ensp;&ensp; 使用AioHTTPTestCase中的AioHTTPTestCase.loop来执行异步函数。
# 虚假请求对象
aiohttp提供创建虚假aiohttp.web.Request对象的测试工具: `aiohttp.test_utils.make_mocked_request()`，在一些简单的单元测试中特别好用，比如处理器测试，或者很难在真实服务器上重现的错误之类的。
```
from aiohttp import web
from aiohttp.test_utils import make_mocked_request

def handler(request):
    assert request.headers.get('token') == 'x'
    return web.Response(body=b'data')

def test_handler():
    req = make_mocked_request('GET', '/', headers={'token': 'x'})
    resp = handler(req)
    assert resp.body == b'data'
```

### 警告
    我们不建议在任何地方都用make_mocked_request()来测试，最好使用真实的操作。
    make_mocked_request()的存在只是为了测试那些很难或根本不能通过简便方法测试的复杂案例（比如模仿网络错误）。
aiohttp.test_utils.**make_mocked_request**(*method, path, headers=None, \*, version=HttpVersion(1, 1), closing=False, app=None, match_info=sentinel, reader=sentinel, writer=sentinel, transport=sentinel, payload=sentinel, sslcontext=None, loop=...*)    
&ensp;&ensp;&ensp; 创建一个用于测试的仿真web.Request。   
&ensp;&ensp;&ensp; 对于那些在特殊环境难以触发的错误在单元测试中非常有用。    
&ensp;&ensp;&ensp; **参数:**    
*  method (str) - str, 代表HTTP方法，如GET, POST。   
*  path(str) - str, 带有URL的路径信息但没有主机名的字符串。   
*  headers(dict, multidict.CIMultiDict, 成对列表) -  一个包含头信息的映射对象。可传入任何能被multidict.CIMultiDict接受的对象。   
*  match_info(dict) - 一个包含url参数信息的映射对象。
*  version(aiohttp.protocol.HttpVersion) - 带有HTTP版本的namedtuple。
*  closing(bool) - 一个用于决定是否在响应后保持连接的标识。  
*  app(aiohttp.web.Application) - 带有虚假请求的aiohttp.web.application。
*  writer - 管理如何输出数据的对象。
*  transport (asyncio.transports.Transport) - asyncio transport 实例。
*  payload (aiohttp.streams.FlowControlStreamReader) - 原始载体读取器对象。
*  sslcontext(ssl.SSLContext) - ssl.SSLContext对象，用于HTTPS连接。
*  loop (asyncio.AbstractEventLoop) - 事件循环对象，默认是仿真（mocked）循环。
&ensp;&ensp;&ensp; **返回**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;返回aiohttp.web.Request对象。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;2.3版本新增: match_info参数。     


# 未知框架工具箱
创建高等级测试:
```
from aiohttp.test_utils import TestClient, loop_context
from aiohttp import request

# loop_context is provided as a utility. You can use any
# asyncio.BaseEventLoop class in it's place.
with loop_context() as loop:
    app = _create_example_app()
    with TestClient(app, loop=loop) as client:

        async def test_get_route():
            nonlocal client
            resp = await client.get("/")
            assert resp.status == 200
            text = await resp.text()
            assert "Hello, world" in text

        loop.run_until_complete(test_get_route())
```

如果需要更细粒度的创建/拆除，可以直接用TestClient对象:
```
from aiohttp.test_utils import TestClient

with loop_context() as loop:
    app = _create_example_app()
    client = TestClient(app, loop=loop)
    loop.run_until_complete(client.start_server())
    root = "http://127.0.0.1:{}".format(port)

    async def test_get_route():
        resp = await client.get("/")
        assert resp.status == 200
        text = await resp.text()
        assert "Hello, world" in text

    loop.run_until_complete(test_get_route())
    loop.run_until_complete(client.close())
```

你可以在api参考中找到所有工具包清单。

# 编写可测试服务
一些如motor, aioes等依赖asyncio循环来执行代码的库，当它们运行正常程序时，都会选一个主事件循环给asyncio.get_event_loop。问题在于，当处在测试环境中时，我们没有主事件循环，因为每个测试都有一个独立的循环。
这样当其他库尝试找这个主事件循环时就会发生出错。不过幸运的是，这问题很好解决，我们可以显式的传入循环。我们来看aioes客户端中的代码:
```
def __init__(self, endpoints, *, loop=None, **kwargs)
```
如你所见，有一个可选的loop参数。当然，我们并不打算直接测试aioes客户端只是我们的服务建立在它之上。所以如果我们想让我们的AioESService容易测试，我们可以这样写:
```
import asyncio

from aioes import Elasticsearch


class AioESService:

    def __init__(self, loop=None):
        self.es = Elasticsearch(["127.0.0.1:9200"], loop=loop)

    async def get_info(self):
        cluster_info = await self.es.info()
        print(cluster_info)


if __name__ == "__main__":
    client = AioESService()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(client.get_info())
```
注意它接受的loop参数。正常情况下没有什么影响因为我们不用显示地传递loop就能让服务有一个主事件循环。问题出在我们测试时:
```
import pytest

from main import AioESService


class TestAioESService:

    async def test_get_info(self):
        cluster_info = await AioESService().get_info()
        assert isinstance(cluster_info, dict)
```

如果尝试运行测试，一般会失败并给出类似下面的信息:
```
...
RuntimeError: There is no current event loop in thread 'MainThread'.
```
因为aioes在主线程中找不到当前的事件循环，所以就报错咯。显式地传递事件循环可以解决这个问题。
如果你的代码依靠隐式循环工作，你可以需要点小技巧。请看FAQ。

# 测试API参考
## 测试服务器
在随机TCP端口上运行给定的aiohttp.web.Application。    
创建完成后服务器并没开始，请用start_server()确保服务器开启和使用close()来确保关闭。    
测试服务器通常与aiohttp.test_utils.TestClient连用，后者可以提供便利的客户端方法来访问服务器。    
class aiohttp.test_utils.**BaseTestServer**(*\*, scheme='http', host='127.0.0.1'*)     
&ensp;&ensp;&ensp; 测试服务器的基础类。    
&ensp;&ensp;&ensp; **参数：**    
* scheme(str) - HTTP协议，默认是无保护的“http”。     
* host(str) - TCP套接字主机，默认是IPv4本地主机（127.0.0.1）。  
&ensp;&ensp;&ensp; **scheme**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 被测试应用使用的协议，'http'是无保护的，'https'是有TLS加密的。    
&ensp;&ensp;&ensp; **host**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于启动测试服务器的主机名。    
&ensp;&ensp;&ensp; **port**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于启动测试服务器的端口（随机的）。    
&ensp;&ensp;&ensp; **handler**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于处理HTTP请求的aiohttp.web.WebServer对象。     
&ensp;&ensp;&ensp; **server**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于管理已接受连接的asyncio.AbstractServer对象。
&ensp;&ensp;&ensp;*coroutine start_server(loop=None, \*\*kwargs)*    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数：**    loop(asyncio.AbstractEventLoop) - 用于开启测试服务器的事件循环。     
&ensp;&ensp;&ensp; *coroutine close()*     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;停止和结束开启的测试服务器。    
&ensp;&ensp;&ensp; *make_url(path)*    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回给定path的绝对URL。

class aiohttp.test_utils.**RawTestServer**(*handler, \*, scheme="http", host="127.0.0.1"*)      
&ensp;&ensp;&ensp;  低级测试服务器（派生于BaseTestServer）     
&ensp;&ensp;&ensp;  **参数：**    
* handler - 用于处理web请求的协同程序。处理器需要接受aiohttp.web.BaseRequest实例并且返回响应实例（StreamResponse或Response之类的）。对于非200的HTTP响应，处理器可以抛出HTTPException异常。     
* scheme(str) - HTTP协议，默认是无保护的“http”。
* host(str) -  TCP套接字主机，默认是IPv4本地主机（127.0.0.1）。     
class aiohttp.test_utils.**TestServer**(*app, \*, scheme="http", host="127.0.0.1"*)    
&ensp;&ensp;&ensp;  用于启动应用程序的测试服务器（派生于BaseTestServer）。    
&ensp;&ensp;&ensp; **参数：**    
* app - 要启动的aiohttp.web.Application实例对象。
* scheme(str) - HTTP协议，默认是无保护的“http”。
* host(str) - TCP套接字主机，默认是IPv4本地主机（127.0.0.1）。
&ensp;&ensp;&ensp; app    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 要启动的aiohttp.web.Application实例对象。

# 测试客户端。
class aiohttp.test_utils.**TestClient**(*app_or_server, \*, loop=None, scheme='http', host='127.0.0.1'*)    
&ensp;&ensp;&ensp; 一个用于制造请求来测试服务器的测试客户端。    
&ensp;&ensp;&ensp; **参数：**    
* app_or_server - BaseTestServer实例对象，用于向其发起请求。如果是aiohttp.web.Application对象，会为应用程序自动创建一个TestServer。  
* cookie_jar - 可选的aiohttp.CookieJar实例对象，搭配CookieJar(unsafe=True)更佳。   
* scheme (str) - HTTP协议，默认是无保护的“http”。
* loop (asyncio.AbstractEventLoop) - 需要使用的事件循环。
* host (str) - TCP套接字主机，默认是IPv4本地主机（127.0.0.1）。
&ensp;&ensp;&ensp; scheme    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 被测试应用的使用的协议，'http'是无保护的，'https'是有TLS加密的。      
&ensp;&ensp;&ensp; host    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于启动测试服务器的主机名。   
&ensp;&ensp;&ensp; port     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于启动测试服务器的端口（随机的）。   
&ensp;&ensp;&ensp; server    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; BaseTestServer测试服务器实例，一般与客户端连用。    
&ensp;&ensp;&ensp; session    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 内部aiohttp.ClientSession对象.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 不同于TestClient中的那样，客户端会话的请求不会自动将url查询放入主机名中，需要传入一个绝对路径。     
&ensp;&ensp;&ensp; coroutine start_server(\*\*kwargs)    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 开启测试服务器。  
&ensp;&ensp;&ensp; coroutine close()         
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 关闭在运行的服务器。    
&ensp;&ensp;&ensp; make_url(path)     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回给定path的绝对URL。     
&ensp;&ensp;&ensp;coroutine request(method, path, \*args, \*\*kwargs)    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 将请求发送给测试服务器。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 除了loop参数被测试服务器所使用的循环覆盖外，该接口与asyncio.ClientSession.request()相同。    
&ensp;&ensp;&ensp; coroutine get(path, \*args, \*\*kwargs)      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 执行HTTP GET请求。    
&ensp;&ensp;&ensp; coroutine post(path, \*args, \*\*kwargs)     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 执行HTTP POST请求。     
&ensp;&ensp;&ensp; coroutine options(path, \*args, \*\*kwargs)     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 执行HTTP OPTIONS请求。      
&ensp;&ensp;&ensp; coroutine head(path, \*args, \*\*kwargs)     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 执行HTTP HEAD请求。     
&ensp;&ensp;&ensp; coroutine put(path, \*args, \*\*kwargs)     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 执行HTTP PUT请求。      
&ensp;&ensp;&ensp; coroutine patch(path, \*args, \*\*kwargs)      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 执行HTTP PATCH请求。      
&ensp;&ensp;&ensp; coroutine delete(path, \*args, \*\*kwargs)      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 执行HTTP DELETE请求。    
&ensp;&ensp;&ensp; coroutine ws_connect(path, \*args, \*\*kwargs)      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 初始化websocket连接。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  该api与aiohttp.ClientSession.ws_connect()相同。     

# 其他工具包
aiohttp.test_utils.make_mocked_coro(return_value)    
&ensp;&ensp;&ensp; 创建一个协程mock。    
&ensp;&ensp;&ensp; 其表现形式像一个协程一般，作用是返回要返回的值（return_value）。而同时又是一个mock对象，你可以使用一般的Mock来测试它:
```
mocked = make_mocked_coro(1)
assert 1 == await mocked(1, 2)
mocked.assert_called_with(1, 2)
```
&ensp;&ensp;&ensp; **参数：**  return_value - 当mock对象被调用时返回的值。     
&ensp;&ensp;&ensp; **像协程一样返回return_value的值。**    

aiohttp.test_utils.unused_port()    
&ensp;&ensp;&ensp; 返回一个可以用在IPv4 TCP协议上的还没有被使用的端口。
&ensp;&ensp;&ensp; 返回一个可以使用端口值（类型为整数int）。      

aiohttp.test_utils.loop_context(loop_factory=<function asyncio.new_event_loop>)    
&ensp;&ensp;&ensp; 一个上下文管理器，可以创建一个用于测试目的事件循环。     
&ensp;&ensp;&ensp; 用于进行测试循环的创建和清理工作。   

aiohttp.test_utils.setup_test_loop(loop_factory=<function asyncio.new_event_loop>)    
&ensp;&ensp;&ensp;  创建并返回asyncio.AbstractEventLoop实例对象。    
&ensp;&ensp;&ensp;  如果要调用它，需要在结束循环时调用下teardown_test_loop.     

aiohttp.test_utils.teardown_test_loop(loop)    
&ensp;&ensp;&ensp;  销毁并清除setup_test_loop所创建的event_loop。    
&ensp;&ensp;&ensp; **参数：**   loop(asyncio.AbstractEventLoop) - 需要拆除的循环。















