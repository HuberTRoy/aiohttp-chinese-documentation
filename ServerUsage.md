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