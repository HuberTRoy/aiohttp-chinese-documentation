# 流API
`aiohttp.web.Request.content`和`aiohttp.ClientResponse.content`都使用流来接受数据，同时也是流形式的API属性。     

*class aiohttp.StreamReader*   
&ensp;&ensp;&ensp;流内容读取器  
&ensp;&ensp;&ensp;除了已经存在的`aiohttp.web.Request.content`和`aiohttp.ClientResponse.content`用于读取原始内容的`StreamReader`实例，用户不要手动创建`StreamReader`实例对象。

## 读取方法
*coroutine StreamReader.read(n=-1)*     
&ensp;&ensp;&ensp;读取n字节内容，如果n没有提供或为-1，会一直读到EOF（无更多内容可读）并返回全部所读字节。  
&ensp;&ensp;&ensp;如果接受的是EOF并且内部缓存器没有内容，则返回空字节对象。  
&ensp;&ensp;&ensp;**参数**:  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;n(int) - 指定读取的字节数，-1则为全部读取。  
&ensp;&ensp;&ensp;**返回所读字节。**  
*coroutine StreamReader.readany()*      
&ensp;&ensp;&ensp;用于读取下一部分流内容。       
&ensp;&ensp;&ensp;如果内部缓存器有数据则立即返回。       
&ensp;&ensp;&ensp;**返回所读字节。**       
*coroutine StreamReader.readexactly(n)*      
&ensp;&ensp;&ensp;返回n字节数据。      
&ensp;&ensp;&ensp; 如果超过了读取上限则会抛出`asyncio.IncompleteReadError`异常，`asyncio.IncompleteReadError.partial`属性包含了已经读取到的那部分字节。      
&ensp;&ensp;&ensp; **参数:** n (int) - 要读的字节数。      
&ensp;&ensp;&ensp; **返回所读字节。**       
*coroutine StreamReader.readline()*       
&ensp;&ensp;&ensp; 读取其中的一行数据，行是以"\n"所区分开的数据。      
&ensp;&ensp;&ensp; 如果发现EOF而不是\n则会返回所读到的字节数据。     
&ensp;&ensp;&ensp; 如果发现EOF但内部缓存器是空的，则返回空字节对象。    
&ensp;&ensp;&ensp; **返回所读行。**     
*coroutine StreamReader.readchunk()*     
&ensp;&ensp;&ensp; 读取从服务器接受到的块数据。     
&ensp;&ensp;&ensp; 返回包含信息的元组（(data, end_of_HTTP_chunk)）。     
&ensp;&ensp;&ensp; 如果使用了分块传输编码，`end_of_HTTP_chunk`是一个指示末尾数据是否对应HTTP块末尾的布尔值，其他情况下都是`False`。      
&ensp;&ensp;&ensp; 返回元组[bytes, bool]:       
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;bytes指代数据块，bool指代是否对应了HTTP块的最后一部分。    


## 异步迭代
流读取器支持异步迭代。        
默认是以行读取的: 
```
async for line in response.content:
    print(line)

```
当然我们还提供几个如可以指定最大限度的数据块迭代和任何可读内容的迭代器等。

*async-for StreamReader.iter_chunked(n)*      
&ensp;&ensp;&ensp; 指定了最大可读字节的数据块迭代器:        
```
async for data in response.content.iter_chunked(1024):
    print(data)
```

*async-for StreamReader.iter_any()*       
&ensp;&ensp;&ensp; 按顺序读取在流中的数据块:         
```
async for data in response.content.iter_any():
    print(data)
```

*async-for StreamReader.iter_chunks()*        
&ensp;&ensp;&ensp; 读取从服务器接收到的数据块:         
```
async for data, _ in response.content.iter_chunks():
    print(data)
```
如果使用了分块传输编码，使用返回的元组中的第二个元素即可检索原始http块的格式信息。           
```
buffer = b""

async for data, end_of_http_chunk in response.content.iter_chunks():
    buffer += data
    if end_of_http_chunk:
        print(buffer)
        buffer = b""
```


## 其他帮助信息       
StreamReader.exception()         
&ensp;&ensp;&ensp; 返回数据读取时发生的异常。   

aiohttp.is_eof()      
&ensp;&ensp;&ensp; 如果检索到EOF则返回`Ture`。     

&ensp;&ensp;&ensp; 这个函数检索到EOF的场合内部缓存器可能不是空的。      

aiohttp.at_eof()   
&ensp;&ensp;&ensp;如果检索到EOF同时缓存器是空的则返回`True`。   

StreamReader.read_nowait(n=None)   
&ensp;&ensp;&ensp;返回内部缓存器的任何数据，没有则返回空对象。     
&ensp;&ensp;&ensp;如果其他协同程序正在等待这个流则抛出`RuntimeError`异常。    
&ensp;&ensp;&ensp;**参数**: n (int) - 要读的字节数。-1则是缓存器中的所有数据。   
&ensp;&ensp;&ensp;返回所读字节。   

StreamReader.unread_data(data)       
&ensp;&ensp;&ensp;将所读的一些内容回滚到数据流中，数据会插入到缓存器的头部。        

&ensp;&ensp;&ensp;**参数**: data (bytes) - 需要压入流中的数据。       

### 警告
    该方法不会唤醒正在等待的协同程序。
    如对read()方法就不会奏效。

coroutine aiohttp.wait_eof()   
&ensp;&ensp;&ensp;等待直到发现EOF。所给出的数据可以被接下来的读方法获取到。

