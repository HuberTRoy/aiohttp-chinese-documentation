# 服务器端参考
## 请求(Request)和基础请求(BaseRequest)
Request 对象中包含所有的HTTP请求信息。       
BaseRequest 用在低级服务器中（低级服务器没有应用，路由，信号和中间件）。Request对象拥有Request.app和Request.match_info属性。       
BaseRequest和Reuqest都是类字典对象，以便在中间件和信号处理器中共享数据。      
*class aiohttp.web.**BaseRequest***     
&ensp;&ensp;&ensp; **version**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 发起请求的HTTP版本，该属性只读。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回aiohttp.protocol.HttpVersion实例对象。    
&ensp;&ensp;&ensp; **method**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 发起请求的HTTP方法，该属性只读。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回值为大写字符串，如"GET" ，"POST"，"PUT"。      
&ensp;&ensp;&ensp; **url**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 包含资源的绝对路径的URL实例对象。      
### 注意
    如果是不合适的请求（如没有HOST HTTP头信息）则是不可用的。
&ensp;&ensp;&ensp; **rel_url**     
&ensp;&ensp;&ensp; 包含资源的相对路径的URL实例对象。   
&ensp;&ensp;&ensp; 与 .url.relative()相同。     
&ensp;&ensp;&ensp; **scheme**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示请求中的协议（scheme）部分。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果处理方式是SSL则为"https"，否则是"http"。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性的值可能会被 clone()方法覆盖。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为str。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 2.3版本时更改内容: Forwarded 和 X-Forwarded-Proto不在被使用。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 调用 .clone(scheme=new_scheme)来设置一个新的值。   
### 扩展
    基于代理部署服务器

&ensp;&ensp;&ensp; **secure**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 设置request.url.scheme == 'https'的简写。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，True或者False。    
&ensp;&ensp;&ensp; **forwarded**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 包含了所有已编译过的Forwarded头信息的元组。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 尽可能做到符合RFC 7239规定:     
1. 为每个Forwarded域值添加一个不可变的字典。字典内的元素等同于Forwarded域值中的数据，该数据是客户端首次遇到的代理时所添加的值。随后的项是客户端后来遇到的代理所添加的值。   
2. 检查每个值是否符合RFC 7239#section-4中的语法规定：令牌（token）或已编译字符串（quoted-string）。
3. 对已编译对（quoted-pairs）进行un-escape解码。
4. 根据RFC 7239#section-6规定，将不验证‘by’和‘for’的内容。
5. 不验证host内容（Host ABNF）。
6. 对于有效的URI协议名不验证其协议内容。

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回包含一个或多个MappingProxy对象的元组。     
&ensp;&ensp;&ensp; **host**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请求中的主机（Host）名，以此顺序解析：
* 被clone()方法的值所覆盖。
* Host HTTP头信息。
* socket.gtfqdn()
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为str。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 2.3版本时更改内容: Forwarded 和 X-Forwarded-Proto不在被使用。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 调用 .clone(host=new_host)来设置一个新的值。    

### 扩展
    基于代理部署服务器
&ensp;&ensp;&ensp; **remote**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 初始化HTTP请求的IP地址。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 按以下顺序解析：
* 被clone()方法的值所覆盖。
* 已打开的socket的Peer的值。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为str。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 调用 .clone(remote=new_remote)来设置一个新的值。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 2.3版本。    

### 扩展
    基于代理部署服务器
&ensp;&ensp;&ensp; **path_qs**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 包含路径信息和查询字符串的URL（/app/blog?id=10）。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为str。     

&ensp;&ensp;&ensp; **path**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 包含路径信息但不含有主机（host）或协议（scheme）的URL（/app/blog）。路径是URL解码（URL-unquoted）后的。获取原始路径信息请看 raw_path。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为str。     

&ensp;&ensp;&ensp; **raw_path**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 包含原始路径信息但不含有主机（host）或协议（scheme）的URL，该路径可以被编码并且可能包含无效的URL字符，如`/my%2Fpath%7Cwith%21some%25strange%24characters`。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 获取解码的URL请看上面那个。    

&ensp;&ensp;&ensp; **query**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 带有查询字符串的并联字典。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  该属性只读，类型为MultiDictProxy。      

&ensp;&ensp;&ensp; **query_string**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; URL中的查询字符串，如 `id=10`。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为str。      

&ensp;&ensp;&ensp; **headers**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 带有所有头信息的并联字典对象，大小写不敏感。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为 CIMuliDictProxy。     

&ensp;&ensp;&ensp; **raw_headers**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 响应中的头信息，字符未经转换，是一个键值对（key, value）序列。     

&ensp;&ensp;&ensp; **keep_alive**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果允许与HTTP客户端进行`keep-alive`连接并且协议版本也支持的话则为True，否则是False。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为Bool。     

&ensp;&ensp;&ensp; **transport**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于处理请求的传输端口，该属性只读。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性可以被用在获取客户端 peer的IP地址时。    
```
peername = request.transport.get_extra_info('peername')
if peername is not None:
    host, port = peername
```

&ensp;&ensp;&ensp; **loop**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于进行HTTP请求处理的事件循环。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为 asyncio.AbstractEventLoop。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于2.3版本。      

&ensp;&ensp;&ensp; **cookies**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 包含所有请求的cookies信息的并联字典。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为 MultiDictProxy。       

&ensp;&ensp;&ensp; **content**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; StreamReader实例对象，用于读取请求的主体（BODY）的输入流。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读。      

&ensp;&ensp;&ensp; **body_exists**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果请求有HTTP主体（BODY）的话则为True，否则为False。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为Bool。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于2.3版本。      

&ensp;&ensp;&ensp; **can_read_body**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果请求的HTTP主体（BODY）可以被读取则为True，否则为Falase。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为Bool。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于2.3版本。      

&ensp;&ensp;&ensp; **has_body**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果请求的HTTP主体（BODY）可以被读取则为True，否则为Falase。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性只读，类型为Bool。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 2.3版本后不赞成使用: 请使用`can_read_body()`代替。     

&ensp;&ensp;&ensp; **content_type**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回Content-Type头信息的内容，该属性只读。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 类型是str，如'text/html'。      

### 注意 
    如果无Content-Type头信息，根据RFC 2616，返回的值为 'application/octet-stream'

&ensp;&ensp;&ensp; **charset**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请求主体（BODY）使用的编码，该属性只读。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该值由Content-Type头信息解析而来。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回值为字符串，如'utf-8'如果没有则为None。     

&ensp;&ensp;&ensp; **content_length**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请求主体（BODY）的长度，该属性只读。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性由Content-Length解析而来。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回值为整数，如果没有Content-Length信息则为None。       

&ensp;&ensp;&ensp; **http_range**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回Range HTTP头信息的内容，该属性只读。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回值为切片（slice）对象，包含开头（.start）的值但不包含结尾（.stop）的值，步长（.step）为1。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 有以下两种使用方式：
1. 属性方式（假设我们设置了左右两端，开放的边界更加复杂一些）：
```
rng = request.http_range
with open(filename, 'rb') as f:
    f.seek(rng.start)
    return f.read(rng.stop-rng.start)
```
2. 切片方式：
```
return buffer[request.http_range]
```

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  新增于 1.2版本。     

&ensp;&ensp;&ensp; **if_modified_since**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回由If-Modified-Since头信息指定的日期值。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 类型为 datetime.datetime，如果没有If-Modified-Since值或是一个无效的HTTP日期则为None。   

&ensp;&ensp;&ensp; **clone(\*, method=..., rel_url=..., headers=...)**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 克隆自己，并将相应的值替换。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 创建并返回一个新Request实例对象。如果没有传递任何参数，将会复制一份一模一样的。如果没有传递某一个参数，那个值与原值一样。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：
*  method (str) - http方法。     
*  rel_url - 使用的url，str或URL对象。
*  headers - CIMuliDictProxy对象或其他兼容头信息的容器。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **返回**：
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回克隆的Request对象。    

&ensp;&ensp;&ensp; *coroutine read()*      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 读取请求主体，返回带有主体内容的bytes对象。     

### 注意
    该方法在内部存储已读信息，之后的调用会返回同样的值。    

&ensp;&ensp;&ensp; *coroutine text()*       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 读取请求主体，使用charset所返回的编码解码如果MIME-type并未指定编码信息则使用UTF-8解码。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回带有主体内容的str对象。        

### 注意
    该方法在内部存储已读信息，之后的调用会返回同样的值。

&ensp;&ensp;&ensp; *coroutine json(\*, loads=json.loads)*    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 读取请求主体，以JSON形式返回。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 此方法的实现是这样的：
```
async def json(self, *, loads=json.loads):
    body = await self.text()
    return loads(body)
```
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**: loads(callable) - 任何接受str并返回JSON内容的可调用对象（默认是json.loads()）。   

### 注意
    该方法在内部存储已读信息，之后的调用会返回同样的值。
&ensp;&ensp;&ensp; *coroutine multipart(\*, reader=aiohttp.multipart.MultipartReader)*      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回一个用于处理即将到来的multipart请求的aiohttp.multipart.MultipartReader对象。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 此方法的实现是这样的：
```
async def multipart(self, *, reader=aiohttp.multipart.MultipartReader):
    return reader(self.headers, self._payload)
```
此方法是协程方法的原因是为了与其他可能的reader方法（协程方式的reader）保持一致。    

### 警告
    该方法并不在内部存储已读信息。也就是说你读完一次之后不能在用它读第二次了。

### 扩展
    使用Multipart。

&ensp;&ensp;&ensp; *coroutine post()*     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个从请求主体中读取POST参数的协程方法。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回带有已解析后的数据的MultiDictProxy实例对象。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果HTTP方法不是 POST, PUT, PATCH, TRACE 或 DELETE，或者content_type非空，或者存在application/x-www-form-urlencoded，multipart/form-data，将会返回空的并联字典（multidict）。

### 注意 
    该方法在内部存储已读信息，之后的调用会返回同样的值。

&ensp;&ensp;&ensp; *coroutine release()*      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于释放请求。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 未读的HTTP主体将会被清空。    

### 注意
    用户代码中不应存在调用release()，aiohttp.web会在内部自行处理。     


*class aiohttp.web.Request*
&ensp;&ensp;&ensp; 在web处理器中接受请求信息的Request类。      
&ensp;&ensp;&ensp; 所有的处理器的第一个参数都要接受Request类的实例对象。      
&ensp;&ensp;&ensp; 该类派生于BaseRequest，支持父类中所有的方法和属性。还有几个额外的：
&ensp;&ensp;&ensp; **match_info**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回AbstractMatchInfo实例对象，内容是路由解析的结果，该属性只读。     

### 注意
    属性的具体类型由路由决定，如果app.router是UrlDispatcher,则该属性包含的是UrlMappingMatchInfo实例对象。

&ensp;&ensp;&ensp; **app**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回一个用于调用请求处理器的应用（Application）实例对象。


### 注意
    用户不要手动创建Request实例对象，aiohttp.web会自动完成这些操作。但你可以使用clone()方法来进行一些修改，比如路径和方法之类的修改。

## 响应类
目前为止，aiohttp.web提供三个不同的响应类: `StreamResponse`, `Response`和`FileResponse`。      
通常你要使用的是第二个也就是`Response`。`StreamResponse`用于流数据，`Response`会将HTTP主体保存在一个属性中，以正确的Content-Length头信息发送自己的内容。       
为了设计考虑，`Response`的父类是`StreamResponse`。      
如果请求支持keep-alive的话，响应也是同样支持的，无需其他操作。    
当然，你可以使用`force_close()`来禁用keep-alive。     
从web处理器中进行响应通常的做法是返回一个`Response`实例对象:
```
def handler(request):
    return Response("All right!")

```

### StreamResponse
*class aiohttp.web.StreamResponse(\*, status=200, reason=None)*      
### 注:
```
源码中参数多了一个headers。
```
&ensp;&ensp;&ensp; 用于HTTP响应处理的基础类。    
&ensp;&ensp;&ensp; 提供几个如设置HTTP响应头，cookies，响应状态码和写入HTTP响应主体等方法。      
&ensp;&ensp;&ensp; 你需要知道关于响应的最重要的事是——有限状态机。       
&ensp;&ensp;&ensp; 也就是说你只能在调用`prepare()`之前操作头信息，cookies和状态码。     
&ensp;&ensp;&ensp; 一旦你调用了`prepare()`，之后任何有关操作都会导致抛出`RuntimeError`异常。    
&ensp;&ensp;&ensp; 在调用`write_eof()`之后任何`write()`操作也是同样禁止的。    
&ensp;&ensp;&ensp; **参数**：      
* status (int) - HTTP状态码，默认200。      
* reason (int) - HTTP原因（reason）。如果该参数为None，则会根据状态码参数的值进行计算。其他情况请传入用于说明状态码的字符串。     
&ensp;&ensp;&ensp;**prepared**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;如果`prepare()`已被调用过，则返回True否则返回False。该属性只读。新增于0.18版本。       

&ensp;&ensp;&ensp; **task**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个承载请求处理的任务。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 在关闭服务器时对于那些需要长时间运行的请求（流内容，长轮询或web-socket）非常有用。 新增于1.2版本。     

&ensp;&ensp;&ensp; **status**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回HTTP响应状态码，默认是200，该属性只读。       

&ensp;&ensp;&ensp; **reason**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回HTTP状态码的说明字符串，该属性只读。 

&ensp;&ensp;&ensp; **set_status(status, reason=None)**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 设置状态码和说明字符串。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果不设置说明字符串则自动根据状态码计算。   

&ensp;&ensp;&ensp; **keep_alive**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 默认复制Request.keep_alive的值，该属性只读。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 可以通过调用`force_close()`来设为False。    

&ensp;&ensp;&ensp; **force_close()**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 禁止keep-alive连接。不过禁止后没有方法再次开启了。   

&ensp;&ensp;&ensp; **compression**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果允许压缩则返回True，否则返回False。该属性只读。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 默认是False。       

&ensp;&ensp;&ensp; **enable_compression(force=None)**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 开启压缩。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; force如果没有设置则使用的压缩编码从请求的`Accept-Encoding`头信息中选择。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果force设置了则不会管`Accept-Encoding`的内容。    

&ensp;&ensp;&ensp; **chunked**        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 指代是否使用了分块编码，该属性只读。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 可以通过调用`enable_chunked_encoding()`开启分块编码。      

&ensp;&ensp;&ensp; **enable_chunked_encoding()**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 允许响应使用分块编码。 开启之后没有方法关闭它。允许之后，每次`write()`都会编码进分块中。

### 警告
    分块编码只能在HTTP/1.1中使用。    
    content_length和分块编码是相互冲突的。

&ensp;&ensp;&ensp; **headers**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 携带HTTP头信息的CIMultiDict实例对象。   

&ensp;&ensp;&ensp; **cookies**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 携带cookies信息的http.cookies.SimpleCookie实例对象。      

### 警告
    直接使用Set-Cookie头信息设置cookie信息会被显式设置cookie操作所覆盖。    
    我们推荐使用cookies，set_cookie()，del_cookie()进行cookie相关操作。    

&ensp;&ensp;&ensp; **set_cookie(name, value, \*, path='/', expires=None, domain=None, max_age=None, secure=None, httponly=None, version=None)**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 方便设置cookies，允许指定一些额外的属性，如max_age等。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：
* name (str) - cookie名称。   
* value (str) - cookie值（如果是其他类型的话会尝试转换为str类型）
* expires - 过期时间（可选）。
* domain (str) - cookie主域（可选）。
* max_age (int) - 定义cookie的生命时长，以秒为单位。该参数为非负整数。在经过这些秒后，客户端会抛弃该cookie。如果设置为0则表示立即抛弃。     
* path (str) - 设置该cookie应用在哪个路径上（可选，默认是'/'）。
* secure (bool) - 该属性（没有任何值）会让用户代理使用安全协议。用户代理（或许会在用户控制之下）需要决定安全等级，在适当的时候考虑使用“安全”cookie。是不是要使用“安全”要考虑从服务器到用户代理的这段历程，“安全”协议的目的是保证会话在安全情况下进行（可选）。
* httponly (bool) - 如果要设置为HTTP only则为True（可选）。   
* version (int) - 一个十进制数，表示使用哪个版本的cookie管理（可选，默认为1）。

### 警告
    在HTTP 1.1版本中，expires已不再赞成使用，请使用更简单的 max-age来设置，但IE6,7,8并不支持max-age。

&ensp;&ensp;&ensp; **del_cookie(name, \*, path='/'. domain=None)**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 删除某cookie。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：
* name (str) - cookie名称。 
* domain (str) - cookie主域（可选）。
* path (str) - 该cookie的路径（可选，默认是'/'）。
&ensp;&ensp;&ensp; 1.0版本修改内容: 修复对IE11以下版本的过期时间支持。    

&ensp;&ensp;&ensp; **content_length**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 响应中的Content-Length头信息。    

&ensp;&ensp;&ensp; *content_type***     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 响应中Content-Type中设置的内容部分。       

&ensp;&ensp;&ensp; **charset**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 响应中Content-Type中设置的编码部分。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该值会在调用时被转成小写字符。 

&ensp;&ensp;&ensp; **last_modified**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 响应中的Last-Modified头信息。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该属性接受原始字符串，datetime.datetime对象，Unix时间戳（整数或浮点数），None则表示不设置。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 注: 源代码中该方法接受一个参数，文档中并没有标明。下面是源代码:
```
    @last_modified.setter
    def last_modified(self, value):
        if value is None:
            self.headers.pop(hdrs.LAST_MODIFIED, None)
        elif isinstance(value, (int, float)):
            self.headers[hdrs.LAST_MODIFIED] = time.strftime(
                "%a, %d %b %Y %H:%M:%S GMT", time.gmtime(math.ceil(value)))
        elif isinstance(value, datetime.datetime):
            self.headers[hdrs.LAST_MODIFIED] = time.strftime(
                "%a, %d %b %Y %H:%M:%S GMT", value.utctimetuple())
        elif isinstance(value, str):
            self.headers[hdrs.LAST_MODIFIED] = value
```

&ensp;&ensp;&ensp; **tcp_cork**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果该属性为True则把底层传输端口设置为TCP_CORK(Linux)或TCP_NOPUSH(FreeBSD和MacOSX)。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 使用set_tcp_cork()来为该属性设置新值。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 默认为False。     

&ensp;&ensp;&ensp; **set_tcp_cork(value)**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 设置tcp_cork的值。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果为True则会清除tcp_nodelay。   

&ensp;&ensp;&ensp; **tcp_nodelay**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果该属性为True则把底层传输端口设置为TCP_NODELAY。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 使用set_tcp_nodelay()来为该属性设置新值。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 默认是True。     

&ensp;&ensp;&ensp; **set_tcp_nodelay(value)**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 设置tcp_nodelay的值。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果为True则会清除tcp_cork。     

&ensp;&ensp;&ensp; *corotine prepare(request)*     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**： request (aiohttp.web.Request) - HTTP请求对象，要响应的对象。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 会发送HTTP头信息出去。 在调用了该方法之后你就不要在修改任何头信息的数据了。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法同时会调用on_response_prepare信号所连接的处理器。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于0.18版本。    

&ensp;&ensp;&ensp; **write(data)**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 发送byte-ish数据，该数据作为响应主体的一部分。 
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 需要在此之前调用`prepare()`。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果data不是字节（bytes），字节数组（bytearray）或内存查看对象（memoryview）则会抛出`TypeError`异常。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果`prepare()`没有调用则会抛出`RuntimeError`异常。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果`write_eof()`已经被调用则会抛出`RuntimeError`异常。

&ensp;&ensp;&ensp; *coroutine drain()*     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法可以让底层传输端口的写缓存器有机会被刷新。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用在写操作时：
```
resp.write(data)
await resp.drain()
```
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 调用 `drain()`能让事件循环安排写和刷新缓存器的操作。尤其是在写了一个很大的数据时，调用（其他）write()时协程不会被启动。

&ensp;&ensp;&ensp; *coroutine write_eof()*      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个可以作为HTTP响应结束标志的协程方法。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果需要的话，内部会在完成请求处理后调用这个方法。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 调用`write_eof()`后，任何对响应对象的操作都是禁止的。   

### Response

*class aiohttp.web.Response(\*, status=200, headers=None, content_type=None, charset=None, body=None, text=None)*     
### 注：
```
参数方面文档中与源码不符，源码中如下:
Response(*, body=None, status=200, reason=None, text=None, headers=None, content_type=None, charset=None)
```
&ensp;&ensp;&ensp; 最常用的响应类，继承于`StreamResponse`。     
&ensp;&ensp;&ensp; body参数用于设置HTTP响应主体。   
&ensp;&ensp;&ensp; 实际发送body发生在调用write_eof()时。     
&ensp;&ensp;&ensp; **参数**：      
* body (bytes) - 响应主体。      
* status (int) - HTTP状态码，默认200。
* headers (collections.abc.Mapping) - HTTP 头信息，会被添加到响应里。 
* text (str) - 响应主体。
* content_type (str) - 响应的内容类型。如果有传入text参数的话则为`text/plain`，否则是`application/octet-stream`。
* charset (str) - 响应的charset。如果有传入text参数则为`utf-8`，否则是None。   
&ensp;&ensp;&ensp; **body**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 存储响应内容或者叫响应主体，该属性可读可写，类型为bytes。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 设置body会重新计算content_length的值。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 设置body为None会将content_length也一并设置为None，也就是不写入Content-Length HTTP头信息。     

&ensp;&ensp;&ensp; **text**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 存储响应内容（响应主体），该属性可读可写，类型为str。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 设置text会重新计算content_length和body的值。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 设置text为None会将content_length也一并设置为None，也就是不写入Content-Length HTTP头信息。    

### WebSocketResponse
*class aiohttp.web.WebSocketResponse(\*, timeout=10.0, receive_timeout=None, autoclose=True, autoping=True, heartbeat=None, protocols=(), compress=True)*      
&ensp;&ensp;&ensp; 用于进行服务端处理websockets的类。       
&ensp;&ensp;&ensp; 调用`prepare()`后你就不能在使用`write()`方法了，但你仍然可以与websocket客户端通过`send_str()`, `receive()`等方法沟通。     
&ensp;&ensp;&ensp; 新增于 1.3.0版本。     




