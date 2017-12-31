# 使用Multipart

aiohttp 支持功能完备的multipart 读取器和写入器。这俩都使用流式设计，以避免不必要的占用，尤其是处理的载体较大时，但这也意味着大多数I/O操作只能被执行一次。

# 读取Multipart响应
假设你发起了一次请求，然后想读取multipart响应数据:

```
async with aiohttp.request(...) as resp:
    pass
```

首先，你需要使用MultipartReader.from_response()来处理下响应内容。这样可以让数据从响应和连接中分离出来并保持MultipartReader状态，使其更便捷的使用:
```
reader = aiohttp.MultipartReader.from_response(resp)
```
假设我们需要接受JSON数据和multipart文件，但并不需要所有数据，只是其中的一个。

那我们首先需要进入一段循环中，在里面处理multipart：
```
metadata = None
filedata = None
while True:
    part = await reader.next()
```
所返回的类型取决于下一次循环时的值: 如果是一个正常响应内容那会得到BodyPartReader实例对象，否则将会是一个嵌套multipart的MultipartReader实例对象。记住，multipart的格式就是递归并且支持嵌套多层。如果接下来没有内容可以获取了，则返回None - 然后就可以跳出这个循环了:

```
if part is None:
    break
```

BodyPartReader和MultipartReader都可访问内容的headers: 这样就可以使用他们的属性来进行过滤:
```
if part.headers[aiohttp.hdrs.CONTENT_TYPE] == 'application/json':
    metadata = await part.json()
    continue
```
不明确说明的话，不管是BodyPartReader还是MultipartReader都不会读取出全部的内容。BodyPartReader提供一些易用的方法来帮助获取比较常见的内容类型:
* BodyPartReader.text() 普通文本内容。
* BodyPartReader.json() JSON内容。
* BodyPartReader.form() application/www-urlform-encode内容。
如果传输内容使用了gzip和deflate进行过编码则会自动识别，或者如果是base64或quoted-printable这种情况也会自动解码。不过如果你需要读取原始数据，使用BodyPartReader.read()和BodyPartReader.read_chunk()协程方法都可以读取原始数据，只不过一个是一次性读取全部一个是分块读取。
BodyPartReader.filename属性对于处理multipart文件时可能会有些用处:
```
if part.filename != 'secret.txt':
    continue
```
当前的内容不符合你的期待然后要跳过的话只需要使用continue来继续这个循环。在获取下一个内容之前`await reader.next()`会确保之前那个已经完全被读取出来了。如果没有的话，所有的内容将会被抛弃然后来获取下一个内容。所以你不用关心如何清理这些无用的数据。
一旦你发现你搜寻的那个文件，直接读就行。我们可以先不使用解码读取:
```
filedata = await part.read(decode=False) 
```
之后如果要解码的话也很简单:
```
filedata = part.decode(filedata)
```
一旦完成了关于multipart的处理，只需要跳出循环就好了:
```
break
```


