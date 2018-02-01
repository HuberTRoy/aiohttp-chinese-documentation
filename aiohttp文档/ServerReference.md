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

    


