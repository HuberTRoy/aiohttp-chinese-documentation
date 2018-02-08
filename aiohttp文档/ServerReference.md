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
&ensp;&ensp;&ensp; 为了支持缓慢的websockets所带来的背压（back-pressure，服务器达到可连接的上限，而这时又有新的客户端请求连接就会出现背压。具体可看<a href="https://www.zhihu.com/question/49618581?from=profile_question_card">知乎中的解释</a>）现象，相关方法都被设计成协程方法。默认写缓存的大小为64K。
&ensp;&ensp;&ensp; **参数**：     
* autoping (bool) - 自动向发送了PING消息的客户端发送PONG消息，以及自动处理客户端的PONG响应。需要注意的是该方法不会自动向客户端发送PING消息，你需要自己调用ping()方法实现。新增于1.3.0版本。       
* heartbeat (float) - 每一次心跳都发送ping消息并等待pong消息，如果没有接收到pong响应则关闭连接。     
* receive_timeout (float) - 接收操作的超时时间。默认是None也就是不限时间。      
* compress (float) - 允许每一次消息都使用deflate扩展。传入False禁止此功能。默认是True。（文档中应该写错啦，不应该是float，应该是bool，或者也应该是int（False和True对应0和1））
&ensp;&ensp;&ensp; 新增于 0.19版本的内容: 该类支持`async for`语句来对即将到来的消息进行迭代:
```
ws = web.WebSocketResponse()
await ws.prepare(request)

async for msg in ws:
    print(msg.data)
``` 
&ensp;&ensp;&ensp;  *coroutine prepare(request)*     
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  开启websocket。调用此方法后你就可以使用其他的websockets方法了。    
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  **参数**： request (aiohttp.web.Request) - HTTP 请求对象，要响应的对象。     
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  如果websocket握手失败将抛出`HTTPException`异常。       
&ensp;&ensp;&ensp; &ensp;&ensp;&ensp;  新增于 0.18版本。    

&ensp;&ensp;&ensp;  **can_prepare(request)**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 检测这个请求是否能够开启websocket。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果`can_prepare()`执行成功，则`prepare()`也会执行成功。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**： request (aiohttp.web.Request) - HTTP请求对象，要响应的对象。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回WebSocketReady实例对象。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果成功的话，WebSocketReady.ok为True，WebSocketReady.protocol是websocket的子协议，由客户端传递被服务端所接受（一个由WebSocketResponse构造函数所产生的协议序列）。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果客户端和服务器端的子协议不一致，WebSocketReady.protocol则为None。

### 注意
    该方法不会抛出任何异常。

&ensp;&ensp;&ensp; **closed**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果连接已经关闭或正在关闭则为True。这时CLOSE消息已经从peer中接受到了。该属性只读。     

&ensp;&ensp;&ensp; **close_code**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; peer所发送的关闭代码。如果连接是打开的则为None。该属性只读。      

&ensp;&ensp;&ensp; **protocol**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; WebSocket子协议，调用`start()`后才会选择。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果客户端和服务器端的子协议不一致，则为None。     

&ensp;&ensp;&ensp; **exception()**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回最后发生的异常，或者None。     

&ensp;&ensp;&ensp; **ping(message=b'')**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 向peer发送PING消息。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**： message - 可选ping消息载体，str（会被转换成utf-8编码的bytes）或者bytes。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果连接未开启或已关闭则抛出`RuntimeError`异常。    

&ensp;&ensp;&ensp; **pong(message=b'')**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 发送迷之PONG消息到peer。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：message - 可选pong消息载体，str（会被转换成utf-8编码的bytes）或者bytes。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果连接未开启或已关闭则抛出`RuntimeError`异常。       

&ensp;&ensp;&ensp;*coroutine send_str(data)*     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 向peer发送TEXT(文本)消息。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：data (str) - 要发送的数据。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **可能抛出的异常**：
* RuntimeError - 如果连接未开启或已关闭则抛出。
* TypeError - 如果data不是字符串（str）则抛出。

&ensp;&ensp;&ensp; *coroutine send_bytes(data)*    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 向peer发送BINARY(二进制)消息。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**: data - 要发送的数据。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **可能抛出的异常**：
* RuntimeError - 如果连接未开启或已关闭则抛出。
* TypeError - 如果data不是字节（bytes），字节数组（bytearray）或内存查看对象（memoryview）则会抛出。

&ensp;&ensp;&ensp; *coroutine send_json(data, \*, dumps=json.dumps)*     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 向peer发送JSON字符串。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：
* data - 待发送的数据。    
* dumps(callable) - 任何接受某对象并返回JSON字符串的可调用对象（默认是json.dumps()）。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **可能抛出的异常**： 
* RuntimeError - 如果连接未开启或已关闭则抛出。
* ValueError - 如果data不是序列化对象则会抛出。
* TypeError - 如果由dumps参数所传进的对象处理后的值不是str则会抛出。   

&ensp;&ensp;&ensp; *corotine close(\*, code=1000, message=b'')*       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个初始化关闭握手消息的协程方法。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 不同任务中的close()调用会被保留。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：    
* code(int) - 关闭代码。   
* message - 可选的pong消息载体，str(会被转换为utf-8编码的bytes)或bytes。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **可能抛出的异常**： 
* RuntimeError - 如果连接未开启则抛出。

&ensp;&ensp;&ensp;  *coroutine receive(timeout=None)*      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 等待peer即将发来的数据消息并返回它的协程方法。   
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法在PING，PONG和CLOSE处理时都有调用，但并不返回。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法在内部会执行ping-pong游戏和closing握手。    
### 注意
    该方法只能在请求处理任务中被调用。 

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**： timeout - 接受操作的超时时间。会覆盖响应中receive_timeout属性的值。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回WSMessage实例对象。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果连接未开启则抛出`RuntimeError`异常。     

&ensp;&ensp;&ensp; *coroutine receive_str(\*, timeout=None)*     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 调用`receive()`方法，并判断其消息类型是否为TEXT(文本)。     
### 注意
    该方法只能在请求处理任务中被调用。

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**： timeout - 接受操作的超时时间。会覆盖响应中receive_timeout属性的值。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回peer发来的消息数据。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果消息类型是BINARY则会抛出`TypeError`异常。      

&ensp;&ensp;&ensp; *coroutine receive_bytes(\*, timeout=None)*        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 调用`receive()`方法，并判断其消息类型是否为BINARY(二进制)。      
### 注意 
    该方法只能在请求处理任务中被调用。

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**： timeout - 接受操作的超时时间。会覆盖响应中receive_timeout属性的值。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回peer发来的消息数据。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果消息类型是TEXT则会抛出`TypeError`异常。      

&ensp;&ensp;&ensp; *coroutine receive_json(\*, loads=json.loads,timeout=None)*        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 调用`receive_str()`方法，并将JSON字符串转换为Python字典（dict）。      
### 注意 
    该方法只能在请求处理任务中被调用。

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**： 
* loads (callable) - 任何接受str并返回JSON内容的可调用对象（默认是json.loads()）。
* timeout - 接受操作的超时时间。会覆盖响应中receive_timeout属性的值。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回解析JSON后的dict对象。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **可能抛出的异常**：
* TypeError - 如果消息类型为BINARY则会抛出。
* ValueError - 如果消息数据并不是合法的JSON数据则会抛出。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法新增于0.22版本。

### 扩展
    WebSockets处理。

### WebSocketReady
*class aiohttp.web.WebSocketReady*       
&ensp;&ensp;&ensp; `WebSocketResponse.can_prepare()`所返回的对象。   
&ensp;&ensp;&ensp; 可使用bool类型判断：
```
if not await ws.can_prepare(...):
    cannot_start_websocket()
```

&ensp;&ensp;&ensp; **ok**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果websocket可以建立则返回True，否则是False。     

&ensp;&ensp;&ensp; **protocol**      
&ensp;&ensp;&ensp; 表示websocket选择的子协议，类型为str。      

### json_response
*aiohttp.web.json_response([data, ]\*, text=None, body=None, status=200, reason=None, headers=None, content_type='application/json', dumps=json.dumps)*      
&ensp;&ensp;&ensp; 返回内容为JSON数据（默认由json.dumps()转换），并带有'application/json'信息的响应对象。     

## 应用和路由器
### 应用
应用（Application）是web服务器的代名词。       
要得到完整地可工作例子，你必须创建应用（Application），路由表（Router）并且使用`Server`创建服务器套接字作为协议工厂。*Server*可以使用`Application.make_handler()`来创建。      
应用(Application)中包含一个路由实例对象和一个在应用运行期间被调用的回调列表。       
同时应用(Application)还是一个类字典对象，所以你可以用它作为全局共享数据容器，你可以在处理器中使用`Request.app`来访问它：
```
app = Application()
app['database'] = await aiopg.create_engine(**db_config)

async def handler(request):
    with (await request.app['database']) as conn:
        conn.execute("DELETE * FROM table")
```
尽管它是个类字典对象，你也不能用`Application.copy()`来弄个副本。    

*class aiohttp.web.Application(\*, logger=<default>, router=None, middlewares=(), handler_args=None, client_max_size=1024\*\*2, loop=None, debug=...)*       
&ensp;&ensp;&ensp; 该类继承于dict。  
&ensp;&ensp;&ensp; **参数**: 
* logger - logging.Logger实例对象，用于存储应用程序的日志。默认值为`logging.getLogger("aiohttp.web")`      
* router - aiohttp.abc.AbstractRouter实例对象，如果是None则默认创建`UrlDispatcher`。  
* middlewares - 存放中间件工厂的列表，请看Middlewares一节获取详细信息。
* handler_args - 类字典对象，用于覆盖`Application.make_handler()`中的关键字参数。
* client_max_size - 客户端请求中携带的数据的最大大小。如果POST请求超过这个值，将会抛出`HTTPRequestEntityTooLarge`异常。     
* loop - 事件循环。自2.0版本后不再赞成使用：在冻结阶段Loop会被自动设置。     
* debug - 调试组件。   

&ensp;&ensp;&ensp; **router**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回router实例对象，该属性只读。      

&ensp;&ensp;&ensp; **logger**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回logging.Logger实例对象（用于存储应用程序日志）。      

&ensp;&ensp;&ensp; **loop**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回用于处理HTTP请求的事件循环。     

&ensp;&ensp;&ensp; **debug**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回布尔值，表示debug组件是否开启。    

&ensp;&ensp;&ensp; **on_response_prepare**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个在`StreamResponse.prepare()`执行时发送的信号，触发信号时将请求（request）和响应（response）对象作为参数传递。在某些情况下很有用，比如说，你想为每个响应都添加一个自定义头信息。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 信号处理器需要具有如下特征:
```
async def on_prepare(request, response):
    pass
```

&ensp;&ensp;&ensp; **on_startup**          
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个在应用程序开启时触发的信号。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 我们可以捕获这个信号来做一些后台任务。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 信号处理器需要具有如下特征：
```
async def on_startup(app):
    pass
```
 
### 扩展:
```
后台任务。
```

&ensp;&ensp;&ensp; **on_shutdown**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  一个在应用程序关闭时触发的信号。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 我们可以捕获这个信号来做一些对于需要长时间运行的连接的清理工作（websockets和数据流之类的）。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 信号处理器需要具有如下特征：
```
async def on_shutdown(app):
    pass
```
当然，用户需要弄清楚哪些web处理器还在工作以及怎么才能正确地结束它们。      
我们的建议是将那些需要长时间运行的处理器添加到一个列表中并放在Application中。

&ensp;&ensp;&ensp; **on_cleanup**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个在应用程序执行清理时发送的信号。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 我们可以捕获这个信号来优雅的关闭数据库服务器之类的连接。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 信号处理器需要具有如下特征：
```
async def on_cleanup(app):
    pass
```

&ensp;&ensp;&ensp; **make_handler(loop=None, \*\*kwargs)**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 创建一个处理请求的HTTP协议工厂。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：
* loop - 用于处理HTTP请求的事件循环。如果该参数为None则使用`asyncio.get_event_loop()`来获取默认事件循环。2.0版本后已不再赞成使用。        
* tcp_keepalive (bool) - 是否允许`TCP Keep-Alive`。默认为`True`。
* keepalive_timeout (int) - 关闭`Keep-Alive`连接前所持续的秒数。默认是75（NGINX的默认值）。
* slow_request_timeout - 缓慢请求的超时时间。默认是0。
* logger - 自定义logger对象。默认是`aiohttp.log.server_logger`。
* access_log - 自定义logger对象。默认是`aiohttp.log.access_logger`。
* access_log_class - access_logger的类。默认是`aiohttp.helpers.AccessLogger`。必须是`aiohttp.abc.AbstractAcessLogger`的子类。
* access_log_format (str) - 访问日志的字符串格式。默认是`helpers.AccessLogger.LOG_FORMAT`。
* debug (bool) - 选择调试组件。默认是`False`。为了有效利用`Application.make_handler()`，自1.0版本后不再建议使用该方法中的这个参数。因为`Application`的调试组件应该只有一个。
* max_line_size (int) - 最大标题行（header line）大小。默认是8190。
* max_headers (int) - 最大标题(header)大小。默认是32768。
* max_field_size (int) - 最大标题字段（header field）大小。默认是8190。
* linegering_time (float) - 当延迟关闭开启时，服务器读取以及忽略从客户端传来的额外信息的最大时间。传入0来禁止延迟关闭。
* linegering_timeout (float) - 延迟关闭工作时，等待客户端送达额外信息的超时时间。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;你应该把这个方法所返回的结果作为`protocol_factory`传递给`create_server()`：
```
loop = asyncio.get_event_loop()

app = Application()

# setup route table
# app.router.add_route(...)

await loop.create_server(app.make_handler(),
                         '0.0.0.0', 8080)
```

&ensp;&ensp;&ensp; *coroutine startup()*      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个会与应用程序的请求处理器一起调用的协程方法。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法的目的是调用on_startup信号所连接的处理器。    

&ensp;&ensp;&ensp; *coroutine shutdown()*     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法应该在服务器正在停止且在（要）调用`cleanup()`前调用。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法会调用on_shutdown信号所连接的处理器。     

&ensp;&ensp;&ensp; *coroutine cleanup()*       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法应该在服务器正在停止且在（要）调用`shutdown()`后调用。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法会调用on_cleanup信号所连接的处理器。      

### 注意
    Application对象拥有路由属性，但并不拥有add_route()方法。原因是:我们想支持不同的路由部署方式（甚至基于遍历而不是基于url匹配）。
    由于这个原因，我们有非常细小的AbstractRouter抽象类：这个抽象基类也只有一个AbstractRouter.resolve()协程方法。
    没有添加路由和倒推路由（由路由名来获得URL）的方法。这些已是路由部署的全部细节了（但是说真的，用这个路由你需要在你的应用程序中自己解决这个问题）。

## Server
一个与`create_server()`兼容的协议工厂。
*class aiohttp.web.Server*      
&ensp;&ensp;&ensp; 该类用于创建处理HTTP连接的HTTP协议对象。     

*Server.connections*          
&ensp;&ensp;&ensp; 一个包含当前已开启的连接的列表。      

*aiohttp.web.request_count*          
&ensp;&ensp;&ensp; 已处理请求的总数。       
&ensp;&ensp;&ensp; 新增于 1.0版本。        

*coroutine Server.shutdown(timeout)*        
&ensp;&ensp;&ensp; 一个用于关闭所有已开启连接的协程方法。       

## Router
用于将分发URL到特定的处理器，aiohttp.web使用路由来建立联系。       
路由可以是任何部署了`AbstractRouter`接口的对象。        
aiohttp.web提供的部署方式为`UrlDispatcher`。       
`Application`也使用`UrlDispatcher`作为`router()`的默认返回。      

*class aiohttp.web.UrlDispatcher*        
&ensp;&ensp;&ensp; 最直接地url匹配型路由，同时具有`collections.abc.Mapping`的功能，可以用于访问已命名的路由。       
&ensp;&ensp;&ensp; 在运行`Application`之前，你应该首先调用`add_route()`和`add_static()`来填写下路由表。       
&ensp;&ensp;&ensp; 处理器的查找是迭代方式进行的（先进先出顺序）。首先匹配到的路由会被调用对应处理器。        
&ensp;&ensp;&ensp; 如果在创建路由时你指定了`name`参数，那么这就是一个命名路由了。      
&ensp;&ensp;&ensp; 命名路由可以调用`app.router[name]`获得，也可以用于检测该名字是否在`app.router`中。        

&ensp;&ensp;&ensp; **add_resource(path, \*, name=None)**
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个资源到路由表尾部。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; path可以只是字符串`'/a/b/c'`，也可以带变量`'/a/{var}'`。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：     
* path (str) - 资源路径。     
* name (str) - 资源名，可选。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回创建的资源实例（`PlainResource或DynamicResource`）。      

&ensp;&ensp;&ensp; **add_route(method, path, handler, \*, name=None, expect_handler=None)**        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个处理器到路由表尾部。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; path可以只是字符串`'/a/b/c'`，也可以带变量`'/a/{var}'`。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 注意: 如果处理器是一个普通函数，aiohttp会在内部将其转换为协程函数。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：
* method (str) - 该路由的HTTP方法。应是 'GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD', 'OPTIONS' 其中的一个，或者是'\*'来表示所有方法。该方法大小写不敏感，'get'等于'GET'。             
* paht (str) - 路由路径。需要以斜线('/')开头。
* handler (callable) - 路由处理器。     
* name (str) - 路由名称，可选。
* expect_handler (coroutine) - 异常处理器，可选。

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回`PlainResource或DynamicResource`实例对象。        

&ensp;&ensp;&ensp; **add_routes(routes_table)**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 从路由表（routes_table）中注册路由。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 路由表（routes_table）需要是包含`RouteDef`组件或`RouteTableDef`的列表。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于2.3版本。      

&ensp;&ensp;&ensp; **add_get(path, handler, \*, name=None, allow_head=True, \*\*kwargs)**              
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加`GET`方法路由的快捷方式。等价于调用`add_route(method='GET')`。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果`allow_head`是True（默认），也会添加一个HEAD方法到这个路由上。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果有name，HEAD的路由会被添加一个'-head'后缀。举个栗子：`router.add_get(path, handler, name='route')`会添加两个路由：一个是GET方法名为'route'，另一个是HEAD方法名为'route-head'。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法新增于1.0版本。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 2.0版本新增内容: 添加allow_head参数。

&ensp;&ensp;&ensp; **add_post(path, handler, \*\*kwargs)**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加'POST'方法路由的快捷方式。等价于调用`add_route(method='POST')`。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于1.0版本。      

&ensp;&ensp;&ensp; **add_put(path, handler, \*\*kwargs)**        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加`PUT`方法路由的快捷方式。等价于调用`add_route(method='PUT')`。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 1.0版本。      

&ensp;&ensp;&ensp; **add_patch(path, handler, \*\*kwargs)**        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加`PATCH`方法路由的快捷方式。等价于调用`add_route(method='PATCH')`。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于1.0版本。      

&ensp;&ensp;&ensp; **add_delete(path, handler, \*\*kwargs)**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加`DELETE`方法路由的快捷方式。等价于调用`add_route(method='DELETE')`。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 1.0版本。      

&ensp;&ensp;&ensp; **add_static(prefix, path, \*, name=None, expect_handler=None, chunk_size=256\*1024, response_factory=StreamResponse, show_index=False, follow_symlinks=False, append_version=False)**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个用于返回静态文件的路由和处理器。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 对于获取图片，js和css等文件非常有用。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 在支持它的平台上，使用这个可以让文件发送系统(sendfile system)处理器更高效的转发这类文件。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 在某些情况下，即使平台支持也要避免使用文件发送系统。可以使用环境变量AIOHTTP_NOSENDFILE=1来设置它。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果静态文件的内容是gzip压缩的内容，文件的路径需要+ `.gz`，同样也会应用在响应中。

### 警告：
    add_static()仅用于开发。生存环境中，静态文件功能应由wbe服务器（比如nginx,apache）提供。

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 0.18.0版本修改的内容: 在支持的平台中，转发文件使用文件发送系统（sendfile system）。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 0.19.0版本修改的内容: 可以使用环境变量`AIOHTTP_NOSENDFILE=1`来关闭文件发送系统。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 1.2.0  版本修改的内容: 发送gzip压缩的文件路径后会+`.gz`。        

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：          
* prefix (str) - URL路径前缀，用于加到静态文件中。
* path - 包含静态文件的文件系统中的文件夹，可以是str或`pathlib.Path`。       
* name (str) - 路由名称，可选。
* expect_handler (coroutine) - 异常处理器，可选。 
* chunk_size (int) - 下载文件时单个分块的最大大小，默认是256Kb。增加chunk_size参数的值，比如说1Mb可能会提高下载速度，但同时也会增加内存占用。新增于 0.16版本。         
* response_factory (callable) - 用于制造新响应的工厂，默认是`StreamResponse`，传入的对象应有兼容的API。新增于0.17版本。
* show_index (bool) - 是否允许显示目录索引的标识，默认不允许并且返回`HTTP/403`。
* follow_symlinks (bool) - 是否允许使用符号链接的标识，默认是不允许并返回`HTTP/404`。
* append_version (bool) - 是否给url查询字符串添加文件版本号（hash）的标识，不过当你调用`StaticRoute.url()`和`StaticRoute.url_for()`的时候总会使用默认值。     

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回一个`StaticRoute`实例对象。    

&ensp;&ensp;&ensp; **add_subapp(prefix, subapp)**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 在给定路径前缀下注册一个嵌套子应用。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 以定前缀开始的请求会交由subapp处理。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：
* prefix (str) - 资源的路径前缀。    
* subapp (Application) - 在给定前缀下的嵌套应用。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回`PreFixedSubAppResource`实例对象。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 1.1版本。    

&ensp;&ensp;&ensp; *coroutine resolve(request)*      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回request的`AbstractMatchInfo`实例对象。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法不会抛出任何异常，但是返回的`AbstractMatchInfo`的`http_exception`可能会有`HTTPException`实例对象。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 如果该请求没有与任何注册的路由相匹配，请求处理器会抛出`HTTPNotFound`或`HTTPMethodNotAllowed`的异常。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 中间件就可以捕获这些异常来显示一个漂亮的错误页面。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 内部使用，终端用户几乎不需要调用这个方法。       

### 注意
    该方法使用Request.raw_path来匹配已注册的路由。     

&ensp;&ensp;&ensp; **resources()**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回一个包含所有已注册的资源的`view`。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; `view`是一个允许这样做的对象:
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 从中获取路由表大小：
```
len(app.router.resources())
``` 
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 进行迭代：
```
for resource in app.router.resources():
    print(resource)
```
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 检测某资源是否已经在路由表中：
```
route in app.router.resources()
```
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法新增于 0.21.1。         

&ensp;&ensp;&ensp; **routes()**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回一个包含所有已注册的路由的`view`。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 0.18版本。       

&ensp;&ensp;&ensp; **named_resources()**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回一个包含所有已命名资源的类字典对象types.MappingProxyType的`view`。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; view会将所有已命名资源的名字变成`BaseResource`实例对象。 支持常规类字典操作，除了可变性（它是只读的）。      
```
len(app.router.named_resources())

for name, resource in app.router.named_resources().items():
    print(name, resource)

"name" in app.router.named_resources()

app.router.named_resources()["name"]
```
&ensp;&ensp;&ensp; 该方法新增于 0.21版本。      

&ensp;&ensp;&ensp; **named_routes()**          
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 与`named_resources()`一样，这个要早些，因为0.21版本才有`named_resources()`。         
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法新增于 0.19版本。          
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 0.21版本修改的内容: 该方法变为 named_resources()的副本，所以请用resources代替routes。           
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 自0.21版本后不再赞成使用: 请使用`named_resources`代替`name_routes`。          
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 与资源相同的路由会共用一个资源名称。            

## Resource
默认路由`UrlDispatcher`会同资源一起工作。     
资源是路由表中的一个含有路径，独特的名字和至少有一条路由的组件。     
web处理器会按以下方式进行检索:
1. 从资源中一个个迭代。       
2. 如果某资源与请求的URL相匹配则迭代其包含的路由。        
3. 如果某路由的方法与请求的URL相匹配，该路由的处理器就会作为web处理器进行后续处理。之后检索结束。
4. 否则路由会尝试下一条路由表中的资源。      
5. 如果检索到最后都没有一个 资源/路由对符合请求，那么会返回一个特殊的`AbstractMatchInfo`实例对象，该对象会附带 `AbstractMatchInfo.http_exception = HTTPException`或者说`HTTP 404 Not Found / HTTP 405 Method Not Allowed`状态码。这时所注册的`AbstractMatchInfo.handler`会开始工作。      
用户永远不要随意实例化一个资源类，只需要在用到时把它用`UrlDispatcher.add_resource()`添加即可。        
添加之后可能还需要添加路由: `Resource.add_route()`。       
`UrlDispatcher.add_route()`是下列方式的简化版：
```
router.add_resource(path).add_route(method, handler)
```
有名字的资源被叫做 已命名资源（named resource）。已命名资源存在的主要原因是使用路由名来构建URL时可以传递它到模板引擎中使用：
```
url = app.router['resource_name'].url_for().with_query({'a': 1, 'b': 2})
```

资源类的继承等级：
* AbstractResource
* - Resource
* - - PlainResource
* - - DynamicResource
* - - StaticResource

*class aiohttp.web.AbstractResource*        
&ensp;&ensp;&ensp; 所有资源的基类。       
&ensp;&ensp;&ensp; 继承自`collections.abc.Sized`和`collections.abc.Iterable`。     
&ensp;&ensp;&ensp; `len(resource)`会返回属于该资源的路由总数，也允许进行迭代`for route in resource`。     

&ensp;&ensp;&ensp; **name**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 资源的名称，或None，该属性只读。      
### xxx
&ensp;&ensp;&ensp; *coroutine resolve(method, path)*      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 尝试寻找合适的web处理器（method, path）来解析该资源。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：      
* method (str) - 请求的HTTP方法。
* path (str) - 请求的路径。
&ensp;&ensp;&ensp; 返回（match_info, allowed_methods）。      
&ensp;&ensp;&ensp; allowed_methods 是一个包含资源可接受的HTTP方法的集合。      
&ensp;&ensp;&ensp; 如果请求被解析（resolve）match_info会是`UrlMappingMatchInfo`，如果没有找到任何路由则是None。       

### xxx
&ensp;&ensp;&ensp; **get_info()**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回资源的描述。如`{'path': '/path/to'}`, `{'formatter': '/path/{to}', 'pattern': re.compile(r'^/path/(?P<to>[a-zA-Z][_a-zA-Z0-9]+)$)`。        


&ensp;&ensp;&ensp; **url_for(\*args, \*\*kwargs)**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 构建一个该路由的URL（可附带额外参数）。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; `args`和`kwargs`依赖继承于资源类的可接受参数列表。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回URL实例对象。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 1.1版本。       

&ensp;&ensp;&ensp; **url(\*\*kwargs)**        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 同上。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 1.1版本后不再赞成使用，请使用`url_for()`代替。      

*class aiohttp.web.Resource*       
&ensp;&ensp;&ensp; 新类型资源的基类，继承于`AbstractResource`。      

&ensp;&ensp;&ensp; **add_route(method, handler, \*, expect_handler=None)**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个web处理器到资源中。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：     
* method (str) - 路由可接受的HTTP方法。应是'GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD', 'OPTIONS'中的一个，或传入'\*'表示任何方法。该方法大小写不敏感，'get'=='GET'。对该资源来说，method必须是唯一的。           
* handler (callable) - 路由处理器。    
* expect_handler (coroutine) - 异常处理器，可选。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回`ResourceReoute`实例对象。      

*class aiohttp.web.PlainResource*     
&ensp;&ensp;&ensp; 一个资源类，继承于`Resource`。     
&ensp;&ensp;&ensp; 该类等价于使用普通文本匹配的资源。如'/path/to'。       

&ensp;&ensp;&ensp; **url_for()**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回所属资源的URL。        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 1.1版本。     

*class aiohttp.web.DynamicResource*        
&ensp;&ensp;&ensp; 一个资源类，继承于`Resource`。      
&ensp;&ensp;&ensp; 该类等价于使用变量匹配的资源。如'/path/{to}/{param}'。      

&ensp;&ensp;&ensp; **url_for(\*\*params)**        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回所属资源的URL。  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**： params - 置换动态资源的变量。     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 比如 '/path/{to}/{param}'，调用的时候应该是`resource.url_for(to="val1", param='val2')`。       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 1.1版本。    

*class aiohttp.web.StaticResource*      
&ensp;&ensp;&ensp; 一个资源类，继承于`Resource`。     
&ensp;&ensp;&ensp;等价于使用静态文件的资源。     

&ensp;&ensp;&ensp; **url_for(filename, append_version=None)**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回文件路径并带有资源的前缀。      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; **参数**：  
* filename - 文件名，由静态文件处理器调用。接受str和`pathlib.Path`。 '/prefix/dir/file.txt'由`resource.url_for(filename='dir/file.txt')`生成。
* append_version (bool) - 是否给url查询字符串添加文件版本号（hash）的标识（可以加速缓存）。默认情况（False）下会由构造器产生一个值，当是Ture的时候 `v=FILE_HASH`格式的查询字符串参数会被添加到URL中，False则不做操作。如果文件未找到也不会有任何操作。
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 新增于 1.1版本。   

*class aiohttp.web.PreFixedSubAppResource*       
&ensp;&ensp;&ensp; 一个用于嵌套应用的资源类。该类由`add_subapp`方法返回而来。   
&ensp;&ensp;&ensp; 新增于1.1版本。  

&ensp;&ensp;&ensp; **url_for(\*\*kwargs)**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该方法不可调用，调用会抛出`RuntimeError`异常。      

## Route
路由（Route）具有HTTP方法（通配符'\*'是可以用的），web处理器和异常处理器（可选）。      
每个路由都可属于多个资源。    
路由类等级：
* AbstractRoute
* - ResourceRoute
* - SystemRoute

`ResourceRoute`是一个面向资源的路由，`SystemRoute`面向的是URL解析错误的处理，比如`404 Not Found`和`405 Method Not Allowd`之类的。      

*class aiohttp.web.AbstractRoute*      
&ensp;&ensp;&ensp; `UrlDispatcher`可使用的路由基类。       

&ensp;&ensp;&ensp; **method**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该路由要处理的HTTP方法。'GET', 'POST'之类。     

&ensp;&ensp;&ensp; **handler**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该路由的处理器。      

&ensp;&ensp;&ensp; **name**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 该路由的名称，等价于资源名称中属于该路由的名称。

&ensp;&ensp;&ensp; **resource**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 持有该路由的资源，如果该类是`SystemRoute`则为None。     

&ensp;&ensp;&ensp; **url_for(\*args, \*\*kwargs)**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 返回该路由所能构建的url。（只是个基础方法，意思是功能要自己继承后实现）      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 同时也是`route.resource.url_for(...)`的快捷方式。      

&ensp;&ensp;&ensp; *coroutine handle_expect_header(request)*            
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; `100-continue`的处理器。       

*class aiohttp.web.ResourceRoute*       
&ensp;&ensp;&ensp; 一个用于在资源中处理不同HTTP方法的路由类。       

*class aiohttp.web.SystemRoute*         
&ensp;&ensp;&ensp; 一个用于处理URL错误（404/405之类的）的路由类。            

&ensp;&ensp;&ensp; **status**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; HTTP状态码。     

&ensp;&ensp;&ensp; **reason**        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; HTTP状态码说明。       

## RouteDef     
路由定义，是对还没注册路由的描绘，说明。     
在填写路由表时使用的是包含路由定义的列表时非常有用（Django风格）。        
定义由`get()`或`post()`之类的函数创建，包含定义的列表可以用`UrlDispatcher.add_routes()`添加到路由器上：
```
from aiohttp import web

async def handle_get(request):
    ...


async def handle_post(request):
    ...

app.router.add_routes([web.get('/get', handle_get),
                       web.post('/post', handle_post),
```

*class aiohttp.web.RouteDef*        
&ensp;&ensp;&ensp; 创建一个还没被添加的路由的定义。
&ensp;&ensp;&ensp; **method**     
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; HTTP方法（GET, POST之类），类型为字符串。     

&ensp;&ensp;&ensp; **path**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 资源的路径，比如/path/to，可以是包含`{}`的可变资源，类型为字符串。   

&ensp;&ensp;&ensp; **handler**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 用于处理HTTP请求的协程函数。       

&ensp;&ensp;&ensp; **kwargs**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 包含额外参数的字典。      

&ensp;&ensp;&ensp; 新增于2.3版本。     

*aiohttp.web.get(path, handler, \*, name=None, allow_head=True, expect_handler=None)*      
&ensp;&ensp;&ensp; 返回处理GET请求的`RouteDef`。请看`UrlDispatcher.add_get()`获取参数信息。     
&ensp;&ensp;&ensp; 新增于2.3版本。   

*aiohttp.web.post(path, handler, \*, name=None, expect_handler=None)*      
&ensp;&ensp;&ensp; 返回处理POST请求的 'RouteDef'。请看`UrlDispatcher.add_post()`获取参数信息。      
&ensp;&ensp;&ensp; 新增于2.3版本。 

*aiohttp.web.head(path, handler, \*, name=None, expect_handler=None)*
&ensp;&ensp;&ensp; 返回处理HEAD请求的`RouteDef`。请看`UrlDispatcher.add_head()`获取参数信息。     
&ensp;&ensp;&ensp; 新增于2.3版本。   

*aiohttp.web.put(path, handler, \*, name=None, expect_handler=None)*
&ensp;&ensp;&ensp; 返回处理PUT请求的`RouteDef`。请看`UrlDispatcher.add_put()`获取参数信息。     
&ensp;&ensp;&ensp; 新增于2.3版本。   

*aiohttp.web.patch(path, handler, \*, name=None, expect_handler=None)*
&ensp;&ensp;&ensp; 返回处理PATCH请求的`RouteDef`。请看`UrlDispatcher.add_patch()`获取参数信息。     
&ensp;&ensp;&ensp; 新增于2.3版本。   

*aiohttp.web.delete(path, handler, \*, name=None, expect_handler=None)*
&ensp;&ensp;&ensp; 返回处理DELETE请求的`RouteDef`。请看`UrlDispatcher.add_delete()`获取参数信息。     
&ensp;&ensp;&ensp; 新增于2.3版本。   

### xxx
*aiohttp.web.route(method, path, handler, *, name=None, expect_handler=None)*
&ensp;&ensp;&ensp; 返回处理POST请求的`RouteDef`（文档中应该写错了，这个方法处理什么由method决定）。请看`UrlDispatcher.add_route()`获取参数信息。     
&ensp;&ensp;&ensp; 新增于2.3版本。   

## RouteTableDef
路由表定义（RouteTableDef）用于以装饰器模式描绘路由（Flask风格）：
```
from aiohttp import web

routes = web.RouteTableDef()

@routes.get('/get')
async def handle_get(request):
    ...


@routes.post('/post')
async def handle_post(request):
    ...

app.router.add_routes(routes)
```

*class aiohttp.web.RouteTableDef*      
&ensp;&ensp;&ensp; 包含RouteDef实例的序列（具有abc.collections.Sequence的功能）。       
&ensp;&ensp;&ensp; 除了所有标准列表方法，该类还提供如'get()', 'post()'之类用于添加新路由定义的方法。
&ensp;&ensp;&ensp; *@get(path, \*, allow_head=True, name=None, expect_handler=None)*        
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个GET方法的路由定义。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请看`UrlDispatcher.add_get()`查看更多信息。             

&ensp;&ensp;&ensp; *@post(path, \*, name=None, expect_handler=None)*           
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个POST方法的路由定义。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请看`UrlDispatcher.add_post()`查看更多信息。             

&ensp;&ensp;&ensp; *@head(path, \*, name=None, expect_handler=None)*           
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个HEAD方法的路由定义。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请看`UrlDispatcher.add_head()`查看更多信息。           

&ensp;&ensp;&ensp; *@put(path, \*, name=None, expect_handler=None)*           
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个PUT方法的路由定义。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请看`UrlDispatcher.add_put()`查看更多信息。           

&ensp;&ensp;&ensp; *@patch(path, \*, name=None, expect_handler=None)*           
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个PATCH方法的路由定义。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请看`UrlDispatcher.add_patch()`查看更多信息。           

&ensp;&ensp;&ensp; *@delete(path, \*, name=None, expect_handler=None)*           
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加一个DELETE方法的路由定义。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请看`UrlDispatcher.add_delete()`查看更多信息。           

&ensp;&ensp;&ensp; *@route(method, path, \*, name=None, expect_handler=None)*           
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 添加任意HTTP方法的路由定义。    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 请看`UrlDispatcher.add_route()`查看更多信息。           

&ensp;&ensp;&ensp; 新增于 2.3版本。      

### MatchInfo    
路由匹配之后，web应用会调用任何找到的处理器。       
匹配结果可以在处理器中使用`Request.match_info`属性访问。       
一般情况下，匹配结果可以是任何从`AbstractMatchInfo`继承的对象（如果是默认`UrlDispatcher`路由器则是`UrlMappingMatchInfo`）。          

*class aiohttp.web.UrlMappingMatchInfo*        
&ensp;&ensp;&ensp; 继承自`dict`和`AbstractMatchInfo`。字典内的项由匹配的内容填充，其内容只是与资源相关的信息。      

&ensp;&ensp;&ensp; **expect_handler**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个用于处理 100-continue的协程处理器。     

&ensp;&ensp;&ensp; **handler**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 一个用于处理请求的协程处理器。      

&ensp;&ensp;&ensp; **route**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; url所匹配的路由实例。      

### View      
*class aiohttp.web.View(request)*      
&ensp;&ensp;&ensp; 继承自`AbstractView`。        
&ensp;&ensp;&ensp; 该类是以类为基础的视图的基类。部署时需要继承`View`并且要覆盖处理HTTP请求的方法（get(), post()等）：
```
class MyView(View):

    async def get(self):
        resp = await get_response(self.request)
        return resp

    async def post(self):
        resp = await post_response(self.request)
        return resp

app.router.add_route('*', '/view', MyView)
```
&ensp;&ensp;&ensp; 如果请求的web方法不支持的话，视图会抛出`405 Method Not allowed (HTTPMethodNotAllowed)`。        
&ensp;&ensp;&ensp; **参数**：request - 创建视图处理的Request实例对象。       

&ensp;&ensp;&ensp; **request**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 发送到视图构造器中的请求，该属性只读。         

&ensp;&ensp;&ensp; 可覆盖的协程方法有: `connect(), delete(), get(), head(), options(), patch(), post(), put(), trace()`。        

### 其他工具
*class aiohttp.web.FileField*       
&ensp;&ensp;&ensp; 该类是在Request.POST()有上传文件时所返回的类（作为并联字典的namedtuple实例对象）。          

&ensp;&ensp;&ensp; **name**         
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 字段名。      

&ensp;&ensp;&ensp; **filename**       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 上传时指定的文件名字。      

&ensp;&ensp;&ensp; **file**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 携带上传的文件的内容的 `io.IOBase`实例对象。     

&ensp;&ensp;&ensp; **content_type**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 上传的文件的MIME类型，默认是'text/plain'。       


*aiohttp.web.run_app(app, \*, host=None, port=None, path=None, sock=None, shutdown_timeout=60.0, ssl_context=None, print=print,  backlog=128, access_log_format=None, access_log=aiohttp.log.access_logger, handle_signals=True, loop=None)*       
&ensp;&ensp;&ensp; 用于运行应用程序的函数，会一直运行，直到键盘打断然后执行关闭操作。      
&ensp;&ensp;&ensp; 适合作为基础aiohttp项目使用。生存配置可能需要更复杂些的启动器，但在项目最初阶段，用它就够了。      
&ensp;&ensp;&ensp; 该函数使用 `app.loop`作为事件循环。     
&ensp;&ensp;&ensp; 服务器会监听你所提供的所有主机或Unix域套接字路径。如果没有提供任何主机或路径，或只提供了端口，则TCP服务器会监听 `0.0.0.0`(表示所有主机)。      
&ensp;&ensp;&ensp; 在同一应用程序中将HTTP请求分发到多个主机或路径对于在同一个事件循环中处理请求没有执行效率上的提升。请查阅`Server Deployment`来了解如何进行分发工作以提升效率。      

&ensp;&ensp;&ensp; **参数**：     
* app - 要运行的`Application`实例对象。      
* host (str) - 用于监听的TCP/IP 主机或一组主机序列。如果有提供端口或path没有提供则默认是`0.0.0.0`。     
* port (int) - 用于监听TCP/IP 端口。对于普通HTTP默认是8080，经由SSL的HTTP默认则是8443（需要指定ssl_context参数）。
* path (str) - 作为HTTP服务器，Unix域套接字的文件系统中的路径。绑定多个域套接字可以是一组路径序列。监听Unix域套接字并不是支持所有操作系统的。      
* sock (socket) - 预先存在的套接字（socket）对象，用于接收连接。可以传递一组套接字（socket）序列。
* shuntdown_timeout (int) - 关闭服务器时，解除所有已连接的客户端套接字的超时时间。配置有良好关闭程序的系统基本不会用到这个超时时间，配置良好只需要几毫秒即可完成关闭。     
* ssl_context - HTTPS服务器所使用的ssl.SSLContext，None的话会使用HTTP连接。      
* print - 请传入与`print()`相同的可调用对象。覆盖`STDOUT`输出或取缔它时会有些用处。传入None也可以禁止输出。
* backlog (int) - 无法接受的连接的总数，达到设定的值时会拒绝新连接（默认128）。
* access_log - 用于存储访问日志的`logging.Logger`实例对象。传入None来禁用日志可以对响应速度有些许提升。      
* handle_signals (bool) - 是否覆盖信号TERM处理来正常关闭应用程序。
* loop - 用于运行应用程序的事件循环（默认是None）。如果loop并未显式指定，函数关闭时（调用close()后）不会对非默认循环做任何事。

## 常量
*class aiohttp.web.ContentCoding*     
&ensp;&ensp;&ensp; 可以使用的内容代码（Content Codings）的枚举（`enum.Enum`）类。

&ensp;&ensp;&ensp; **deflate**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示`DEFLATE`压缩。      

&ensp;&ensp;&ensp; **gzip**      
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 表示`GZIP`压缩。        

*aiohttp.web.identity*      
&ensp;&ensp;&ensp; 表示无压缩。      


## 中间件     
### 标准化路径的中间件
*aiohttp.web.normalize_path_middleware(\*, append_slash=True, merge_slashes=True)*       
&ensp;&ensp;&ensp; 标准化请求中路径的中间件。标准化的意思是：
* 在路径尾添加一个斜线（'/'）。
* 将两个斜线变成一个斜线（'//'->'/'）。
&ensp;&ensp;&ensp; 只要改路径符合规范，则立即返回。如果两个参数所指代的功能都是允许的话，执行顺序是这样的：
1. merge_slashes 
2. append_slash 
3. 执行merge_slashes和append_slash
&ensp;&ensp;&ensp; 如果路径至少符合其中一个条件，则重定向到一个新路径上。        
&ensp;&ensp;&ensp; 在需要添加斜线时`append_slash`也需要是Ture才会执行。是True的话当一个资源定义时尾部有斜线，但请求没有斜线时，将会自动给请求加上斜线。
&ensp;&ensp;&ensp; 如果merge_slashes是True，将会把路径中连续的斜线变成一个斜线。






