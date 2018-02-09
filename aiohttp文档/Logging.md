# 日志
aiohttp使用标准库logging追踪库活动。    

aiohttp中有以下日志记录器（以名字排序）:

* 'aiohttp.access'
* 'aiohttp.client'
* 'aiohttp.internal'
* 'aiohttp.server'
* 'aiohttp.web'
* 'aiohttp.websocket'

你可以追踪这些记录器来查看日志信息。此文档中没有关于配置日志追踪的说明，logging.config.dictConfig()对于将记录器配置到应用程序中来说已经很好用了。    

# 访问日志
默认情况下访问日志是开启的使用的是'aiohttp.access'记录器。   
可以调用aiohttp.web.Applicaiton.make_handler()来控制日志。   
将logging.Logger实例通过access_log参数传递即可覆盖默认记录器。

### 注意
    可以使用 web.run_app(app, access_log=None)来禁用记录访问日志。
其他参数如*access_log_format*可以指定日志格式（详情见下文）

# 指定格式
aiohttp提供自定义的微语言来指定请求和响应信息:

选项 |含义              
----|----              
%%    | 百分号
%a    | 远程IP地址(如果使用了反向代理则为代理IP地址)
%t    | 请求被处理时的时间
%P    | 处理请求的ID。
%r    | 请求的首行。
%s    | 响应状态码。
%b    | 响应字节码的大小，包括HTTP头。
%T    | 处理请求的时间，单位为秒。
%Tf  | 处理请求的时间，秒的格式为 %.06f。
%D    | 处理请求的时间，单位为毫秒。
%{FOO}i |request.headers['FOO']
%{FOO}o |response.headers['FOO']

访问日志的默认格式为:
```
'%a %l %u %t "%r" %s %b "%{Referrer}i" "%{User-Agent}i"'
```

**aiohttp.helper.AccessLogger**替换例子:
```
class AccessLogger(AbstractAccessLogger):

    def log(self, request, response, time):
        self.logger.info(f'{request.remote} '
                         f'"{request.method} {request.path} '
                         f'done in {time}s: {response.status}')
```

### 注意
    如果使用了Gunicorn来部署，默认的访问日式格式会自动被替换为aiohttp的默认访问日志格式。    
    如果要指定Gunicorn的访问日志格式，需要使用aiohttp的格式标识。

# 错误日志
aiohttpw.web使用`aiohttp.server`来存储处理web请求时的错误日志。    
默认情况下是开启的。   
使用不同的记录器名字请在调用`aiohttp.web.Application.make_handler()`时指定`logge`参数的值（需要`logging.Logger`实例对象）。

