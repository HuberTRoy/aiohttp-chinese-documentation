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

