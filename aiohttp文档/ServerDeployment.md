# 服务器部署
关于aiohttp服务器部署，这里有以下几种选择:
1. 独立的服务器。
2. 使用<a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/Glossary.md#nginx">nginx</a>, HAProxy等反向代理服务器，之后是后端服务器。
3. 在反向代理之后在部署一层<a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/Glossary.md#gunicorn">gunicorn</a>，然后才是后端服务器。

# 独立服务器
只需要调用`aiohttp.web.run_app()`,并传递`aiohttp.web.Application`实例即可。     
该方法最简单，也是在比较小的程序中最好的解决方法。但该方法并不能完全利用CPU。   
如果要运行多个aiohttp服务器实例请用反向代理。   

# Nginx + supervisord
将aiohttp服务器组运行在<a href="https://github.com/HuberTRoy/aiohttp-chinese-document/blob/master/aiohttp%E6%96%87%E6%A1%A3/Glossary.md#nginx">nginx</a>之后有好多好处。       
首先，nginx是个很好的前端服务器。它可以预防很多攻击如格式错误的http协议的攻击。     
第二，部署nginx后可以同时运行多个aiohttp实例，这样可以有效利用CPU。   
最后，nginx提供的静态文件服务器要比aiohttp内置的静态文件支持快很多。   

但部署它也意味着更复杂的配置。


## 配置Nginx

下面是一份简短的配置Nginx参考，并没涉及到所有的Nginx选项。    

你可以阅读<a href="https://www.nginx.com/resources/admin-guide/">Nginx指南</a>和<a href="http://nginx.org/en/docs/http/ngx_http_proxy_module.html">官方文档</a>来找到所有的参考。   

好啦，首先我们要配置HTTP服务器本身:
```
http {
  server {
    listen 80;
    client_max_body_size 4G;

    server_name example.com;

    location / {
      proxy_set_header Host $http_host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_redirect off;
      proxy_buffering off;
      proxy_pass http://aiohttp;
    }

    location /static {
      # path for static files
      root /path/to/app/static;
    }

  }
}
```

这样配置之后会监听80端口，服务器名为`example.com`，所有的请求都会被重定向到aiohttp后端处理组。     
静态文件也同样适用，/path/to/app/static路径的静态文件访问时就变成了 example.com/static。

接下来我们配置aiohttp上游组件:
```
http {
  upstream aiohttp {
    # fail_timeout=0 means we always retry an upstream even if it failed
    # to return a good HTTP response

    # Unix domain servers
    server unix:/tmp/example_1.sock fail_timeout=0;
    server unix:/tmp/example_2.sock fail_timeout=0;
    server unix:/tmp/example_3.sock fail_timeout=0;
    server unix:/tmp/example_4.sock fail_timeout=0;

    # Unix domain sockets are used in this example due to their high performance,
    # but TCP/IP sockets could be used instead:
    # server 127.0.0.1:8081 fail_timeout=0;
    # server 127.0.0.1:8082 fail_timeout=0;
    # server 127.0.0.1:8083 fail_timeout=0;
    # server 127.0.0.1:8084 fail_timeout=0;
  }
}
```

所有来自`http://example.com`的HTTP请求（除了`http://example.com/static`）都将重定向到example1.sock, example2.sock, example3.sock 或 example4.sock后端服务器。默认情况下，Nginx使用轮询调度算法（round-robin）来选择后端服务器。

### 注意
    Nginx 并不是反向代理的唯一选择，不过却是最流行的。你也可以选择HAProxy。

## Supervisord

配置完Nginx，我们要开始配置aiohttp后端服务器了。使用些工具可以在系统重启或后端出现错误时更快地自动启动。     
我们有很多工具可以选择: Supervisord, Upstart, Systemd, Gaffer, Runit等等。    
我们用<a href="http://supervisord.org/">Supervisord</a>当做例子：    
```
[program:aiohttp]
numprocs = 4
numprocs_start = 1
process_name = example_%(process_num)s

; Unix socket paths are specified by command line.
command=/path/to/aiohttp_example.py --path=/tmp/example_%(process_num)s.sock

; We can just as easily pass TCP port numbers:
; command=/path/to/aiohttp_example.py --port=808%(process_num)s

user=nobody
autostart=true
autorestart=true
```

## aiohtto服务器
最后我们要让aiohttp服务器在supervisord上工作。    
假设我们已经正确配置`aiohttp.web.Application`，端口也被正确指定，这些工作挺烦的:
```
# aiohttp_example.py
import argparse
from aiohttp import web

parser = argparse.ArgumentParser(description="aiohttp server example")
parser.add_argument('--path')
parser.add_argument('--port')


if __name__ == '__main__':
    app = web.Application()
    # configure app

    args = parser.parse_args()
    web.run_app(app, path=args.path, port=args.port)
```
当然在真实环境中我们还要做些其他事情，比如配置日志等等，但这些事情不在本章讨论范围内。

# Nginx + Gunicorn
我们还可以使用Gunicorn来部署aiohttp，<a href="http://docs.gunicorn.org/en/latest/index.html">Gunicorn</a>基于pre-fork worker模式。Gunicorn将你的app当做worker进程来处理即将到来的请求。     
与部署Ngnix相反，使用Gunicorn不需要我们手动启动aiohttp进程，也不需要使用如supervisord之类的工具进行监控。但这并不是没有代价：在Gunicorn下运行aiohttp应用会有些许缓慢。

## 准备环境
在做以上操作之前，我们首先要做的就是配置我们的部署环境。本章例子基于*Ubuntu 14.04*。    

首先为应用程序创建个目录:
```
>> mkdir myapp
>> cd myapp
```
在Ubuntu中使用pyenv有一个bug，所以我们需要做些额外的操作才能配置虚拟环境：
```
>> pyvenv-3.4 --without-pip venv
>> source venv/bin/activate
>> curl https://bootstrap.pypa.io/get-pip.py | python
>> deactivate
>> source venv/bin/activate
```
好啦，虚拟环境我们已经配置完了，之后我们要安装aiohttp和gunicorn:
```
>> pip install gunicorn
>> pip install -e git+https://github.com/aio-libs/aiohttp.git#egg=aiohttp
```

## 应用程序
我们写一个简单的应用程序，将其命名为 my_app_module.py:
```
from aiohttp import web

def index(request):
    return web.Response(text="Welcome home!")


my_web_app = web.Application()
my_web_app.router.add_get('/', index)
```

## 启动Gunicorn
<a href="http://docs.gunicorn.org/en/latest/run.html">启动Gunicorn</a>时我们要将模块名字（如*my_app_module*）和应用程序的名字（如*my_web_app*）传入，可以一起在<a href="http://docs.gunicorn.org/en/latest/settings.html">配置Gunicorn</a>其他选项时写入也可以写在配置文件中。      
本章例子所使用到的选项：
* -bind  用于设置服务器套接字地址。
* -worker-class 表示使用我们自定义的worker代替Gunicorn的默认worker。
* 你可能还想用 -workers 让Gunicorn知道应该用多少个worker来处理请求。（建议的worker数量请看 <a href="http://docs.gunicorn.org/en/latest/design.html#how-many-workers">设置多少个worker合适？</a>）

```
>> gunicorn my_app_module:my_web_app --bind localhost:8080 --worker-class aiohttp.GunicornWebWorker
[2015-03-11 18:27:21 +0000] [1249] [INFO] Starting gunicorn 19.3.0
[2015-03-11 18:27:21 +0000] [1249] [INFO] Listening at: http://127.0.0.1:8080 (1249)
[2015-03-11 18:27:21 +0000] [1249] [INFO] Using worker: aiohttp.worker.GunicornWebWorker
[2015-03-11 18:27:21 +0000] [1253] [INFO] Booting worker with pid: 1253
```
现在，Gunicorn已成功运行，随时可以将请求交由应用程序的worker处理。    

### 注意
如果你使用的是另一个asyncio事件循环<a href="https://github.com/MagicStack/uvloop">uvloop</a>, 你需要用`aiohttp.GunicornUVLoopWebWorker` worker类。

## 其他内容
Gunicorn 文档建议将Gunicorn部署在Nginx代理服务器之后。可看下官方文档的<a href="http://docs.gunicorn.org/en/latest/deploy.html">建议</a>。

## 配置日志
`aiohttp`和`Gunicorn`使用不同的日志格式。
默认aiohttp使用自己的日志格式：
```
'%a %l %u %t "%r" %s %b "%{Referrer}i" "%{User-Agent}i"'
```
查看<a href="">访问日志的格式</a>来获取更多信息。
