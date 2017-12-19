# 服务端使用

# 启动一个简单的Web服务器

部署web服务器首先要创建一个 请求处理器（request handler）。
请求处理器可以是协程方法也可以是普通方法，它只有一个用于接受Request实例对象的参数，之后返回Response实例对象:

```
from aiohttp import web

async def hello(request):
    return web.Response(text="Hello, world")
```

之后我们需要创建应用（Appliaction）并将请求处理器配置到应用的路由，注意选择合适的HTTP方法和请求路径:

```
app = web.Application()
app.router.add_get('/', hello)
```

我们调用run_app()来启动应用:
```
web.run_app(app)
```

最后，我们访问 http://localhost:8080/ 来查看内容。

## 扩展
可查看 优雅的关闭（Graceful shutdown）一节了解 run_app()做了什么以及如何从头开始实现复杂服务器的初始化/关闭。

# 命令行接口（CLI）
aiohttp.web 带有一个基于TCP/IP的基本命令行接口，用于快速启动一个应用(Application)。

```
$ python -m aiohttp.web -H localhost -P 8080 package.module:init_func
```

package.module:init_func 应是一个可导入，调用的对象，有一个接受包含命令行参数列表的参数，配置好之后返回Application对象:
```
def init_func(argv):
    app = web.Application()
    app.router.add_get("/", index_handler)
    return app
```

# 处理器
处理器对象可以被任意调用，它只接受一个Request实例对象，并且返回 StreamResponse的派生对象实例（如Response）:
```
def handler(request):
    return web.Response()
```
它还可以是协程方法，这样aiohttp.web会等待处理:
```
async def handler(request):
    return web.Response()
```

处理器必须被注册在路由上才能使用:
```
app.router.add_get('/', handler)
app.router.add_post('/post', post_handler)
app.router.add_put('/put', put_handler)
```

add_route()同样支持使用通配符方法:
```
app.router.add_route('*', '/path', all_handler)
```
之后可以使用Request.method来获知请求所使用的HTTP方法。

默认情况下，add_get()也会接受HEAD请求，并像使用GET请求一样返回响应头。你可以禁用它:
```
app.router.add_get('/', handler, allow_head=False)
```
如果处理器不能被调用，服务器将会返回405。
### 注意
根据RFC 7231 aiohttp 2.0版本后做了接受HEAD请求的调整，之前的版本使用add_get()添加的请求，如果使用HEAD方法总会返回405。

如果处理器会写入很多响应体内容，你可以在执行HEAD方法时跳过处理响应体内容以提高执行效率。

# 资源和路由

内部路由是一个资源列表。
资源是路由表的入口，相当于所请求的URL。
每个资源至少有一个路由。
路由负责调用web处理器进行处理。

UrlDispatcher.add_get() / UrlDispatcher.add_post() 及其同类方法是 UrlDispatcher.add_route()的一种简便写法。
同样 UrlDispatcher.add_route() 是 UrlDispatcher.add_resource() and Resource.add_route() 的一种简便写法。

```
resource = app.router.add_resource(path, name=name)
route = resource.add_route(method, handler)
return route
```

扩展:
查看 Router refactoring in 0.21 获取更多信息。

# 可变形资源
资源可以是可变的路径。比如，如果某一路径是 '/a/{name}/c'，那'/a/b/c', '/a/1/c'，'/a/etc/c'，这样的路径就都可以匹配到。
可变部分需要用 {标识字符} 这样的形式，标识字符的作用是让处理器可以匹配到这个值。使用Request.match_info来完成这一操作:
```
async def variable_handler(request):
    return web.Response(
        text="Hello, {}".format(request.match_info['name']))

resource = app.router.add_resource('/{name}')
resource.add_route('GET', variable_handler)
```

默认情况下，每个可变部分所使用的正则表达式是 [^{}/]+。

当然你也可以自定义正则表达式 {标识符: 正则}:
```
resource = app.router.add_resource(r'/{name:\d+}')
```

注意:
    正则表达式匹配的是非百分号编码的URL（request.raw_path）字符。比如 空格编码后是%20。
    RFC 3986规定可在路径中的字符是:

```
allowed       = unreserved / pct-encoded / sub-delims
                        / ":" / "@" / "/"

pct-encoded   = "%" HEXDIG HEXDIG

unreserved    = ALPHA / DIGIT / "-" / "." / "_" / "~"

sub-delims    = "!" / "$" / "&" / "'" / "(" / ")"
              / "*" / "+" / "," / ";" / "="
```    

# 使用命名的资源反向引用URL。

路由中可以指定name:

```
resource = app.router.add_resource('/root', name='root')
```
之后可以访问和构建此资源的URL。
```
>>> request.app.router['root'].url_for().with_query({"a": "b", "c": "d"})
URL('/root?a=b&c=d')
```

带有可变形部分的资源:
```
app.router.add_resource(r'/{user}/info', name='user-info')
```
这时传入那部分即可:
```
>>> request.app.router['user-info'].url_for(user='john_doe')\
...                                         .with_query("a=b")
'/john_doe/info?a=b'
```

# 将处理器放到类中使用

如上所述，处理器可以是全局函数或协同程序:
```
async def hello(request):
    return web.Response(text="Hello, world")
app.router.add_get('/', hello)
```
但可以使用Python中的类将一同一类聚合在一起。
aiohttp.web并未提供任何该方面的例子，开发者可任意使用:
```
class Handler:

    def __init__(self):
        pass

    def handle_intro(self, request):
        return web.Response(text="Hello, world")

    async def handle_greeting(self, request):
        name = request.match_info.get('name', "Anonymous")
        txt = "Hello, {}".format(name)
        return web.Response(text=txt)

handler = Handler()
app.router.add_get('/intro', handler.handle_intro)
app.router.add_get('/greet/{name}', handler.handle_greeting)
```

# 基础视图类

aiohttp.web提供 django风格的基础试图类。

你可以从 View 类中继承，并自定义http请求的处理方法:

```
class MyView(web.View):
    async def get(self):
        return await get_resp(self.request)

    async def post(self):
        return await post_resp(self.request)
```

处理器应是一个协程方法，且只接受self参数，并且返回标准web处理器的响应对象。请求对象可使用View.request中取出。

当然还需要在路由中注册:
```
app.router.add_route('*', '/path/to', MyView)
```
这样/path/to既可使用GET也可使用POST进行请求，不过其他未指定的HTTP方法会抛出405错误。


# 资源视图
所有在路由中注册的资源科使用UrlDispatcher.resources()查看:
```
for resource in app.router.resources():
    print(resource)
```

同样，有name的资源可以用UrlDispatcher.named_resources()来查看:
```
for name, resource in app.router.named_resources().items():
    print(name, resource)
```
UrlDispatcher.routes() 新增于 0.18.
UrlDispatcher.named_routes() 新增于 0.19.

0.21版本后请使用 UrlDispatcher.named_routes() / UrlDispatcher.routes() 来代替 UrlDispatcher.named_resources() / UrlDispatcher.resources() 。

# 其他注册路由的方式
上述例子使用常规方式添加新路由: app.router.add_get(...) 之类.

还提供两种其他方法: 路由表和路由装饰器。

路由表和Django中一样:
```
async def handle_get(request):
    ...


async def handle_post(request):
    ...

app.router.add_routes([web.get('/get', handle_get),
                       web.post('/post', handle_post),
```
使用 aiohttp.web.get() 或 aiohttp.web.post()定义处理器，并放到一个列表中然后传给add_routes。
### 扩展:
RouteDef reference.

路由装饰器有点像Flask风格:
```
routes = web.RouteTableDef()

@routes.get('/get')
async def handle_get(request):
    ...


@routes.post('/post')
async def handle_post(request):
    ...

app.router.add_routes(routes)
```
首先是要创建一个 aiohttp.web.RouteTableDef 对象。

该对象是一个类列表对象，额外提供aiohttp.web.RouteTableDef.get()，aiohttp.web.RouteTableDef.post()这些装饰器来注册路由。

最后调用 add_routes()添加到应用的路由里。

### 扩展:
RouteTableDef reference.

这三种方法都是一样的，你可以自行选择喜欢的方式或者混着用也行。

新增于2.3版本。

# Web 处理器中的取消操作
### 警告:
    web处理器中每一条await语句都可被取消，这种情况一般发生在客户端还没有完全读取响应体然后中断了连接。
    与著名WSGI架构 如Flask和Django在这点上有所不同。

在处理GET请求时，代码可能会从数据库或其他web资源中获取数据，这些查询可能很慢。
这时候取消查询是最好的: 该连接已经被抛弃了，没有理由再浪费时间和资源(内存等)进行查询，已经没有机会响应了。

不过在POST请求时可能会有些不好，POST请求常常需要将信息存进数据库，不管该连接是否被抛弃了。

预防被取消掉可以用下列几种方法:

* 使用 asyncio.shield() 来进行存进数据库的处理。
* 开启一个存入数据库的协程任务。
* 使用aiojobs或其他第三方库。

asyncio.shield() 挺不错的，唯一的缺点是你需要分清哪些是需要得到保护的代码哪些不是。

比如这样的代码就不靠谱:
```
async def handler(request):
    await asyncio.shield(write_to_redis(request))
    await asyncio.shield(write_to_postgres(request))
    return web.Response('OK')
```
如果保存到redis时触发了取消，那之后的write_to_postgres将不会被执行。

生成一个新的任务更不靠谱，这样不能等待:
```
async def handler(request):
    request.loop.create_task(write_to_redis(request))
    return web.Response('OK')
```
write_to_redis所产生的错误并不能被捕获，这样会导致有很多asyncio的Future异常的日志消息并且是不可恢复的，因为Task被销毁而不是挂起了。

此外，在按照优雅的关闭（Graceful shutdown）中的进行关闭操作时，aiohttp并不会等待这些任务完成，所以你还要找个机会来释放这些重要的数据。

另一方面，aiojobs提供一个生成新任务并且能等待结果(等此类操作)的API。aiojobs会将所有待完成的操作存入内部数据结构中，并且可以优雅的终止这些操作。
```
from aiojobs.aiohttp import setup, spawn

async def coro(timeout):
    await asyncio.sleep(timeout)  # do something in background

async def handler(request):
    await spawn(request, coro())
    return web.Response()

app = web.Application()
setup(app)
app.router.add_get('/', handler)
```

所有未完成的任务会被终止并由aiohttp.web.Application.on_cleanup 信号发送出去。

使用@atomic装饰器可以使整个处理器都防止有取消操作:

```
from aiojobs.aiohttp import atomic

@atomic
async def handler(request):
    await write_to_db()
    return web.Response()

app = web.Application()
setup(app)
app.router.add_post('/', handler)
```
这样避免了整个async处理器函数执行时被取消，write_to_db也不会被中断。

# 自定义路由准则

有时，比起简单的使用HTTP方法和路径注册路由，你可能会有其他更复杂些的规则。

尽管UrlDispatcher不直接提供额外的准则，但你可以这样做:
```
class AcceptChooser:

    def __init__(self):
        self._accepts = {}

    async def do_route(self, request):
        for accept in request.headers.getall('ACCEPT', []):
            acceptor = self._accepts.get(accept)
            if acceptor is not None:
                return (await acceptor(request))
        raise HTTPNotAcceptable()

    def reg_acceptor(self, accept, handler):
        self._accepts[accept] = handler


async def handle_json(request):
    # do json handling

async def handle_xml(request):
    # do xml handling

chooser = AcceptChooser()
app.router.add_get('/', chooser.do_route)

chooser.reg_acceptor('application/json', handle_json)
chooser.reg_acceptor('application/xml', handle_xml)
```

# 静态文件的处理
处理静态文件( 图面，JavaScripts, CSS文件等)最好的方法是使用反向代理，像是nginx或CDN服务。

但就开发来说，aiohttp服务器本身可以很方便的处理静态文件。

只需要通过 UrlDispatcher.add_static()注册个新的静态路由即可:

```
app.router.add_static('/prefix', path_to_static_folder)
```

当访问静态文件的目录时，默认服务器会返回 HTTP/403 Forbidden(禁止访问)。 使用show_index并将其设置为True可以显示出索引:
```
app.router.add_static('/prefix', path_to_static_folder, show_index=True)
```

当从静态文件目录访问一个符号链接（软链接）时，默认服务器会响应 HTTP/404 Not Found(未找到)。使用follow_symlinks并将其设置为True可以让服务器使用符号链接:
```
app.router.add_static('/prefix', path_to_static_folder, follow_symlinks=True)
```

如果你想允许缓存清除，使用append_version并设为True。

缓存清除会对资源文件像JavaScript 和 CSS文件等的文件名上添加一个hash后的版本。这样的好处是我们可以让浏览器无限期缓存这些文件而不用担心这些文件是否发布了新版本。
```
app.router.add_static('/prefix', path_to_static_folder, append_version=True)
```


# 模板

aiohttp.web并不直接提供模板读取，不过可以使用第三方库 aiohttp_jinja2，是由aiohttp作者维护的。
使用起来也很简单。首先我们用 aiohttp_jinja2.setup()来设置下jinja2环境:
```
app = web.Application()
aiohttp_jinja2.setup(app,
    loader=jinja2.FileSystemLoader('/path/to/templates/folder'))
```

然后将模板引擎应用到处理器中。最简单的方式是使用aiohttp_jinja2.templates()装饰器:
```
@aiohttp_jinja2.template('tmpl.jinja2')
def handler(request):
    return {'name': 'Andrew', 'surname': 'Svetlov'}
```
如果你更喜欢Mako模板引擎，可以看看 aiohttp_mako库。

# JSON 响应

使用 aiohttp.web.json_response()可以返回JSON响应:
```
def handler(request):
    data = {'some': 'data'}
    return web.json_response(data)
```

这个方法返回的是 aiohttp.web.Response实例对象，所以你可以做些其他的事，比如设置 cookies。

# 用户会话
你经常想要一个可以通过请求存储用户数据的仓库。一般简称为会话。

aiohttp.web没有内置会话，不过你可以使用第三方库 aiohttp_session来提供会话支持:
```
import asyncio
import time
import base64
from cryptography import fernet
from aiohttp import web
from aiohttp_session import setup, get_session, session_middleware
from aiohttp_session.cookie_storage import EncryptedCookieStorage

async def handler(request):
    session = await get_session(request)
    last_visit = session['last_visit'] if 'last_visit' in session else None
    text = 'Last visited: {}'.format(last_visit)
    return web.Response(text=text)

def make_app():
    app = web.Application()
    # secret_key must be 32 url-safe base64-encoded bytes
    fernet_key = fernet.Fernet.generate_key()
    secret_key = base64.urlsafe_b64decode(fernet_key)
    setup(app, EncryptedCookieStorage(secret_key))
    app.router.add_route('GET', '/', handler)
    return app

web.run_app(make_app())
```


# HTTP表单
aiohttp直接提供HTTP表单支持。

如果表单的方法是 “GET”（<form method="get">），使用Request.query获取数据。
如果是“POST”则用Request.post()或 Request.multipart()
Request.post()接受标明为'application/x-www-form-urlencoded'和'multipart/form-data' 的数据（<form enctype="multipart/form-data">）。它会将数据存进一个临时字典中。如果指定了client_max_size，超出了的话会抛出ValueError异常。用Request.multipart()是更好的选择，尤其是在上传大文件时。

小例子:
由以下表单发送的数据:
```
<form action="/login" method="post" accept-charset="utf-8"
      enctype="application/x-www-form-urlencoded">

    <label for="login">Login</label>
    <input id="login" name="login" type="text" value="" autofocus/>
    <label for="password">Password</label>
    <input id="password" name="password" type="password" value=""/>

    <input type="submit" value="login"/>
</form>
```

可以用以下方法获取:
```
async def do_login(request):
    data = await request.post()
    login = data['login']
    password = data['password']
```

# 文件上传

aiohttp.web内置处理文件上传的功能。

首先呢我们要确保 <form>标签的有enctype元素并且设置为了"multipart/form-data"。 
我们写一个可以上传MP3文件的表单（作为例子）:
```
<form action="/store/mp3" method="post" accept-charset="utf-8"
      enctype="multipart/form-data">

    <label for="mp3">Mp3</label>
    <input id="mp3" name="mp3" type="file" value=""/>

    <input type="submit" value="submit"/>
</form>
```

之后你可以在请求处理器中接受这个文件，它变成了一个FileField实例对象。FileFiled包含了该文件的原信息。

```
async def store_mp3_handler(request):

    # 注意，如果是个很大的文件不要用这种方法。
    data = await request.post()

    mp3 = data['mp3']

    # .filename 包含该文件的名称，是个字符串。
    filename = mp3.filename

    # .file 包含该文件的内容。
    mp3_file = data['mp3'].file

    content = mp3_file.read()

    return web.Response(body=content,
                        headers=MultiDict(
                            {'CONTENT-DISPOSITION': mp3_file}))
```

注意例子中的警告。通常Reuest.post()会把所有数据读到内容，可能会引起OOM(out of memory 内存炸了)错误。你可以用Request.multipart()来避免这种情况，它返回的是multipart读取器。

```
async def store_mp3_handler(request):

    reader = await request.multipart()

    # /!\ 不要忘了这步。（至于为什么请搜索 Python 生成器/异步）/!\

    mp3 = await reader.next()

    filename = mp3.filename

    # 如果是分块传输的，别用Content-Length做判断。
    size = 0
    with open(os.path.join('/spool/yarrr-media/mp3/', filename), 'wb') as f:
        while True:
            chunk = await mp3.read_chunk()  # 默认是8192个字节。
            if not chunk:
                break
            size += len(chunk)
            f.write(chunk)

    return web.Response(text='{} sized of {} successfully stored'
                             ''.format(filename, size))
```

# WebSockets

aiohttp.web 直接提供WebSockets支持。

在处理器中创建一个WebSocketResponse对象即可设置WebSocket，之后即可进行通信:
```
async def websocket_handler(request):

    ws = web.WebSocketResponse()
    await ws.prepare(request)

    async for msg in ws:
        if msg.type == aiohttp.WSMsgType.TEXT:
            if msg.data == 'close':
                await ws.close()
            else:
                await ws.send_str(msg.data + '/answer')
        elif msg.type == aiohttp.WSMsgType.ERROR:
            print('ws connection closed with exception %s' %
                  ws.exception())

    print('websocket connection closed')

    return ws
```

该处理器需要用HTTP GET方法注册:
```
app.router.add_get('/ws', websocket_handler)

```

从WebSocket中读取数据（await ws.receive()）必须在请求处理器内部完成，不过写数据（ws.send_str(...)），关闭（await ws.close()）和取消操作可以在其他任务中完成。 详情看 FAQ 部分。

aiohttp.web 隐式的使用asyncio.Task处理每个请求。

### 注意:
虽然aiohttp仅支持不带诸如长轮询的WebSocket不过如果我们有维护一个基于aiohttp的SockJS包，用于部署兼容SockJS服务器端的代码。

### 警告
不要试图从websocket中并行地读取数据，aiohttp.web.WebSocketResponse.receive()不能在分布在两个任务中同时调用。

请看 FAQ 部分了解解决方法。

# 异常
aiohttp.web定义了所有HTTP状态码的异常。

每个异常都是HTTPException的子类和某个HTTP状态码。
同样还都是Response的子类，所以就允许你在请求处理器中返回或抛出它们。
请看下面这些代码:

```
async def handler(request):
    return aiohttp.web.HTTPFound('/redirect')
```
```
async def handler(request):
    raise aiohttp.web.HTTPFound('/redirect')
```

每个异常的状态码是根据RFC 2068规定来确定的: 100-300不是由错误引起的; 400之后是客户端错误，500之后是服务器端错误。

异常等级图:
```
  HTTPException
    HTTPSuccessful
      * 200 - HTTPOk
      * 201 - HTTPCreated
      * 202 - HTTPAccepted
      * 203 - HTTPNonAuthoritativeInformation
      * 204 - HTTPNoContent
      * 205 - HTTPResetContent
      * 206 - HTTPPartialContent
    HTTPRedirection
      * 300 - HTTPMultipleChoices
      * 301 - HTTPMovedPermanently
      * 302 - HTTPFound
      * 303 - HTTPSeeOther
      * 304 - HTTPNotModified
      * 305 - HTTPUseProxy
      * 307 - HTTPTemporaryRedirect
      * 308 - HTTPPermanentRedirect
    HTTPError
      HTTPClientError
        * 400 - HTTPBadRequest
        * 401 - HTTPUnauthorized
        * 402 - HTTPPaymentRequired
        * 403 - HTTPForbidden
        * 404 - HTTPNotFound
        * 405 - HTTPMethodNotAllowed
        * 406 - HTTPNotAcceptable
        * 407 - HTTPProxyAuthenticationRequired
        * 408 - HTTPRequestTimeout
        * 409 - HTTPConflict
        * 410 - HTTPGone
        * 411 - HTTPLengthRequired
        * 412 - HTTPPreconditionFailed
        * 413 - HTTPRequestEntityTooLarge
        * 414 - HTTPRequestURITooLong
        * 415 - HTTPUnsupportedMediaType
        * 416 - HTTPRequestRangeNotSatisfiable
        * 417 - HTTPExpectationFailed
        * 421 - HTTPMisdirectedRequest
        * 422 - HTTPUnprocessableEntity
        * 424 - HTTPFailedDependency
        * 426 - HTTPUpgradeRequired
        * 428 - HTTPPreconditionRequired
        * 429 - HTTPTooManyRequests
        * 431 - HTTPRequestHeaderFieldsTooLarge
        * 451 - HTTPUnavailableForLegalReasons
      HTTPServerError
        * 500 - HTTPInternalServerError
        * 501 - HTTPNotImplemented
        * 502 - HTTPBadGateway
        * 503 - HTTPServiceUnavailable
        * 504 - HTTPGatewayTimeout
        * 505 - HTTPVersionNotSupported
        * 506 - HTTPVariantAlsoNegotiates
        * 507 - HTTPInsufficientStorage
        * 510 - HTTPNotExtended
        * 511 - HTTPNetworkAuthenticationRequired
```

所有的异常都拥有相同的结构:
```
HTTPNotFound(*, headers=None, reason=None,
             body=None, text=None, content_type=None)
```
如果没有指定headers，默认是响应中的headers。
其中HTTPMultipleChoices, HTTPMovedPermanently, HTTPFound, HTTPSeeOther, HTTPUseProxy, HTTPTemporaryRedirect的结构是下面这样的:
```
HTTPFound(location, *, headers=None, reason=None,
          body=None, text=None, content_type=None)
```
location参数的值会写入到HTTP头部的Location中。

HTTPMethodNotAllowed 的结构是这样的:
```
HTTPMethodNotAllowed(method, allowed_methods, *,
                     headers=None, reason=None,
                     body=None, text=None, content_type=None)
```
method是不支持的那个方法，allowed_methods是所支持的方法。

# 数据共享

aiohttp.web不推荐使用全局变量进行数据共享。每个变量应在自己的上下文中而不是全局可用的。
因此，aiohttp.web.Application和aiohttp.web.Request提供collections.abc.MutableMapping(类字典对象)来存储数据。

将类全局变量存储到Application实例对象中:
```
app['my_private_key'] = data
```

之后就可以在web处理器中获取出来:
```
async def handler(request):
    data = request.app['my_private_key']
```
如果变量的生命周期是一次请求，可以在请求中存储。
```
async def handler(request):
  request['my_private_key'] = "data"
  ...
```
对于中间件和信号处理器存储需要进一步被处理的数据特别有用。
为了避免与其他aiohttp所写的程序和其他第三方库中的命名出现冲突，请务必写一个独一无二的键名。
如果你要发布到PyPI上，使用你公司的名称或url或许是个不错的选择。(如 org.company.app)

# 中间件

aiohttp.web提供一个强有力的中间件组件来编写自定义请求处理器。

中间件是一个协程程序帮助修正请求和响应。下面这个例子是在响应中添加'wink'字符串:
```
from aiohttp.web import middleware

@middleware
async def middleware(request, handler):
    resp = await handler(request)
    resp.text = resp.text + ' wink'
    return resp
```
（对于流式响应和websockets该例子不起作用）

每个中间件需要接受两个参数，一个是请求实例另一个是处理器，中间件需要返回响应内容。

创建Application时，可以通过middlewares参数传递中间件过去:
```
app = web.Application(middlewares=[middleware_1,
                                   middleware_2])
```

内部会将单个请求处理器处理的结果经由所传入的中间件处理器倒序的处理一遍。
因为middlewares（中间件）都是协程程序，所以它们可以执行await语句以进行如从数据库中查询等操作。
middlewares（中间件）常常会调用处理器，不过也可以不调用，比如用户访问了没有权限访问的资源则显示403Forbidden页面或者抛出HTTPForbidden异常。
处理器中也可能抛出异常，比如执行一些预处理或后续处理（HTTP访问资源控制等）。

下列代码演示中间件的执行顺序:
```
from aiohttp import web

def test(request):
    print('Handler function called')
    return web.Response(text="Hello")

@web.middleware
async def middleware1(request, handler):
    print('Middleware 1 called')
    response = await handler(request)
    print('Middleware 1 finished')
    return response

@web.middleware
async def middleware2(request, handler):
    print('Middleware 2 called')
    response = await handler(request)
    print('Middleware 2 finished')
    return response


app = web.Application(middlewares=[middleware1, middleware2])
app.router.add_get('/', test)
web.run_app(app)
Produced output:

Middleware 1 called
Middleware 2 called
Handler function called
Middleware 2 finished
Middleware 1 finished
```

# 例子
通常中间件用于部署自定义错误页面。下面的例子将会使用JSON响应（JSON REST）显示404错误页面。
```
import json
from aiohttp import web

def json_error(message):
    return web.Response(
        body=json.dumps({'error': message}).encode('utf-8'),
        content_type='application/json')

@web.middleware
async def error_middleware(request, handler):
    try:
        response = await handler(request)
        if response.status == 404:
            return json_error(response.message)
        return response
    except web.HTTPException as ex:
        if ex.status == 404:
            return json_error(ex.reason)
        raise

app = web.Application(middlewares=[error_middleware])
```

# 旧式中间件
2.3之前的版本中间件需要一个返回中间件协同程序的外部中间件处理工厂。2.3之后就不在需要这样做了，使用@middleware装饰器即可。
旧式中间件（使用外部处理工厂不使用@middleware装饰器）仍然支持。此外，新旧两个版本可以混用。
中间件工厂是一个用于在调用中间件之前做些其他操作的协同程序，下面例子是一个简单中间件工厂:
```
async def middleware_factory(app, handler):
    async def middleware_handler(request):
        resp = await handler(request)
        resp.text = resp.text + ' wink'
        return resp
    return middleware_handler
```
中间件工厂要接受两个参数: app实例对象和请求处理器最后返回一个新的处理器。

### 注意:
外部中间件工厂和内部中间件处理器会处理每一个请求。
中间件工厂要返回一个与请求处理器有同样功能的新处理器——接受Request实例对象并返回响应或抛出异常。

# 信号
信号组件新增于0.18版本。

尽管中间件可以自定义之前和之后的处理行为，但并不能自定义响应中的行为。所以信号量由此而生。

比如，中间件只能改变没有预定义HTTP头的响应的HTTP 头（看prepare()），但有时我们需要一个可以改变流式响应和WebSockets HTTP头的钩子。所以我们可以用on_response_prepare信号来充当这个钩子:
```
async def on_prepare(request, response):
    response.headers['My-Header'] = 'value'

app.on_response_prepare.append(on_prepare)
```

此外，你也可以用on_startup和on_cleanup信号来捕获应用开启和释放时的状态。

请看以下代码:
```
from aiopg.sa import create_engine

async def create_aiopg(app):
    app['pg_engine'] = await create_engine(
        user='postgre',
        database='postgre',
        host='localhost',
        port=5432,
        password=''
    )

async def dispose_aiopg(app):
    app['pg_engine'].close()
    await app['pg_engine'].wait_closed()

app.on_startup.append(create_aiopg)
app.on_cleanup.append(dispose_aiopg)
```

信号处理器不要返回内容，一般用于修改传入的可变对象。
信号处理器会循环执行，它们会一直累积。如果处理器是异步方式，在调用下个之前会一直等待。

### 警告:
信号API目前是临时状态，在未来可能会改变。

信号注册和发送方式基本不会被，不过信号对象的创建可能会变。只要你不创建新信号只用已经存在的信号量那基本不受影响。

# 嵌套应用
子应用用来完成某些特定的功能。比如我们有一个项目，有独立的业务逻辑和其他功能如管理功能和调试工具。

管理功能是一个独立于业务逻辑的功能，但使用的时候要在URL中加上如 /admin这样的前缀。
因此我们可以创建名为 admin的子应用，我们可以用add_subapp()来完成:
```
admin = web.Application()
# setup admin routes, signals and middlewares

app.add_subapp('/admin/', admin)
```

主应用和子应用间的中间件和信号是一个环一样的结构。
也就是说如果请求'/admin/something'会先调用主应用的中间件然后在调用子应用（admin.middlewares）的中间件。
信号也同样。
所有注册的基础信号如on_startup，on_shutdown，on_cleanup都会给子应用也注册一份。只不过传递的参数是子应用。
子应用也可以嵌套子应用。
子应用也可以使用Url反向引用，只不过会带上前缀:
```
admin = web.Application()
admin.router.add_get('/resource', handler, name='name')

app.add_subapp('/admin/', admin)

url = admin.router['name'].url_for()
```
这样我们得到的url是 '/admin/resource'。
如果主应用想得到子应用的Url反向引用，可以这样:
```
admin = web.Application()
admin.router.add_get('/resource', handler, name='name')

app.add_subapp('/admin/', admin)
app['admin'] = admin

async def handler(request):  # main application's handler
    admin = request.app['admin']
    url = admin.router['name'].url_for()
```

# 流控制
aiohttp.web有复杂的流控制机制来应对底层TCP套接字写入缓存。
问题是: 默认情况下TCP套接字使用Nagle算法来输出缓存，但这种算法对于流式数据协议如HTTP不是很理想。

Web服务器的响应是以下几种状态其中一个:

1. CORK（tcp_cork设置为True）。这个选项不会发送一部分TCP/IP帧。当这个选项被（再次）清除时会发送所有已经在队列中的片段帧。因为会把各种小帧聚合起来发送，所以这种方式对发送大量片段数据非常理想。 如果操作系统不支持CORK模式（不管是socket.TCP_CORK还是socket.TCP_NOPUSH）那该模式与Nagle模式一样。一般来说是windows系统不支持此模式。

2. NODELAY（tcp_nodelay设置为True）。这个选项会禁用Nagle算法。选用这个那么无论数据多小都会尽快发出去，即使是很小的数据。该模式对发送少量数据非常理想。

3. Nagle算法（tcp_cork和tcp_nodelay都为False）。该模式会先缓存数据，直到达到预定的数据大小后再一起发送。如果要发送HTTP数据应该避免使用这个模式除非你确定要使用它。

默认情况下，流数据（StreamResponse）, 标准响应（Response和http异常及其派生类）和websockets（WebSocketResponse）使用NODELAY模式，静态文件处理器使用CORK模式。

可以使用set_tcp_cork()方法和set_tcp_nodelay()方法手动切换。

# Expect头。

aiohttp.web支持使用Expect头。默认是HTTP/1.1 100 Continue，如果Expect头不是"100-continue"则抛出HTTPExpectationFailed异常。你可以自定义Expect头处理器。如果Expect头存在的话则会调用Expect处理器，Expect处理器会先于中间件和路由处理器被调用。Expect处理器可以返回None，返回None则会继续执行（调用中间件和路由处理器）。如果返回的是StreamResponse实例对象，之后请求处理器则使用该返回对象作为响应内容。也可以抛出HTTPException的子类对象。抛出错误的时候之后的处理将不会进行，客户端将会接受一个适当的http响应。
### 注意：
如果服务器不能理解或不支持的Expect域中的任何期望值，则服务器必须返回417或其他适当的错误码（4xx状态码）。
http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.20

如果所有的检查通过，在返回前自定义处理器需要将 HTTP/1.1 100 Continue写入。

下面是一个自定义Expect处理器的例子：

```
async def check_auth(request):
    if request.version != aiohttp.HttpVersion11:
        return

    if request.headers.get('EXPECT') != '100-continue':
        raise HTTPExpectationFailed(text="Unknown Expect: %s" % expect)

    if request.headers.get('AUTHORIZATION') is None:
        raise HTTPForbidden()

    request.transport.write(b"HTTP/1.1 100 Continue\r\n\r\n")

async def hello(request):
    return web.Response(body=b"Hello, world")

app = web.Application()
app.router.add_get('/', hello, expect_handler=check_auth)
```

# 自定义资源部署
使用 UrlDispatcher.register_resource()注册自定义的资源。资源实例必须有AbstractResource接口。

新增于1.2.1版本。

# 优雅的关闭
停止aiohttp web服务器时只关闭打开的连接时不够的。
因为可能会有一些websockets或流，在服务器关闭时这些连接还是打开状态。
aiohttp没有内置如何进行关闭，但开发者可以使用Applicaiton.on_shutdown信号来完善这一功能。
下面的代码是关闭websocket处理器的例子:
```
app = web.Application()
app['websockets'] = []

async def websocket_handler(request):
    ws = web.WebSocketResponse()
    await ws.prepare(request)

    request.app['websockets'].append(ws)
    try:
        async for msg in ws:
            ...
    finally:
        request.app['websockets'].remove(ws)

    return ws
```
信号处理器差不多这样：
```
async def on_shutdown(app):
    for ws in app['websockets']:
        await ws.close(code=WSCloseCode.GOING_AWAY,
                       message='Server shutdown')

app.on_shutdown.append(on_shutdown)
```

合适的关闭程序要注意以下死点:
1. 不在接受新的连接。注意调用asyncio.Server.close()和asyncio.Server.wait_closed()来关闭。
2. 解除Application.shutdown()事件。
3. 在一小段延迟后调用Server.shutdown()关闭已经开启的连接。
4. 发出Application.cleanup()信号。

下列代码演示从开始到结束：
```
loop = asyncio.get_event_loop()
handler = app.make_handler()
f = loop.create_server(handler, '0.0.0.0', 8080)
srv = loop.run_until_complete(f)
print('serving on', srv.sockets[0].getsockname())
try:
    loop.run_forever()
except KeyboardInterrupt:
    pass
finally:
    srv.close()
    loop.run_until_complete(srv.wait_closed())
    loop.run_until_complete(app.shutdown())
    loop.run_until_complete(handler.shutdown(60.0))
    loop.run_until_complete(app.cleanup())
loop.close()
```

# 后台任务
有些时候我们需要执行些异步操作。
甚至需要在请求处理器处理问题时进行一些后台操作。比如监听消息队列或者其他网络消息/事件资源（ZeroMQ，Redis Pub/Sub，AMQP等）然后接受消息并作出反应。
例如创建一个后台任务，用于在zmq.SUB套接字上监听ZeroMQ，然后通过WebSocket（app['websockets']）处理并转发接收到的消息给客户端。
使用Application.on_startup信号注册的后台任务可以让这些任务在应用的请求处理器执行时一并执行。
比如我们需要一个一次性的任务和两个常驻任务。最好的方法是通过信号注册：
```
async def listen_to_redis(app):
    try:
        sub = await aioredis.create_redis(('localhost', 6379), loop=app.loop)
        ch, *_ = await sub.subscribe('news')
        async for msg in ch.iter(encoding='utf-8'):
            # Forward message to all connected websockets:
            for ws in app['websockets']:
                ws.send_str('{}: {}'.format(ch.name, msg))
    except asyncio.CancelledError:
        pass
    finally:
        await sub.unsubscribe(ch.name)
        await sub.quit()


async def start_background_tasks(app):
    app['redis_listener'] = app.loop.create_task(listen_to_redis(app))


async def cleanup_background_tasks(app):
    app['redis_listener'].cancel()
    await app['redis_listener']


app = web.Application()
app.on_startup.append(start_background_tasks)
app.on_cleanup.append(cleanup_background_tasks)
web.run_app(app)
```
listen_to_redis()将会一直运行下去。当关闭时发出的on_cleanup信号会调用关闭处理器以关闭它。

# 处理错误页面
404 NotFound和500 Internal Error之类的错误页面可以看Middlewares章节了解详情，这些都可以通过自定义中间件完成。

# 基于代理部署服务器

