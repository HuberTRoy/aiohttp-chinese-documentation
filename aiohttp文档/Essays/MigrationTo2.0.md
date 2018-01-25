# 走进2.x

##  客户端方面
### 分块
aiohttp现在不支持自定义分块大小了。至于分块的多大取决于开发者决定要放多少分块数据。如果`chunking`是允许的，aiohttp会将分块的内容以“Transfer-encoding: chunked”提供的格式编码。    
aiohttp并不能自动编码分块内容，即使指定了`transfer-encoding`头信息也不行， 必须显式地设置`chunked`。如果设置了`chunked`，`Transfer-encoding`和`content-length`头信息就不能指定了。

### 压缩
可以显式地使用compress参数来开启压缩功能。如果压缩功能开启，就不能添加`content-encoding`头信息了。压缩同样适用于分块传输编码。同样，压缩也并不能和`Content-Length`一块使用。

### 客户端连接器
默认只有一个连接器对象管理着所有的连接。1.x版本的规则是每台主机有多少限制。到了2.x，`limit`参数定义当前有多少连接可以打开，新的参数`limit_per_host`则是指定每台主机的限制。默认是无限制的。
与1.x不同的是，`BaseConnector.close`现在是常规方法。
`BaseConnector.conn_timeout`也移动到了`ClientSession`中。

### ClientResponse.release
内部结构有了很大改变，做了很多调整。得到响应对象后不需要再调用`release`。 当客户端完全接受到内容后，底层连接会自动返回到连接池中。如果内容并没有完全读取，连接将会关闭。

### 客户端异常
异常的等级制度也有了极大改变。aiohttp现在只定义了一个异常，它覆盖了连接处理和服务器异常响应。对于开发者的错误，aiohttp现在使用python的标准异常如`ValueError`，·`TypeError`。
读取响应内容可能会抛出`ClientPayloadError`异常。 这个异常表示载体的编码出了错误。像是无效的压缩信息，不合理的分块编码块或是内容长度与`content-length`指定的不匹配等等。
所有的异常已被移动到顶级模块aiohttp.errors模块。
新的异常等级制度:
* ClientError - 所以异常的基类。
* - ClientResponseError - 从服务器获得相应后有可能被抛出。
* - - WSServerHandshakeError -  web socket服务器响应错误。
* - - - ClientHttpProxyError - 代理响应错误。
* - ClientConnectionError - 表示低级连接方面的相关问题。
* - - ClientOSError - 连接错误的子集，继承于OSError异常。
* - - - ClientConnectorError - 连接器方面的相关问题。
* - - - - ClientProxyConnectionError - 代理连接初始化时的问题。
* - - - - - ServerConnectionError - 服务器连接方面的相关问题。
* - - - - ServerDisconnectedError - 服务器断开连接的异常。
* - - - - ServerTimeoutError - 服务器操作超时的异常（比如读取超时）。
* - - - - ServerFingerprintMismatch - 服务器指纹不匹配的异常。
* - ClientPayloadError - 此错误只会在读取响应载体时发生以下错误: 无效的压缩信息，不合理的分块编码或长度不匹配的内容才会被抛出。

### 客户端payload（form-data）
为了整合form-data/payload(载体)的处理，引入了一个新的Payload系统。它可以执行现存类型的处理，也可以处理由用户定义的类型。
`FormData.__call__`不再需要`encoding`参数，返回的内容也由迭代器对象或字节变为Payload实例。对于标准类型如`str`，`byte`，`io.IOBase`，·`StreamReader`，`DataQueue`，aiohttp都提供相关payload适配器。
生成器不能再作为数据生成器提供，你可以用streamer代替。比如，上传文件可以这样:
```
@aiohttp.streamer
def file_sender(writer, file_name=None):
      with open(file_name, 'rb') as f:
          chunk = f.read(2**16)
          while chunk:
              yield from writer.write(chunk)
              chunk = f.read(2**16)

# Then you can use `file_sender` like this:

async with session.post('http://httpbin.org/post',
                        data=file_sender(file_name='huge_file')) as resp:
       print(await resp.text())
```

### 其他杂项
`ClientSession.request()`中不再赞成使用`encoding`参数。Payload编码由payload层控制。为payload实例指定编码还是可以的。
`version`参数从`ClientSession.request()`中移除，客户端版本可以在`ClientSession`构造器中指定。
使用`aiohttp.WSMsgType`代替`aiohttp.MsgType`。
`ClientResponse.url`现在是一个yarl.URL实例（url_obj不在赞成使用）。
`ClientResponse.raise_for_status()`将抛出`aiohttp.ClientResponseError`异常。
`ClientResponse.json()`严格按照响应的内容类型来解析。如果内容类型不匹配，将会抛出`aiohttp.ClientResponseError`异常。你可以通过传递`content_type=None`来关闭内容类型检测。

# 服务器方面

## ServerHttpProtocol和低级服务器的一些详情
为了提供更好的执行效率和支持HTTP流水线，内部重新进行了设计。`ServerHttpProtocol`已移除，与`RequestHandler`合并在一起，很多低级api也被移除。

## 应用方面
1. 构造器的loop已不赞成再使用。Loop会从应用运行器中获取，*run_app*函数可以被任意gunicorn worker使用。
2. `Application.router.add_subapp`已移除，使用`Application.add_subapp`代替。
3. `Application.finished`已移除，使用`Application.cleanup`代替。

## WebRequest和WebResponse
1. `GET`和`POST`属性将不再存在。使用`query`代替`GET`。
2. WebResponse.chunked不支持自定义分块大小 - 开发者有义务手动分块。
3. Payloads 的功能和body（响应体）差不多。所以在WebResponse中可以使用客户端响应的内容对象作为body参数的值。
4. `FileSender`api已移除，被更通用的`FileResponse`类所代替:
```
async def handle(request):
    return web.FileResponse('path-to-file.txt')
```
5. `WebSocketResponse.protocol`重命名为`WebSocketResponse.ws_protocol`。`WebSocketResponse.protocol`现在是`RequestHandler`类的实例对象。

## RequestPayloadError
读取请求的payload时可能会抛出`RequestPayloadError`异常。与`ClientPayloadError`类似。

## WSGI
WSGI的支持现已移除，现在支持gunicorn wsgi。我们仍然对`web.Application`提供默认功能和基于uvloop的gunicorn worker。




