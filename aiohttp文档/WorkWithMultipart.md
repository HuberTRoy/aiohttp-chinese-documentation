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


# 发送Multipart请求
MultipartWriter提供将Python数据转换到multipart载体（以二进制流的形式）的接口。因为multipart格式是递归的而且支持深层嵌套，所以你可以使用with语句设计multipart数据的关闭流程:

```
with aiohttp.MultipartWriter('mixed') as mpwriter:
    ...
    with aiohttp.MultipartWriter('related') as subwriter:
        ...
    mpwriter.append(subwriter)

    with aiohttp.MultipartWriter('related') as subwriter:
        ...
        with aiohttp.MultipartWriter('related') as subsubwriter:
            ...
        subwriter.append(subsubwriter)
    mpwriter.append(subwriter)

    with aiohttp.MultipartWriter('related') as subwriter:
        ...
    mpwriter.append(subwriter)
```

MultipartWriter.append()用于将新的内容压入同一个流中。它可以接受各种输入，并且决定给这些输入用什么headers。
对于文本数据默认的Content-Type是text/plain; charset=utf-8:
```
mpwriter.append('hello')
```
二进制则是 application/octet-stream:
```
mpwriter.append(b'aiohttp')
```
你也可以使用第二参数来覆盖默认值:
```
mpwriter.append(io.BytesIO(b'GIF89a...'),
                {'CONTENT-TYPE': 'image/gif'})
```
对于文件对象Content-Type会使用Python的mimetypes模块来做判断，此外，Content-Disposition头会把文件的基本名包含进去。
```
part = root.append(open(__file__, 'rb'))
```

如果你想给文件设置个其他的名字，只需要操作BodyPartWriter实例即可，使用BodyPartWriter.set_content_disposiition()后MultipartWriter.append()方法总会显式的返回和设置Content-Disposition:
```
part.set_content_disposition('attachment', filename='secret.txt')
```
此外，你还可以设置些其他的头信息:
```
part.headers[aiohttp.hdrs.CONTENT_ID] = 'X-12345'
```
如果你设置了Content-Encoding，后续的数据都会自动编码:
```
part.headers[aiohttp.hdrs.CONTENT_ENCODING] = 'gzip'
```
常用的方法还有MultipartWriter.append_json()和MultipartWriter.append_form()对JSON和表单数据非常好用，这样你就不需要每次都手动编码成需要的格式:
```
mpwriter.append_json({'test': 'passed'})
mpwriter.append_form([('key', 'value')])
```
最后，只需要将根MultipartWriter实例通过aiohttp.client.request()的data参数传递出去即可:
```
await aiohttp.post('http://example.com', data=mpwriter)
```
后台的MultipartWriter.serialize()对每个部分都生成一个块，如果拥有Content-Encoding或者Content-Transfer-Encoding头信息会被自动应用到流数据上。

注意，在被MultipartWriter.serialize()处理时，所有的文件对象都会被读至末尾，不将文件指针重置到开始时是不能重复读取的。

# Multipart技巧

互联网上充满陷阱，有时你可能会发现一个支持Multipart的服务器出现些奇怪的情况。
比如，如果服务器使用了cgi.FieldStorage，你就必须确认是否包含Content-Length头信息:
```
for part in mpwriter:
    part.headers.pop(aiohttp.hdrs.CONTENT_LENGTH, None)
```
另一方面，有些服务器可能需要你为所有的multipart请求指定Content-Length头信息。但aiohttp并不会指定因为默认是用块传输来发送multipart的。要实现的话你必须连接MultipartWriter来计算大小:
```
body = b''.join(mpwriter.serialize())
await aiohttp.post('http://example.com',
                   data=body, headers=mpwriter.headers)
```
有时服务器的响应并没有一个很好的格式: 可能不包含嵌套部分。比如，我们请求的资源返回JSON和文件的混合体。如果响应中有任何附加信息，他们应该使用嵌套multipart的形式。如果没有则是普通形式:
```
CONTENT-TYPE: multipart/mixed; boundary=--:

--:
CONTENT-TYPE: application/json

{"_id": "foo"}
--:
CONTENT-TYPE: multipart/related; boundary=----:

----:
CONTENT-TYPE: application/json

{"_id": "bar"}
----:
CONTENT-TYPE: text/plain
CONTENT-DISPOSITION: attachment; filename=bar.txt

bar! bar! bar!
----:--
--:
CONTENT-TYPE: application/json

{"_id": "boo"}
--:
CONTENT-TYPE: multipart/related; boundary=----:

----:
CONTENT-TYPE: application/json

{"_id": "baz"}
----:
CONTENT-TYPE: text/plain
CONTENT-DISPOSITION: attachment; filename=baz.txt

baz! baz! baz!
----:--
--:--
```
在单个流内读取这样的数据是可以的，不过并不清晰:
```
result = []
while True:
    part = await reader.next()

    if part is None:
        break

    if isinstance(part, aiohttp.MultipartReader):
        # Fetching files
        while True:
            filepart = await part.next()
            if filepart is None:
                break
            result[-1].append((await filepart.read()))

    else:
        # Fetching document
        result.append([(await part.json())])
```
我们换一种方式来处理，让普通文档和与文件相关的读取器成对附到每个迭代器上:
```
class PairsMultipartReader(aiohttp.MultipartReader):

    # keep reference on the original reader
    multipart_reader_cls = aiohttp.MultipartReader

    async def next(self):
        """Emits a tuple of document object (:class:`dict`) and multipart
        reader of the followed attachments (if any).

        :rtype: tuple
        """
        reader = await super().next()

        if self._at_eof:
            return None, None

        if isinstance(reader, self.multipart_reader_cls):
            part = await reader.next()
            doc = await part.json()
        else:
            doc = await reader.json()

        return doc, reader
```
这样我们就可以更轻快的解决:
```
reader = PairsMultipartReader.from_response(resp)
result = []
while True:
    doc, files_reader = await reader.next()

    if doc is None:
        break

    files = []
    while True:
        filepart = await files_reader.next()
        if file.part is None:
            break
        files.append((await filepart.read()))

    result.append((doc, files))
```

# 扩展
Multipart API in Helpers API section.
