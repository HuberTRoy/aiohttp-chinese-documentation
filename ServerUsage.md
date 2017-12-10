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
    
