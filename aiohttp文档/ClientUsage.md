# 客户端使用

## 发起请求
让我们从导入aiohttp模块开始:

`import aiohttp`

好啦，我们来尝试获取一个web页面。比如我们来获取下GitHub的时间轴。
```
async with aiohttp.ClientSession() as session:
    async with session.get('https://api.github.com/events') as resp:
        print(resp.status)
        print(await resp.text())
```
我们现在有了一个`会话(session)`对象，由**ClientSession**对象赋值而来，还有一个变量`resp`，它其实是**ClientResponse**对象。我们可以从这个响应对象中获取我们任何想要的信息。协程方法`ClientSession.get()`的主要参数接受一个HTTP URL。

发起HTTP POST请求我们可以使用协程方法**ClientSession.post()**:
```
session.post('http://httpbin.org/post', data=b'data')
```
其他的HTTP方法也同样支持:
```
session.put('http://httpbin.org/put', data=b'data')
session.delete('http://httpbin.org/delete')
session.head('http://httpbin.org/get')
session.options('http://httpbin.org/get')
session.patch('http://httpbin.org/patch', data=b'data')
```
> ### 注意:
不要为每个请求都创建一个会话。大多数情况下每个应用程序只需要一个会话就可以执行所有的请求。
每个会话对象都包含一个连接池，可复用的连接和持久连接状态(keep-alives，这两个是默认的)可提升总体的执行效率。

## JSON请求:
每个会话的请求方法都可接受json参数。
```
async with aiohttp.ClientSession() as session:
    async with session.post(json={'test': 'object'})
```
默认情况下会话(session)使用Python标准库里的json模块解析json信息。但还可使用其他的json解析器。可以给ClientSession指定*json_serialize*参数来实现:
```
import ujson

async with aiohttp.ClientSession(json_serialize=ujson.dumps) as session:
    async with session.post(json={'test': 'object'})
```

## 传递URL中的参数:
你可能经常想在URL中发送一系列的查询信息。如果你手动构建他们，这些信息会以键值对的形式出现在?后面，比如: `httpbin.org/get?key=val`。请求对象允许你使用**dict(字典，python中的数据类型)**发送它们，使用`params`参数即可。例如: 如果你要把 `key1=value1，key2=value2`放到`httpbin.org/get`后面，你可以用下面的方式:

```
params = {'key1': 'value1', 'key2': 'value2'}
async with session.get('http://httpbin.org/get',
                       params=params) as resp:
    assert str(resp.url) == 'http://httpbin.org/get?key2=value2&key1=value1'
```
看，URL已经被正确的编码啦。
同键不同值的**并联字典(MultiDict )**也同样支持。
可使用带有两个tuples(元组，python中的数据类型)的list(列表，python中的数据类型)来构建:
```
params = [('key', 'value1'), ('key', 'value2')]
async with session.get('http://httpbin.org/get',
                       params=params) as r:
    assert str(r.url) == 'http://httpbin.org/get?key=value2&key=value1'
```
同样也允许你传递**str**（字符串）给params，但要小心一些不能被编码的字符。`+`就是一个不能被编码的字符:
```
async with session.get('http://httpbin.org/get',
                       params='key=value+1') as r:
        assert str(r.url) == 'http://httpbin.org/get?key=value+1'
```
> ### 注意:
*aiohttp*会在发送请求前标准化URL。
域名部分会用<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-idna">IDNA</a> 编码，路径和查询条件会<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-requoting">重新编译(requoting)</a>。
比如:`URL('http://example.com/путь%30?a=%31')` 会被转化为`URL('http://example.com/%D0%BF%D1%83%D1%82%D1%8C/0?a=1')`
如果服务器需要接受准确的表示并不要求编译URL，那标准化过程应是禁止的。
禁止标准化可以使用`encoded=True`:
```
await session.get(URL('http://example.com/%30', encoded=True))
```

> ### 警告:
传递*params*时不要用`encode=True`，这俩参数不能同时使用。

## 响应内容:
我们可以读取服务器的响应内容。想想我们获取GitHub时间轴的例子:
```
async with session.get('https://api.github.com/events') as resp:
    print(await resp.text())
```
这样会打印出类似于下面的信息:
```
'[{"created_at":"2015-06-12T14:06:22Z","public":true,"actor":{...
```
`aiohttp`将会自动解码内容。你可以为**text()**方法指定编码(使用encoding参数):
```
await resp.text(encoding='windows-1251')
```

## 二进制响应内容:
你也可以以字节形式获取响应，这样得到的就不是文本了:
```
print(await resp.read())
b'[{"created_at":"2015-06-12T14:06:22Z","public":true,"actor":{...
```
`gzip`和`defalte`传输编码会自动解码。
你也可以使其支持`brotli`传输编码的解码，只需安装<a href="https://github.com/python-hyper/brotlipy">brotlipy</a>即可。

## JSON响应内容:
以防你需要处理JSON数据，内置了一个JSON解码器:
```
async with session.get('https://api.github.com/events') as resp:
    print(await resp.json())
```
如果JSON解码失败，**json**()方法将会抛出一个异常。你还可以在调用**json**()时指定编码器和解码器函数。

> ### 注意:
这些方法会读出内存中所有响应的内容。如果你要读非常多的数据，考虑使用流式响应方法进行读取。请看之后的文档。

## 流式响应内容:
**read**(), **json**(), **text**()等方法使用起来很方便，但也要注意谨慎地使用。上述方法会将所有的响应内容加载到内存。举个例子，如果你要下载几个G的文件，这些方法还是会将所有内容都加载到内存，内存会表示"臣妾做不到啊~"(如果内存不够的话)。作为代替你可以用**content**属性。content其实是 **aiohttp.StreamReader**类的实例。`gzip`和`deflate`传输编码同样会自动解码。

```
async with session.get('https://api.github.com/events') as resp:
    await resp.content.read(10)
```
一般情况下你可以使用下列模式将内容保存在一个文件中:
```
with open(filename, 'wb') as fd:
    while True:
        chunk = await resp.content.read(chunk_size)
        if not chunk:
            break
        fd.write(chunk)
```
在使用**content**读了数据后，就不要在用**read**(), **json**(), **text**()了。

## 请求信息:
*ClientResponse（客户端响应）*对象含有request_info(请求信息)，主要是*url*和*headers*信息。 *raise_for_status*结构体上的信息会被复制给ClientResponseError实例。

## 自定义Headers:
如果你需要给某个请求添加HTTP头,可以使用headers参数，传递一个**dict**对象即可。
比如，如果你想给之前的例子指定 content-type可以这样:
```
import json
url = 'https://api.github.com/some/endpoint'
payload = {'some': 'data'}
headers = {'content-type': 'application/json'}

await session.post(url,
                   data=json.dumps(payload),
                   headers=headers)
```

## 自定义Cookies:
发送你自己的cookies给服务器，你可以为**ClientSession**对象指定*cookies*参数:
```
url = 'http://httpbin.org/cookies'
cookies = {'cookies_are': 'working'}
async with ClientSession(cookies=cookies) as session:
    async with session.get(url) as resp:
        assert await resp.json() == {
           "cookies": {"cookies_are": "working"}}
```
> ### 注意:
访问`httpbin.org/cookies` 会看到以JSON形式返回的cookies。查阅会话中的cookies请看<a href="https://aiohttp.readthedocs.io/en/stable/client_reference.html#aiohttp.ClientSession.cookie_jar">ClientSession.cookie_jar</a>。

## 更复杂的POST请求:
一般来说，如果你想以表单形式发送一些数据 - 就像HTML表单。那么只需要简单的将一个dict通过*data*参数传递就可以。传递的dict数据会自动编码:
```
payload = {'key1': 'value1', 'key2': 'value2'}
async with session.post('http://httpbin.org/post',
                        data=payload) as resp:
    print(await resp.text())
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```
如果你想发送非表单形式的数据你可用`str(字符串)`代替`dict(字典)`。这些数据会直接发送出去。
例如，GitHub API v3 接受JSON编码POST/PATCH数据:
```
import json
url = 'https://api.github.com/some/endpoint'
payload = {'some': 'data'}

async with session.post(url, data=json.dumps(payload)) as resp:
    ...
```

## 发送多部分编码文件(Multipart-Encoded):

上传多部分编码文件:
```
url = 'http://httpbin.org/post'
files = {'file': open('report.xls', 'rb')}

await session.post(url, data=files)
```
你也可以显式地设置文件名，文件类型:
```
url = 'http://httpbin.org/post'
data = FormData()
data.add_field('file',
               open('report.xls', 'rb'),
               filename='report.xls',
               content_type='application/vnd.ms-excel')

await session.post(url, data=data)
```
如果你把一个文件对象传递给data参数，aiohttp会自动将其以流的形式上传。查看**StreamReader**以获取支持的格式信息。

> ### 参见:
<a href="https://aiohttp.readthedocs.io/en/stable/multipart.html#aiohttp-multipart">使用Multipart.</a>

## 流式上传:
**aiohttp** 支持多种形式的流式上传，允许你直接发送大文件而不必读到内存。

下面是个简单的例子，提供类文件对象即可:
```
with open('massive-body', 'rb') as f:
   await session.post('http://httpbin.org/post', data=f)
```
或者你也可以使用*aiohttp.streamer*对象：
```
@aiohttp.streamer
def file_sender(writer, file_name=None):
    with open(file_name, 'rb') as f:
        chunk = f.read(2**16)
        while chunk:
            yield from writer.write(chunk)
            chunk = f.read(2**16)

# 之后你可以使用’file_sender‘传递给data:

async with session.post('http://httpbin.org/post',
                        data=file_sender(file_name='huge_file')) as resp:
    print(await resp.text())
```
同样可以使用**StreamReader**对象.

我们来看下如何把来自于另一个请求的内容作为文件上传并计算其SHA1值:
```
async def feed_stream(resp, stream):
    h = hashlib.sha256()

    while True:
        chunk = await resp.content.readany()
        if not chunk:
            break
        h.update(chunk)
        stream.feed_data(chunk)

    return h.hexdigest()

resp = session.get('http://httpbin.org/post')
stream = StreamReader()
loop.create_task(session.post('http://httpbin.org/post', data=stream))

file_hash = await feed_stream(resp, stream)
```
因为响应对象的content属性是一个`StreamReader`实例，所以你可以将get和post请求连在一起用:
```
r = await session.get('http://python.org')
await session.post('http://httpbin.org/post',
                   data=r.content)
```

## 上传预压缩过的数据:
上传一个已经压缩过的数据，需要为Headers中的`Content-Encoding`指定算法名(通常是deflate或者是zlib).
```
async def my_coroutine(session, headers, my_data):
    data = zlib.compress(my_data)
    headers = {'Content-Encoding': 'deflate'}
    async with session.post('http://httpbin.org/post',
                            data=data,
                            headers=headers)
        pass
```

## 持久连接(keep-alive), 连接池和cookies共享:
**ClientSession**可以在多个请求之间共享cookies:
```
async with aiohttp.ClientSession() as session:
    await session.get(
        'http://httpbin.org/cookies/set?my_cookie=my_value')
    filtered = session.cookie_jar.filter_cookies('http://httpbin.org')
    assert filtered['my_cookie'].value == 'my_value'
    async with session.get('http://httpbin.org/cookies') as r:
        json_body = await r.json()
        assert json_body['cookies']['my_cookie'] == 'my_value'
```
你也可以为所有的会话请求设置headers:
```
async with aiohttp.ClientSession(
    headers={"Authorization": "Basic bG9naW46cGFzcw=="}) as session:
    async with session.get("http://httpbin.org/headers") as r:
        json_body = await r.json()
        assert json_body['headers']['Authorization'] == \
            'Basic bG9naW46cGFzcw=='
```
**ClientSession**支持持久连接和连接池，可直接使用，不需要额外操作。

## 安全cookies:
**ClientSession**中的默认的**aiohttp.CookiesJar**使用的是严苛模式，<a href="https://tools.ietf.org/html/rfc2109.html">RFC 2109</a>明确禁止使用ip地址形式的URL携带cookies信息。比如: *http://127.0.0.1:80/cookie*
这样很好，不过有些时候我们测试时需要允许携带cookies。在**aiohttp.CookiesJar**中传递*unsafe=True*来实现这一效果:

```
jar = aiohttp.CookieJar(unsafe=True)
session = aiohttp.ClientSession(cookie_jar=jar)
```

## 假人(Dummy) Cookie Jar:
有时不想处理cookie。这时可以在会话中使用**aiohttp.DummyCookieJar**来达到目的。
```
jar = aiohttp.DummyCookieJar()
session = aiohttp.ClientSession(cookie_jar=jar)
```

## 连接器:
调整请求的传输层你可以为**ClientSession**及其同类组件传递自定义的连接器。例如:
```
conn = aiohttp.TCPConnector()
session = aiohttp.ClientSession(connector=conn)
```

> ### 注解:
不要给多个会话对象使用同一个连接器，某一会话对象拥有其所有权。

> ### 参见:
查看<a href="https://aiohttp.readthedocs.io/en/stable/client_reference.html#aiohttp-client-reference-connectors">连接器</a>部分了解更多不同的连接器类型和配置选项信息。

## 限制连接池的容量:
限制同一时间打开的连接数可以传递limit参数:
```
conn = aiohttp.TCPConnector(limit=30)
```
这样就将总数限制在30.

默认情况下是100.

如果你不想有限制，传递0即可:
```
conn = aiohttp.TCPConnector(limit=0)
```

限制同一时间在同一个端点((`host`, `port`, `is_ssl`) 原文写了triple难道要表达"重要的事情说3遍？！")打开的连接数可指定limit_per_host参数:
```
conn = aiohttp.TCPConnector(limit_per_host=30)
```
这样会限制在30.
默认情况下是0(也就是不做限制)。

## 使用自定义域名服务器:
底层需要<a href="https://aiohttp.readthedocs.io/en/stable/glossary.html#term-aiodns">aiodns</a>支持:
```
from aiohttp.resolver import AsyncResolver

resolver = AsyncResolver(nameservers=["8.8.8.8", "8.8.4.4"])
conn = aiohttp.TCPConnector(resolver=resolver)
```

## 为TCP sockets添加SSL控制:
默认情况下aiohttp总会对使用了HTTPS协议(的URL请求)查验其身份。但也可将*verify_ssl*设置为`False`让其不检查:
```
r = await session.get('https://example.com', verify_ssl=False)
```
如果你需要设置自定义SSL信息(比如使用自己的证书文件)你可以创建一个**ssl.SSLContext**实例并传递到**ClientSession**中:
```
sslcontext = ssl.create_default_context(
   cafile='/path/to/ca-bundle.crt')
r = await session.get('https://example.com', ssl_context=sslcontext)
```
如果你要验证*自签名*的证书，你也可以用之前的例子做同样的事，但是用的是load_cert_chain():
```
sslcontext = ssl.create_default_context(
   cafile='/path/to/ca-bundle.crt')
sslcontext.load_cert_chain('/path/to/client/public/device.pem',
                           '/path/to/client/private/device.jey')
r = await session.get('https://example.com', ssl_context=sslcontext)
```
SSL验证失败时抛出的错误:

**aiohttp.ClientConnectorSSLError**:
```
try:
    await session.get('https://expired.badssl.com/')
except aiohttp.ClientConnectorSSLError as e:
    assert isinstance(e, ssl.SSLError)
```
**aiohttp.ClientConnectorCertificateError**:
```
try:
    await session.get('https://wrong.host.badssl.com/')
except aiohttp.ClientConnectorCertificateError as e:
    assert isinstance(e, ssl.CertificateError)
```
如果你需要忽略所有SSL的错误:

**aiohttp.ClientSSLError**:
```
try:
    await session.get('https://expired.badssl.com/')
except aiohttp.ClientSSLError as e:
    assert isinstance(e, ssl.SSLError)

try:
    await session.get('https://wrong.host.badssl.com/')
except aiohttp.ClientSSLError as e:
    assert isinstance(e, ssl.CertificateError)
```
你还可以通过*SHA256*指纹验证证书:
```
# Attempt to connect to https://www.python.org
# with a pin to a bogus certificate:
bad_fingerprint = b'0'*64
exc = None
try:
    r = await session.get('https://www.python.org',
                          fingerprint=bad_fingerprint)
except aiohttp.FingerprintMismatch as e:
    exc = e
assert exc is not None
assert exc.expected == bad_fingerprint

# www.python.org cert's actual fingerprint
assert exc.got == b'...'
```
注意这是以DER编码的证书的指纹。如果你的证书是PEM编码，你需要转换成DER格式:
```
openssl x509 -in crt.pem -inform PEM -outform DER > crt.der
```
> ### 注解:
提示: 从16进制数字转换成二进制字节码，你可以用**binascii.unhexlify()**.

**TCPConnector**中设置的*verify_ssl, fingerprint和ssl_context*都会被当做默认的verify_ssl, fingerprint和ssl_context，**ClientSession**或其他同类组件中的设置会覆盖默认值。

> ### 警告:
*verify_ssl 和 ssl_context*是*互斥*的。
*MD5*和*SHA1*指纹虽不赞成使用但是是支持的 - 这俩是非常不安全的哈希函数。

## Unix 域套接字:
如果你的服务器使用UNIX域套接字你可以用**UnixConnector**:

```
conn = aiohttp.UnixConnector(path='/path/to/socket')
session = aiohttp.ClientSession(connector=conn)
```

## 代理支持:
aiohttp 支持 HTTP/HTTPS形式的代理。你需要使用*proxy*参数:
```
async with aiohttp.ClientSession() as session:
    async with session.get("http://python.org",
                           proxy="http://some.proxy.com") as resp:
        print(resp.status)
```
同时支持认证代理:
```
async with aiohttp.ClientSession() as session:
    proxy_auth = aiohttp.BasicAuth('user', 'pass')
    async with session.get("http://python.org",
                           proxy="http://some.proxy.com",
                           proxy_auth=proxy_auth) as resp:
        print(resp.status)
```
也可将代理的验证信息放在url中:
```
session.get("http://python.org",
            proxy="http://user:pass@some.proxy.com")
```
与`requests(另一个广受欢迎的http包)`不同，aiohttp默认不会读取环境变量中的代理值。但你可以通过传递`trust_env=True`来让**aiohttp.ClientSession**读取*HTTP_PROXY*或*HTTPS_PROXY*环境变量中的代理信息(不区分大小写)。
```
async with aiohttp.ClientSession() as session:
    async with session.get("http://python.org", trust_env=True) as resp:
        print(resp.status)
```

## 响应状态码:
我们可以查询响应状态码:
```
async with session.get('http://httpbin.org/get') as resp:
    assert resp.status == 200
```

## 响应头信息:
我们可以查看服务器的响应信息, **ClientResponse.headers**使用的数据类型是**CIMultiDcitProxy**:
```
>>> resp.headers
{'ACCESS-CONTROL-ALLOW-ORIGIN': '*',
 'CONTENT-TYPE': 'application/json',
 'DATE': 'Tue, 15 Jul 2014 16:49:51 GMT',
 'SERVER': 'gunicorn/18.0',
 'CONTENT-LENGTH': '331',
 'CONNECTION': 'keep-alive'}
 ```
这是一个特别的字典，它只为HTTP头信息而生。根据<a href="http://tools.ietf.org/html/rfc7230#section-3.2"> RFC 7230</a>，HTTP头信息中的名字是不分区大小写的。同时也支持多个不同的值对应同一个键。

所以我们可以通过任意形式访问它:
```
>>> resp.headers['Content-Type']
'application/json'

>>> resp.headers.get('content-type')
'application/json'
```
所有的header信息都是由二进制数据转换而来，使用带有`surrogateescape`选项的UTF-8编码方式(surrogateescape是一种错误处理方式，详情看<a href="https://docs.python.org/3/library/codecs.html#error-handlers" 这里</a>))。大部分时候都可以很好的工作，但如果服务器使用的不是标准编码就不能正常解码了。从<a href="http://tools.ietf.org/html/rfc7230#section-3.2"> RFC 7230</a>的角度来看这样的headers并不是合理的格式，你可以用**ClientReponse.resp.raw_headers**来查看原形:
```
>>> resp.raw_headers
((b'SERVER', b'nginx'),
 (b'DATE', b'Sat, 09 Jan 2016 20:28:40 GMT'),
 (b'CONTENT-TYPE', b'text/html; charset=utf-8'),
 (b'CONTENT-LENGTH', b'12150'),
 (b'CONNECTION', b'keep-alive'))
```

## 响应cookies:
如果某响应包含一些Cookies，你可以很容易地访问他们:
```
url = 'http://example.com/some/cookie/setting/url'
async with session.get(url) as resp:
    print(resp.cookies['example_cookie_name'])
```
> ### 注意:
响应中的cookies只包含重定向链中最后一个请求中的`Set-Cookies`头信息设置的值。如果每一次重定向请求都收集一次cookies请使用<a href="https://aiohttp.readthedocs.io/en/stable/client.html#aiohttp-client-session"> aiohttp.ClientSession</a>对象.

## 响应历史:
如果一个请求被重定向了，你可以用**history**属性查看其之前的响应:
```
>>> resp = await session.get('http://example.com/some/redirect/')
>>> resp
<ClientResponse(http://example.com/some/other/url/) [200]>
>>> resp.history
(<ClientResponse(http://example.com/some/redirect/) [301]>,)
```
如果没有重定向或`allow_redirects`设置为`False`，history会被设置为空。

## WebSockets:
**aiohttp**提供开箱即用的客户端websocket。
你需要使用**aiohttp.ClientSession.ws_connect()**协程对象。它的第一个参数接受URL，返回值是**ClientWebSocketResponse**，这样你就可以用响应的方法与websocket服务器进行通信。
```
session = aiohttp.ClientSession()
async with session.ws_connect('http://example.org/websocket') as ws:

    async for msg in ws:
        if msg.type == aiohttp.WSMsgType.TEXT:
            if msg.data == 'close cmd':
                await ws.close()
                break
            else:
                await ws.send_str(msg.data + '/answer')
        elif msg.type == aiohttp.WSMsgType.CLOSED:
            break
        elif msg.type == aiohttp.WSMsgType.ERROR:
            break
```
你只能使用一种读取方式(例如`await ws.receive()` 或者 `async for msg in ws:`)和写入方法，但可以有多个写入任务，写入任务也是异步完成的(`ws.send_str('data')`)。

## 超时:
默认情况下每个IO操作有5分钟超时时间。可以通过给**ClientSession.get()**及其同类组件传递`timeout`来覆盖原超时时间:
```
async with session.get('https://github.com', timeout=60) as r:
    ...
```
`None` 或者`0`则表示不检测超时。
还可通过调用**async_timeout.timeout**上下文管理器来为连接和解析响应内容添加一个总超时时间:
```
import async_timeout

with async_timeout.timeout(0.001):
    async with session.get('https://github.com') as r:
        await r.text()
```
> ### 注意:
超时时间是累计的，包含如发送情况，重定向，响应解析，处理响应等所有操作在内...

## 不错的结尾:
当一个包含`ClientSession`的`async with`代码块的末尾行结束时(或直接调用了`.close()`)，因为asyncio内部的一些原因底层的连接其实没有关闭。在实际使用中，底层连接需要有一个缓冲时间来关闭。然而，如果事件循环在底层连接关闭之前就结束了，那么会抛出一个 资源警告: 存在未关闭的传输(通道)(`ResourceWarning: unclosed transport`),如果警告可用的话。
为了避免这种情况，在关闭事件循环前加入一小段延迟让底层连接得到关闭的缓冲时间。
对于非SSL的`ClientSession`, 使用0即可(`await asyncio.sleep(0)`):
```
async def read_website():
    async with aiohttp.ClientSession() as session:
        async with session.get('http://example.org/') as response:
            await response.read()

loop = asyncio.get_event_loop()
loop.run_until_complete(read_website())
# Zero-sleep to allow underlying connections to close
loop.run_until_complete(asyncio.sleep(0))
loop.close()
```
对于使用了SSL的`ClientSession`, 需要设置一小段合适的时间:
```
...
# Wait 250 ms for the underlying SSL connections to close
loop.run_until_complete(asyncio.sleep(0.250))
loop.close()
```
合适的时间因应用程序而异。

当asyncio内部的运行机制改变时就可以让aiohttp去等待底层连接关闭在退出啦，上面这种额外的方法总会废弃啦。你也可以跟进问题<a href="https://github.com/aio-libs/aiohttp/issues/1925">#1925</a>来参与改进。
