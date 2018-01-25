# aiohttp 1.1的新内容

## YARL 和 URL编码
自aiohttp 1.1起，aiohttp使用yarl来进行URL的处理。     

## 新API
`yarl.URL`提供非常简便的方法来进行URL的相关操作。       
客户端API仍然可以接受str的url形式，比如`session.get()`等价于`session.get(yarl.URL('http://example.com'))`。       
内部API均已选用yarl.URL做处理。aiohttp.CookieJar将只接收URL实例对象。         
服务端方面添加了`web.Request.url`和`web.Request.rel_url`来表示请求URL的相对和绝对路径。      
使用URL是比较推荐的做法，现存的检索URL的方法已不赞成使用，并最终会被删除。      
web异常引起的重定向也支持 yarl.URL作为地址。str也会一直支持使用的。      
由路由得到URL也已做了修改。      
主API`aiohttp.web.Request.url_for(name, **kwargs)`现在会返回yarl.URL实例。已不支持查询参数，但也可以使用稍微繁琐的方式来添加查询参数:`request.url_for('name_resource', param='a').with_query(arg='val')` 。       
返回的是相对URL，绝对URL需要使用`request.url.join(request.url_for(...))`来构建。

## URL编码
在yarl.URL创建时会编码所有非ASCII的字符。        
`URL('https://www.python.org/путь')`会被编码为`'https://www.python.org/%D0%BF%D1%83%D1%82%D1%8C'`。        
填写路由表时可以使用非ASCII编码的字符和百分号字符两种路径:
```
app.router.add_get('/путь', handler)
```
和
```
app.router.add_get('/%D0%BF%D1%83%D1%82%D1%8C', handler)
```
是同样的效果。因为在内部`'/путь'`会被编码成百分号形式。       
路由匹配也同样接受两种URL格式: 原始形式和在经过路由模式转换的形式。


## 子应用
子应用被用来解决复杂的代码结构问题。假设我们有一个项目，项目中既有业务逻辑也有管理区域和调试工具。       
管理区是一个独立地应用，有其自己的逻辑，而且所有的URL都带有如'/admin'前缀的。       
所以我们可以创建一个完全独立的应用，并将其命名为'admin'，并把前缀附加到主app上:
```
admin = web.Application()
# setup admin routes, signals and middlewares

app.add_subapp('/admin/', admin)
```
中间件和信号是链式的。      
也就是说请求'/admin/something'app中的中间件会首先被调用，其次是admin。中间件是顺序调用得。      
对于信号来说也一样——信号会被发送给app和admin。      
常规信息如`on_startup`, `on_shutdown`和`on_cleanup`会被发送到所有已注册过此信号的子应用。发出的参数是这个子应用的实例对象，并不是顶级应用的实例对象。      
二层应用后可以嵌套第三层，并无层级限制。

## Url 倒推
子应用中的Url倒推会给出带有特定前缀的url。      
不过要得到URL，子应用的路由应该这么用:
```
admin = web.Application()
admin.add_get('/resource', handler, name='name')

app.add_subapp('/admin/', admin)

url = admin.router['name'].url_for()
```
得到的URL为`'/admin/resource'`。

## 应用冻结
应用可以作为主app使用（app.make_handler())也可以做为子应用使用，但不要同时使用。     
在使用`.add_subapp()`连接应用后，或开启顶级应用后，应用就被冻结了。     
这也就意味着不管是注册新路由，新信号还是新中间件都是不允许的。改变冻结应用的状态（app['name'] = 'value'）是不赞成使用的，并最终会删除。



