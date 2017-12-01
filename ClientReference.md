## 客户端会话(Client Session):
客户端会话(Client Session)是比较推荐使用的发起HTTP请求的接口。

会话(Session)封装有一个连接池(连接器实例)，默认支持持久连接。除非你要连接非常多不同的服务器，否则建议你在你的应用程序中只使用一个会话(Session)，这样有利于连接池.

使用案例:
```
import aiohttp
import asyncio

async def fetch(client):
    async with client.get('http://python.org') as resp:
        assert resp.status == 200
        return await resp.text()

async def main():
    async with aiohttp.ClientSession() as client:
        html = await fetch(client)
        print(html)

loop = asyncio.get_event_loop()
loop.run_until_complete(main(loop))
```
*新增于0.17版本。*

客户端会话(ClientSession)支持使用上下文管理器在结束时自动关闭:

*class aiohttp.**ClientSession**(\*, connector=None, loop=None, cookies=None, headers=None, skip_auto_headers=None, auth=None, json_serialize=json.dumps, version=aiohttp.HttpVersion11, cookie_jar=None, read_timeout=None, conn_timeout=None, raise_for_status=False, connector_owner=True, auto_decompress=True, proxies=None)*
    该类用于创建客户端会话以及发起请求。

    **(参数)Parameters**: 
        * connector (aiohttp.connector.BaseConnector) – 基础连接器(BaseConnector)的子类实例，用于支持连接池。
        * loop –
            事件循环(event loop)用于处理HTTP请求。
            如果loop为None,则从connector中获取(如果也有的话)。
            一般用asyncio.get_event_loop()来获取事件循环。 
            自 2.0版本后已被弃用。(2.0版本发布日是2017/3/20，之前的文档内写明遭到弃用的还是会保留一年半。)
        * cookies (dict) –  发送请求时所携带的cookies(可选)。
        * headers – 
                所有请求发送时携带的HTTP头(可选)。
                可以是任何可迭代的键值对(key-value)对象或Mapping.(例如: dict, CIMultiDict)
        * skip_auto_headers –
                跳过会自动产生的headers值.
                如果没有显式传递，那么aiohttp会自动生成诸如User-Agent或Content-Type。使用该参数可以指定跳过，但注意Content-Length是不能跳过的。
                接受str或istr。
        * auth(aiohttp.BasicAuth) – <a href="https://zh.wikipedia.org/wiki/HTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81"> HTTP基本认证</a>。
        * version – 
                使用的HTTP版本，默认是HTTP 1.1。
                新增于0.21版本。
        * cookie_jar –
                Cookie jar ,AbstractCookieJar的实例。
                默认情况下每个会话实例都有自己的私有cookie jar用于自动处理cookies，但用户也可使用自己的jar重新定义其行为。
                在代理模式下不会处理cookies。
                如果不需要cookie处理，可以传递一个aiohttp.helper.DummyCookieJar实例。
                新增于0.22版本。
        * json_serialize (可调用对象) – 
                可调用的Json序列化对象，默认是json.dumps()函数。
        * raise_for_status (布尔类型) –
                每完成一个响应就自动调用 ClientResponse.raise_for_status(), 默认是False.
                新增于2.0版本。
        * read_timeout (浮点数) – 
                请求操作的超时时间。read_timeout是累计自所有的请求操作(请求，重定向，响应，数据处理)。默认情况下是 5\*60秒(5分钟)。传递None或0来禁用超时检测。
        * conn_timeout (浮点数) – 
                建立连接的超时时间(可选)。0或None则禁用超时检测。
        * connector_owner (布尔类型) –
                会话关闭时一同关闭连接器实例。
                传递False让构造器允许在多个会话间共享连接池，不过cookies等信息是不共享的。
                新增于2.1版本。
        * auto_decompress (布尔类型) –
                自动解压响应体。
                新增于2.3版本。
        * trust_env(布尔类型) –
                若该参数设置为True则从环境变量HTTP_PROXY / HTTPS_PROXY中获取代理信息。
                新增于2.3版本。

    **closed**
        若会话已关闭则返回True，否则返回False。
        该属性只读。

    **connector**
        **aiohttp.connector.BaseConnector**的实例对象，通常用于会话。 
        该属性只读。

    **cookie_jar**
        会话中的cookies，**AbstractCookieJar**的实例对象。
        用于访问和修改cookie jar中的内容。
        该属性只读。
        新增于1.0版本。

    **requote_redirect_url**
        aiohttp默认会重新编译传入的urls，但有些时候服务器需要未编译的url。将其设置为False禁用编译。
        新增于2.1版本。
        ### 注意:
            设置后会影响这之后的所有请求。

    **loop**
        返回用于会话创建的循环(loop)实例对象。
        该属性只读。

    *coroutine async-with **request**(method, url, \*, params=None, data=None, json=None, headers=None, skip_auto_headers=None, auth=None, allow_redirects=True, max_redirects=10, compress=None, chunked=None, expect100=False, read_until_eof=True, proxy=None, proxy_auth=None, timeout=5\*60, verify_ssl=None, fingerprint=None, ssl_context=None, proxy_headers=None)*
        执行异步HTTP请求，返回响应对象。
    
        **Parameters**: 
                * method (字符串) – HTTP方法，接受字符串。
                * url – 请求URL, 接受字符串或URL对象.
                * params –
                    可传入Maaping对象，有键值对的元组或字符串，会在发送新请求时作为查询字符串发送。不会对之后的重定向请求也传入查询字符串。(该参数可选)
                    允许传入的值参考:
                        collections.abc.Mapping : dict, aiohttp.MultiDict 或 aiohttp.MultiDictProxy
                        collections.abc.Iterable: tuple 或 list
                        已被url编码好的str(aiohttp不能对str进行url编码)
                * data – 放在请求体中的数据，接受字典(dcit), 字节或类文件对象。(该参数可选)
                * json – 任何可被json解码的python对象(改参数可选)。注: json不能与data参数同时使用。
                * headers (字典) – 发送请求时携带的HTTP头信息。(该参数可选)
                * skip_auto_headers – 跳过会自动生成的headers信息。 
                        aiohttp会自动生成诸如User-Agent或Content-Type的信息(当然如果没有指定的话)。使用skip_auto_headers参数来跳过。
                        接受str或istr。(该参数可选)
                * auth(aiohttp.BasicAuth) – 用于表示HTTP基础认证的对象。(该参数可选)
                * allow_redirects(布尔类型) – 如果设为False，则不允许重定向。默认是True。(该参数可选)
                * compress (布尔类型) – 如果请求内容要进行deflate编码压缩可以设为True。如果设置了Content-Encoding和Content-Length则不要使用这个参数。默认是None。(该参数可选)
                * chunked (整数) – 允许使用分块编码传输。开发者想使用分块数据流时，用它就没错啦。如果是允许的，aiohttp将会设置为"Transfer-encoding:chunked"，这时不要在设置Transfer-encoding和content-length这两个headers了。默认该参数为None。(该参数可选)
                * expect100(布尔类型) 服务器返回100时将等待响应(返回100表示服务器正在处理数据)。默认是False。(该参数可选)
                * read_until_eof (布尔类型) 如果响应不含Content-Length头信息将会一直读取响应内容直到无内容可读。默认是True。(该参数可选)
                * proxy – 代理URL，接受字符串或URL对象(该参数可选) 
                * proxy_auth(aiohttp.BaicAuth) –  传入表示HTTP代理基础认证的对象。(该参数可选)
                * timeout(整数) – 覆盖会话的超时时间。
                * verify_ssl(布尔类型) – 对HTTPS请求验证SSL证书(默认是验证的)。如果某些网站证书无效的话也可禁用。
                    新增于2.3版本。
                * fringerprint (字节) – 传递所期望的SHA256值(使用DER编码)来验证服务器是否可以成功匹配。对证书固定非常有用。
                     警告: 不赞成使用不安全的MD5和SHA1哈希值。
                     新增于2.3版本。
                * ssl_context (ssl.SLLContext) –
                    ssl上下文(管理器)用于处理HTTPS请求。(该参数可选)
                    ssl_context 用于配置证书授权通道，支持SSL选项等作用。
                    新增于2.3版本。
                * proxy_headers(abc.Mapping) – 如果该参数有提供内容，则会将其做为HTTP headers发送给代理。
                    新增于2.3版本。
        **返回ClientResponse**:
            返回一个**客户端响应(client response)**对象。    
        1.0版本新增的内容: 增加`proxy`和`proxy_auth`参数。添加`timeout`参数。
        1.1版本修改的内容: URLs可以是字符串和URL对象。

    *coroutine async-with **get**(url, \*, allow_redirects=True, \*\*kwargs)*
        该方法会进行**GET**请求。
        kwargs用于指定些request的参数。
        Parameters:
            url - 请求的URL, 字符串或URL对象。
            allow_redirects (布尔类型) - 如果设为False，则不允许重定向。默认是True。
            返回ClientResponse:
                返回一个**客户端响应(client response)**对象。
        1.1版本修改的内容: URLs可以是字符串和URL对象。

    *coroutine async-with post(url, \*, data=None, \*\*kwargs)*
        该方法会执行POST请求。
        kwargs用于指定些request的参数。
        Parameters:
            url - 请求的URL, 字符串或URL对象。
            data  - 接受字典，字节或类文件对象，会作为请求主体发送(可选)。
            返回ClientResponse:
                返回一个**客户端响应(client response)**对象。
        1.1版本修改的内容: URLs可以是字符串和URL对象。

    *coroutine async-with put(url, \*, data=None, \*\*kwargs)*
        该方法会执行PUT请求。
        kwargs用于指定些request的参数。
        Parameters:
            url - 请求的URL, 字符串或URL对象。
            data  - 接受字典，字节或类文件对象，会作为请求主体发送(可选)。
            返回ClientResponse:
                返回一个**客户端响应(client response)**对象。
        1.1版本修改的内容: URLs可以是字符串和URL对象。

    *coroutine async-with delete(url, \*\*kwargs)*
        该方法会执行DELETE请求。
        kwargs用于指定些request的参数。
        Parameters:
            url - 请求的URL, 字符串或URL对象。
        返回ClientResponse:
                返回一个**客户端响应(client response)**对象。
        1.1版本修改的内容: URLs可以是字符串和URL对象。
  
    *coroutine async-with head(url, \*, allow_redirects=False, \*\*kwargs)*
        该方法执行HEAD请求。
        kwargs用于指定些request的参数。
        Parameters:
            url - 请求的URL, 字符串或URL对象。
            allow_redirects (布尔类型) - 如果设为False，则不允许重定向。默认是True。
            返回ClientResponse:
                返回一个**客户端响应(client response)**对象。
        1.1版本修改的内容: URLs可以是字符串和URL对象。      
 
    * coroutine async-with options(url, \*, allow_redirects=True, \*\*kwargs)*
        该方法执行options请求。
        kwargs用于指定些request的参数。
        Parameters:
            url - 请求的URL, 字符串或URL对象。
            allow_redirects (布尔类型) - 如果设为False，则不允许重定向。默认是True。
            返回ClientResponse:
                返回一个**客户端响应(client response)**对象。
        1.1版本修改的内容: URLs可以是字符串和URL对象。     

    *coroutine async-with patch(url, \*, data=None, \*\*kwargs)*
        该方法执行patch请求。
        kwargs用于指定些request的参数。
        Parameters:
            url - 请求的URL, 字符串或URL对象。
            data  - 接受字典，字节或类文件对象，会作为请求主体发送(可选)。
            返回ClientResponse:
                返回一个**客户端响应(client response)**对象。
        1.1版本修改的内容: URLs可以是字符串和URL对象。

    *coroutine async-with ws_connect(url, \*, protocols=(), timeout=10.0, receive_timeout=None, auth=None, autoclose=True, autoping=True, heartbeat=None, origin=None, proxy=None, proxy_auth=None, verify_ssl=None, fingerprint=None, ssl_context=None, proxy_headers=None, compress=0)*
        创建一个websocket连接。返回ClientWebSocketResponse对象。

            Parameters:
                * url - Websocket服务器url, str或URL对象。
                * protocols(元组) - websocket 协议。
                * timeout(浮点数) - 超过超时时间后关闭websocket。默认是10秒。
                * receive_timeout(浮点数) - 超过超时时间后不在从websocket接受消息。默认是None(无限时间)。
                * auth (aiohttp.BasicAuth) - 表示HTTP基础认证的对象。(该参数可选)
                * autoclose(布尔类型)- 从服务器接受完消息后自动关闭websocket. 如果该参数为False，那么需要手动进行关闭。
                * autoping(布尔类型) - 当从服务器收到ping信息后自动回复pong信息。
                * heartbeat(浮点数) - 每到心跳时间则发送ping信息并等待pong信息想要，如果没有收到pong信息则关闭连接。
                * origin(字符串) - 发送到服务器的origin信息。
                * proxy(字符串) - 代理URL，接受字符串或URL对象(该参数可选)。
                * proxy_auth(aiohttp.BasicAuth) - 表示代理HTTP基础认证的对象。(该参数可选)
                * verify_ssl(布尔类型) - 对HTTPS请求验证SSL证书(默认是验证的)。如果某些网站证书无效的话也可禁用。(该参数可选)
                    新增于2.3版本。
                * fringerprint (字节) – 传递所期望的SHA256值(使用DER编码)来验证服务器是否可以成功匹配。对证书固定非常有用。
                     警告: 不赞成使用不安全的MD5和SHA1哈希值。
                     新增于2.3版本。
                * ssl_context (ssl.SLLContext) –
                    ssl上下文(管理器)用于处理HTTPS请求。(该参数可选)
                    ssl_context 用于配置证书授权通道，支持SSL选项等作用。
                    新增于2.3版本。
                * proxy_headers(abc.Mapping) – 如果该参数有提供内容，则会将其做为HTTP headers发送给代理。
                    新增于2.3版本。
                * compress (整数) - 
                  `！此处有疑问！`
                    原文:
                        ```
                        Enable Per-Message Compress Extension support.
                        0 for disable, 9 to 15 for window bit support. Default value is 0.
                        ```
                    允许支持(Per-Message Compress Extension).
                    0表示不使用，9-15表示支持的窗口位数。默认是0.
                    ```
                    由于没用过websocket这里不是很懂，从源码来看是一个headers中的信息。可自行查看https://tools.ietf.org/html/rfc7692#section-4
                    ```
                    新增于2.3版本。
    *coroutine close()*
        关闭底层连接器。
        释放所有占用的资源。

    detach()
        从会话中分离出连接器但不关闭连接器。(之前说过每个会话拥有某连接器的所有权。)
        会话会切换到关闭状态。

# 基本API。
我们鼓励使用客户端会话(ClientSession)但同时也提供一个可以更简单的发起HTTP请求的协程方法。
基本API对于不需要持久连接(keepaliving), cookies和复杂的连接附件(如SSL证书)的HTTP请求来说是比较好用的。

*coroutine aiohttp.request(method, url, \*, params=None, data=None, json=None, headers=None, cookies=None, auth=None, allow_redirects=True, max_redirects=10, encoding='utf-8', version=HttpVersion(major=1, minor=1), compress=None, chunked=None, expect100=False, connector=None, loop=None, read_until_eof=True)*
    异步的执行HTTP请求。返回一个响应对象(ClientResponse或其衍生对象)

    **(参数)Parameters**:
        * method (字符串) - HTTP方法，接受字符串。
        * url - 请求URL, 接受str或URL对象。
        * params (字典) – 与原URL组合成带有查询条件的新URL。(该参数可选) 
        * data – 放在请求体中的数据，接受字典(dcit), 字节或类文件对象。(该参数可选)
        * json – 任何可被json解码的python对象(改参数可选)。注: json不能与data参数同时使用。
        * headers (字典) – 发送请求时携带的HTTP头信息。(该参数可选)
        * cookies (字典) - 请求时携带的cookies。(该参数可选)
        * auth(aiohttp.BasicAuth) – 用于表示HTTP基础认证的对象。(该参数可选)
        * allow_redirects(布尔类型) – 如果设为False，则不允许重定向。默认是True。(该参数可选)
        * version (aiohttp.protocols.HttpVersion) - 请求时选用的HTTP版本。(该参数可选)
        * compress (布尔类型) – 如果请求内容要进行deflate编码压缩可以设为True。如果设置了Content-Encoding和Content-Length则不要使用这个参数。默认是None。(该参数可选)
        * chunked (整数) – 允许使用分块编码传输。开发者想使用分块数据流时，用它就没错啦。如果是允许的，aiohttp将会设置为"Transfer-encoding:chunked"，这时不要在设置Transfer-encoding和content-length这两个headers了。默认该参数为None。(该参数可选)
        * expect100(布尔类型) 服务器返回100时将等待响应(返回100表示服务器正在处理数据)。默认是False。(该参数可选)
        * connector(aiohttp.connector.BaseConnector) - 接受BaseConnector的子类实例用于支持连接池。
        * read_until_eof (布尔类型) 如果响应不含Content-Length头信息将会一直读取响应内容直到无内容可读。默认是True。(该参数可选)
        * loop - 
            事件循环(event loop)
            用于处理HTTP请求。如果参数为None，将会用asyncio.get_event_loop()获取。
            2.0版本后不赞成使用。
返回ClientResponse:
    返回一个**客户端响应**对象。
使用案例:
```
import aiohttp

async def fetch():
    async with aiohttp.request('GET', 'http://python.org/') as resp:
        assert resp.status == 200
        print(await resp.text())
```
1.1版本修改的内容: URLs可以是字符串和URL对象。

# 连接器
连接器用于支持aiohttp客户端API传输数据。
这俩是标准连接器:

1. TCPConnector 用于使用TCP连接(包括HTTP和HTTPS连接)。
2. UnixConnector 用于使用UNIX连接(大多数情况下都用于测试的目的)。
所有的连接器都应是BaseConnector的子类。
默认情况下所有连接器都支持持久连接(keep-alive)(该行为由force_close参数控制)

# BaseConnector

*class aiohttp.BaseConnector(\*, keepalive_timeout=15, force_close=False, limit=100, limit_per_host=0, enable_cleanup_closed=False, loop=None)*
    所有连接器的基类。

    **(参数)Parameters**: 
        * keepalive_timeout(浮点数) - 释放后到再次使用的超时时间(可选)。禁用keep-alive可以传入0，或使用force_close=True。
        * limit(整数) - 并发连接的总数。如果为None则不做限制。(默认为100)
        * limit_per_host - 向同一个端点并发连接的总数。同一端点是具有相同 (host, port, is_ssl)信息的玩意 x 3! 如果是0则不做限制。(默认为0)
        * force_close(布尔类型) - 连接释放后关闭底层网络套接字。(该参数可选)
        * enable_cleanup_closed (布尔类型) - 一些SSL服务器可能会没有正确的完成SSL关闭过程，这种时候asyncio会泄露SSL连接。如果设置为True，aiohttp会在两秒后额外执行一次停止。此功能默认不开启。
        * loop - 
            事件循环(event loop)
            用于处理HTTP请求。如果参数为None，将会用asyncio.get_event_loop()获取。
            2.0版本后不赞成使用。
    closed 
        只读属性，如果连接器关闭则返回True。

    force_close 
        只读属性，如果连接释放后关闭网络套接字则返回True否则返回False。
        新增于0.16版本。

    limit 
        并发连接的总数。如果是0则不做限制。默认是100.

    limit_per_host
        向同一个端点并发连接的总数。同一端点是具有相同 (host, port, is_ssl)信息的玩意 x 3! 如果是None则不做限制。(默认为0)
        只读属性。

    close()
        关闭所有打开的连接。
        新增于2.0版本。

    *coroutine connect(request)*
        从连接池中获取一个空闲的连接，如果没有空闲则新建一个连接。
        如果达到限制(limit)的最大上限则挂起直到有连接处于空闲为止。

        **(参数)Parameters**:
            request (aiohttp.client.ClientRequest) - 发起连接的请求对象。
        
        **返回**:   
            返回连接(Connection)对象。

    *coroutine _create_connection(req)*
        建立实际连接的抽象方法，需被子类覆盖。

# TCPConnector

*class aiohttp.TCPConnector(\*, verify_ssl=True, fingerprint=None, use_dns_cache=True, ttl_dns_cache=10, family=0, ssl_context=None, local_addr=None, resolver=None, keepalive_timeout=sentinel, force_close=False, limit=100, limit_per_host=0, enable_cleanup_closed=False, loop=None)*
    用于使用TCP处理HTTP和HTTPS的连接器。
    如果你不知道该用什么连接器传输数据，那就用它吧。
    TCPConnector继承于BaseConnector.
    接受BaseConnector所需的所有参数和几个额外的TCP需要的参数。

    **(参数)Parameters**: 
        * verify_ssl (布尔类型) – 
            对HTTPS请求验证SSL证书(默认是验证的)。如果某些网站证书无效的话也可禁用。(该参数可选)
            2.3版本后不赞成通过ClientSession.get()方法传递该参数。

        * fingerprint (字节码) - 
            传递所期望的SHA256值(使用DER编码)来验证服务器是否可以成功匹配。对证书固定非常有用。
            警告: 不赞成使用不安全的MD5和SHA1哈希值。
            新增于0.16版本。

            2.3版本后不赞成通过ClientSession.get()方法传递该参数。

        * use_dns_cache (布尔类型) - 
            使用内部缓存进行DNS查找，默认为True。
            这个选项可能会加速建立连接的时间，有时也会些副作用。
            新增于0.17版本。
            自1.0版本起该参数默认为True。

        * ttl_dns_cache - 
            查询过的DNS条目的失效时间，None表示永不失效。默认是10秒。
            默认情况下DNS会被永久缓存，一些环境中的一些HOST对应的IP地址会在特定时间后改变。可以使用这个参数来让DNS刷新。
            新增于2.0.8版本。

        * limit (整数) - 并发连接的总数。如果为None则不做限制。(默认为100)
        
        * limit_per_host - 向同一个端点并发连接的总数。同一端点是具有相同 (host, port, is_ssl)信息的玩意 x 3! 如果是0则不做限制。(默认为0)

        * resolver (aiohttp.abc.AbstructResolver) - 传入自定义的解析器实例。默认是aiohttp.DefaultResolver(如果aiodns已经安装并且版本＞1.1则是异步的)。
            自定义解析器可以配置不同的解析域名的方法。
            1.1版本修改的内容: 默认使用aiohttp.ThreadResolver, 异步版本在某些情况下会解析失败。

        * family (整数) - 
            代表TCP套接字成员，默认有IPv4和IPv6.
            IPv4使用的是socket.AF_INET, IPv6使用的是socket.AF_INET6.
            0.18版本修改的内容: 默认是0，代表可接受IPv4和IPv6。可以传入socket.AF_INET或socket.AF_INET6来明确指定只接受某一种类型。

        * ssl_context (ssl.SSLContext) - 
            ssl上下文(管理器)用于处理HTTPS请求。(该参数可选)
            ssl_context 用于配置证书授权通道，支持SSL选项等作用。

        * local_addr (元组) - 
            包含(local_host, local_post)的元组，用于绑定本地socket。
            新增于0.21版本。        

        *force_close (布尔类型) - 连接释放后关闭底层网络套接字。(该参数可选)

        *enable_cleanup_closed(元组)(这里原文应该写错了，应该是布尔类型，不管是之前的文档还是源码都是接受的布尔值。) -  
            一些SSL服务器可能会没有正确的完成SSL关闭过程，这种时候asyncio会泄露SSL连接。如果设置为True，aiohttp会在两秒后额外执行一次停止。此功能默认不开启。

    verify_ssl
        如果返回True则会进行ssl证书检测。
        该属性只读。

    ssl_context
        返回用于https请求的ssl.SSLContext实例，该属性只读。


    family
        TCP套接字成员， 比如socket.AF_INET 或 socket.AF_INET6。
        该属性只读。

    dns_cache
        如果DNS缓存可用的话返回True，否则返回False。
        该属性只读。
        新增于0.17版本。

    cached_hosts
        如果dns缓存可用，则返回已解析的域名缓存。
        该属性只读，返回的类型为types.MappingProxyType。
        新增于0.17版本。

    fingerprint
        返回传入的DER格式证书的MD5,SHA1或SHA256哈希值 ，如果没有的话会返回None.
        该属性只读。
        新增于0.16版本。

    clear_dns_cache(self, host=None, port=None)
        清除内部DNS缓存。
        如果host和port指定了信息会删除指定的这个，否则清除所有的。
        新增于0.17版本。

# UnixConnector
*class aiohttp.UnixConnector(path, *, conn_timeout=None, keepalive_timeout=30, limit=100, force_close=False, loop=None)*
    Unix 套接字连接器.
    使用UnixConnector发送HTTP/HTTPS请求。底层通过UNIX套接字传输。
    UNIX套接字对于测试并快速在同一个主机上的进程间建立连接非常方便。
    UnixConnector继承于BaseConnector。

    使用:
        ```
        conn = UnixConnector(path='/path/to/socket')
        session = ClientSession(connector=conn)
        async with session.get('http://python.org') as resp:
        ```
    接受所有BaseConnector的参数，还有一个额外的参数:
    **(参数)Parameters**: path(字符串) - Unix套接字路径。
    **path**
        返回UNIX套接字路径，该属性只读。


# Connection
*class* aiohttp.Connection
    连接器对象中封装的单个连接。
    终端用户不要手动创建Connection实例，但可调用BaseConnector.connect()来获取Connection实例，这个方法是协程方法。

*   closed
        只读属性，返回布尔值。如果该连接已关闭，释放或从某一连接器分离则返回Ture。

*   loop
        返回处理连接的事件循环。

*   transport
        返回该连接的传输通道。

close()
        关闭连接并强制关闭底层套接字。

release()
        从连接器中将该连接释放。
        底层套接字并未关闭，如果超时时间(默认30秒)过后该连接仍可用则会被重新占用。

detach()
        将底层套接字从连接中分离。
        底层套接字并未关闭， 之后调用close()或release()也不会有空闲连接池对该套接字有其他操作。

# 响应对象(Response object)

*class aiohttp.ClientResponse*
    ClientSession.requests() 及其同类成员的返回对象。
    用户不要创建ClientResponse的实例，它是由调用API获得的返回。

    ClientResponse支持async上下文管理器:

```
resp = await client_session.get(url)
async with resp:
    assert resp.status == 200
```
这样退出后会自动释放。(详情看release()协程方法)
此语法于0.18版本开始支持。

*   version
        返回响应的版本信息，是HttpVersion实例对象。

*   status
        返回响应的HTTP状态码(整数)，比如: 200。

*   reason
        返回响应的HTTP叙述(字符串)，比如"OK"。

*   method
        返回请求的HTTP方法(字符串)。

*   url
        返回请求的URL(URL对象)。

*   connection
        返回处理响应的连接。

*   content
        包含响应主体(StreamReader)的载体流。支持多种读取方法。服务器使用分块传输编码时同样允许分块接受。
        读取时可能会抛出aiohttp.ClientPayloadError，这种情况发生在响应对象在接受所有的数据前就关闭了或有任何传输编码相关的错误如错误的压缩数据所造成的不规则分块编码。
*   cookies
        响应中的HTTP cookies,(由Set-Cookie HTTP头信息设置, 属于SimpleCookie)

*   headers
        返回响应的HTTP头信息，是一个大小写不敏感的并联字典(CIMultiDictProxy)。
*   raw_headers
        返回原始HTTP头信息，未经编码，格式是键值对形式。

*   content_type
        返回Content-Type头信息的内容。该属性只读。

> ###注意
    根据RFC2616，如果没有Content-Type包含其中则它的值为'application/octet-stream'.可以用使用'CONTENT-TYPE' not in resp.headers(raw_headers)来弄清楚服务器的响应是否包含Content-type

*   charset
        返回请求的主体的编码。
        该值来自于Content-Type HTTP头信息。
        返回的值是类似于'utf-8'之类的字符串，如果HTTP头中不含Content-Type或其中没有charset信息则是None。

*   history
        返回包含所有的重定向请求(都是ClientResponse对象，最开始的请求在最前面)的序列，如果没用重定向则返回空序列。

**close()**
        关闭响应和底层连接。
        要关闭持久连接请看release().

**coroutine read()**
        以字节码形式读取所有响应内容。
        如果读取数据时得到一个错误将会关闭底层连接，否则将会释放连接。
        如果不可读则会抛出aiohttp.ClientResponseError错误。
        返回响应内容的字节码。
        参见:
            close(), release().

**coroutine release()**
        一般不需要调用release。当客户端接受完信息时，底层连接将会自动返回到连接池中。如果载体中的内容没有全部读完，连接也会关闭。


**raise_for_status()**
        如果响应的状态码是400或更高则会抛出aiohttp.ClientResponseError
        如果小于400则什么都不会做。

**coroutine text(encoding=None)**
        读取响应内容并返回解码后的信息。
        如果encoding为None则会从Content-Type中获取，如果Content-Type中也没有会用chardet获取。
        如果有cchardet会优先使用cchardet。
        如果读取数据时得到一个错误将会关闭底层连接，否则将会释放连接。
        Parameters: encoding(字符串) - 指定以该编码方式解码内容，None则自动获取编码方式(默认为None)。
        Return 字符串: 解码后的内容。
> ###注意
        如果Content-Type中不含charset信息则会使用cchardet/chardet获取编码。
        这两种方法都会拖慢执行效率。如果知道页面所使用的编码直接指定是比较好的做法:
        ```
        await resp.text('ISO-8859-1')
        ```

**coroutine json(\*, encoding=None, loads=json.loads, content_type='application/json')**
        以JSON格式读取响应内容，解码和解析器可由参数指定。如果数据不能read则会直接结束。
        如果encoding为None，会使用cchardet或chardet获取编码。
        如果响应中的Content-type不能与参数中的content_type的值相匹配则会抛出aiohttp.ContentTypeError错误。可传入None跳过此检查。

        **Parameters:**
*          encoding (字符串) - 传入用于解码内容的编码名，或None自动获取。 
*          loads (可调用对象) - 用于加载JSON数据，默认是json.loads.
*          content_type (字符串) - 传入字符串以查看响应中的content-type是否符合预期，如果不符合则抛出aiohttp.ContentTypeError错误。传入None可跳过该检测，默认是application/json 。
        **Returns:**    
                返回使用loads进行JSON编码后的数据，如果内容为空或内容只含空白则返回None。

**request_info**
        存放有headers和请求URL的nametuple(一种方便存放数据的扩展类，存在于collections模块中。)属于aiohttp.RequestInfo实例。

# ClientWebSocketResponse
    使用协程方法aiohttp.ws_connect()或aiohttp.ClientSession.ws_connect()连接使用websocket的服务器，不要手动来创建ClientWebSocketResponse。

*class aiohttp.ClientWebSocketResponse*
        用于处理客户端websockets的类

    **closed**
        如果close()已经调用过或接受了CLOSE消息则返回True。
        该属性只读。

    **protocol**
        调用start()后选择的websocket协议。
        如果服务器和客户端所选协议不一致则是None。

    **get_extra_info(name, default=None)**
        返回从连接的transport中读取到的额外的信息。

    **exception()**
        如果有错误则返回那个错误，否则返回None。

    **ping(message=b'')**
        向服务器发送PING.
        Parameters: message – 发送PING时携带的消息，类型是由UTF-8编码的字符串或字节码。(可选)

    **coroutine send_str(data)**
        向服务器发送文本消息。
        Parameters: data (字符串) – 要发送的消息.
        Raises: 如果不是字符串会抛出TypeError错误。

    **coroutine send_bytes(data)**
        向服务器发送二进制消息。
        Parameters: data – 要发送的消息。
        Raises: 如果不是字节码，字节数组或memoryview将抛出TypeError错误。

    **coroutine send_json(data, *, dumps=json.dumps)**
        向服务器发送json字符串。
        Parameters: 
            data – 要发送的消息.
            dumps (可调用对象) –任何可接受某个对象并返回JSON字符串的可调用对象。默认是json.dumps()
        Raises: 
            RuntimeError – 如果连接没有启动或已关闭会抛出这个错误。
            ValueError – 如果数据不是可序列化的对象会抛出这个错误。
            TypeError – 如果由dumps调用后返回的不是字符串会抛出这个错误。

    **coroutine close(\*, code=1000, message=b'')**
        用于向服务器发起挥手(CLOSE)信息，请求关闭连接。它会等待服务器响应。这是一个异步方法，所以如果要添加超时时间可以用asyncio.wait()或asyncio.wait_for()包裹。

        Parameters: 
            code (整数) – 关闭状态码。
            message – pong消息携带的信息，类型是由UTF-8编码的字符串或字节码。(可选)

    **coroutine receive()**
            等待服务器传回消息并在接受后将其返回。
            此方法隐式的处理PING, PONG, CLOSE消息。(不会返回这些消息)

            Returns:    WSMessage
    
    **coroutine receive_str()**
            调用receive()并判断该消息是否是文本。

        Return 字符串: 服务器传回的内容。
        Raises: 如果消息是二进制则会抛出TypeError错误。

    **coroutine receive_bytes()**
             调用receive()并判断该消息是否是二进制内容。

        Return 字符串: 服务器传回的内容。
        Raises: 如果消息是文本则会抛出TypeError错误。           
    
    **coroutine receive_json(\*, loads=json.loads)**
        调用receive_str()并尝试将JSON字符串转成Python中的dict(字典)。

        Parameters: 
            loads (可调用对象) – 任何可接受字符串并返回字典的可调用对象。默认是json.loads()

        Return dict:    
            返回处理过的JSON内容。

        Raises: 
            如果消息是二进制则会抛出TypeError错误。
            如果不是JSON消息则会抛出ValueError错误。

# RequestInfo
*class aiohttp.RequestInfo*
        存放请求URL和头信息的namedtuple，使用ClientResponse.request_info属性可访问。

**url**
    请求的URL，是yarl.URL实例对象。

**method**
    请求时使用的HTTP方法，如'GET', 'POST'，是个字符串。

**headers**
    请求时携带的HTTP头信息，是multidict.CIMultiDict 实例对象。

# BasicAuth
*class aiohttp.BasicAuth(login, password='', encoding='latin1')*
        用于协助进行HTTP基础认证的类。

    **Parameters: **
*       login (字符串) - 用户名。
*       password (字符串) – 密码。
*       encoding (字符串) – 编码信息（默认是'latin1'）。
    一般在客户端 API中使用，比如给ClientSession.request()指定auth参数。

    *classmethod decode(auth_header, encoding='latin1')*
            解码HTTP基本认证信息。

        **Parameters:** 
*           auth_header (字符串) – 需要解码的 Authorization 头数据。
*           encoding (字符串) – (可选) 编码信息(默认是‘latin1’)
        **Returns:**   
            返回解码后的认证信息。

    *classmethod from_url(url)*
            从URL中获取用户名和密码。

        **Returns:**   返回BasicAuth，如果认证信息未提供则返回None。
        新增于2.3版本。
    *encode()*
        将认证信息编码为合适的 Authorization头数据。

        **Returns:**  返回编码后的认证信息。

# CookieJar
*class aiohttp.CookieJar(\*, unsafe=False, loop=None)*
        cookie jar实例对象可用在ClientSession.cookie_jar中。
        jar对象包含用来存储内部cookie数据的Morsel组件。
        可查看保存的cookies数量:
        ```
        len(session.cookie_jar)
        ```
        也可以被迭代:
        ```
        for cookie in session.cookie_jar:
            print(cookie.key)
            print(cookie["domain"])
        ```
        该类提供collections.abc.Iterable, collections.abc.Sized 和 aiohttp.AbstractCookieJar中的方法接口。
        提供符合RFC 6265规定的cookie存储。
        **Parameters:** 
*               unsafe (布尔类型) – (可选)是否可从IP(的HTTP请求)中接受cookies。
*               loop (布尔类型) – 一个事件循环实例。请看aiohttp.abc.AbstractCookieJar。2.0版本后不赞成使用。
    

*update_cookies(cookies, response_url=None)*
        从服务器返回的Set-Cookie信息中更新cookies。

        **Parameters: **
*           cookies – 需要collections.abc.Mapping对象(如dict, SimpleCookie) 或包含cookies的可迭代键值对对象。 
*           response_url (字符串) – cookies所属的URL，如果要共享cookies则不要填。标准的cookies应是与服务器URL成对出现，只在向该服务器请求时被发送出去，如果是共享的则会发送给所有的服务器。

**filter_cookies(request_url)**
        返回与request_url相匹配的cookies，和能给这个URL携带的cookie(一般是设置为None也就是共享的cookie)。

        **Parameters:** response_url (字符串) – 需要筛选的URL。
        **Returns:**    返回带有给定URL cookies的http.cookies.SimpleCookie。

**save(file_path)**
        以pickle形式将cookies信息写入指定路径。

        **Parameters:** file_path –  要写入的路径。字符串或pathlib.Path实例对象。

**load(file_path)**
        从给定路径读取被pickle的cookies信息。

        **Parameters:** file_path – 要导入的路径, 字符串或pathlib.Path实例对象。

*class aiohttp.DummyCookieJar(\*, loop=None)*
        假人cookie jar，用于忽略cookie。
        在一些情况下是很有用的: 比如写爬虫时不需要保存cookies信息，只要一直下载内容就好了。
        将它的实例对象传入即可使用:
        ```
        jar = aiohttp.DummyCookieJar()
        session = aiohttp.ClientSession(cookie_jar=DummyCookieJar())
        ```

# Client exceptions
异常等级制度在2.0版本有较大修改。aiohttp只定义连接处理部分和服务器没有正确响应的异常。一些由开发者引起的错误将使用python标准异常如ValueError，TypeError。
读取响应内容可能会抛出ClientPayloadError异常。该异常表示载体编码时有些问题。比如有无效的压缩信息，不符合分块编码要求的分块内容或内容与content-length指定的大小不一致。
所有的异常都可当做aiohttp模块使用。

*exception aiohttp.ClientError*
    所有客户端异常的基类。
    继承于Exception。

*class aiohttp.ClientPayloadError*
        该异常只会因读取响应内容时存在下列错误而抛出:
            1. 存在无效压缩信息。
            2. 错误的分块编码。
            3. 与Conent-Length不匹配的内容。
        继承于ClientError.

*exception aiohttp.InvalidURL*
        不合理的URL会抛出此异常。比如不含域名部分的URL。
        继承于ClientError和ValueError。

        **url**
            返回那个无效的URL, 是yarl.URL的实例对象。

# Response errors

*exception aiohttp.ClientResponseError*
        这个异常有时会在我们从服务器得到响应后抛出。
        继承于ClientError

        **request_info**
            该属性是RequestInfo实例对象，包含请求信息。

        **code**
            该属性是响应的HTTP状态码。如200.

        **message**
            该属性是响应消息。如"OK"

        **headers**
            该属性是响应头信息。数据类型是列表。

        **history**
            该属性存储失败的响应内容，如果该选项可用的话，否则是个空元组。
            元组中的ClientResponse对象用于处理重定向响应。

*class aiohttp.WSServerHandshakeError*
    Web socket 服务器响应异常。
    继承于ClientResponseError

*class aiohttp.WSServerHandshakeError*
    Web socket 服务器响应异常。
    继承于ClientResponseError
    
*class aiohttp.ContentTypeError*
    无效的content type。
    继承于ClientResponseError
    新增于2.3版本。

# Connection errors
*class aiohttp.ClientConnectionError*
        该类异常与底层连接的问题相关。
        继承于ClientError。

*class aiohttp.ClientOSError*
        连接错误的子集，与OSError属同类异常。
        继承于ClientConnectionError和OSError.

*class aiohttp.ClientConnectorError*
        连接器相关异常。
        继承于ClientOSError

*class aiohttp.ClientProxyConnectionError*
        继承于ClientConnectonError。(由源码知继承于ClientConnectorError原文写错了。)

*class aiohttp.ServerConnectionError*
        继承于ClientConnectonError。(由源码知继承于ClientConnectionError原文写错了。)

*class aiohttp.ClientSSLError*
        继承于ClientConnectonError。(由源码知继承于ClientConnectorError原文写错了。)

*class aiohttp.ClientConnectorSSLError*
        用于响应ssl错误。
        继承于ClientSSLError和ssl.SSLError

*class aiohttp.ClientConnectorCertificateError*
        用于响应证书错误。
        继承于 ClientSSLError 和 ssl.CertificateError

*class aiohttp.ServerDisconnectedError*
        服务器无法连接时抛出的错误。
        继承于ServerDisconnectionError。

        **message**
            包含部分已解析的HTTP消息。(可选)

*class aiohttp.ServerTimeoutError*
        进行服务器操作时超时，如 读取超时。
        继承于ServerConnectionError 和 asyncio.TimeoutError。

*class aiohttp.ServerFingerprintMismatch*
        无法与服务器指纹相匹配时的错误。
        继承于ServerDisconnectionError。

异常等级图:
* ClientError

**      ClientResponseError

***         ContentTypeError
***         WSServerHandshakeError
***         ClientHttpProxyError
**      ClientConnectionError

***         ClientOSError

****            ClientConnectorError

*****                   ClientSSLError
******                      ClientConnectorCertificateError
******                      ClientConnectorSSLError
*****                   ClientProxyConnectionError
****            ServerConnectionError

*****                   ServerDisconnectedError
*****                   ServerTimeoutError
****            ServerFingerprintMismatch

**     ClientPayloadError

**      InvalidURL