# 0.21版本的路由重构说明

## 合理性方面
第一代路由（v1）是基于映射对（method, path）的。映射则被称为路由。路由们都有一个唯一的名称。
第一代路由的主要的设计问题出构造路由（method, path）上，真实的URL构造工作是在资源基础上的。HTTP方法并不在URI中只是发送HTTP请求的途径。    
同时多个不同路由都是指向同一个路径的也会造成一些混乱。再有，构造多个同路径路由起名字也是一件麻烦事。
另一方面，有时同一个web处理器会绑定多个不同的HTTP方法。对于v1版本的路由可以通过传递'\*'作为HTTP方法。以类为基础的视图也通常需要'\*'方法。

## 部署方面
更改之后，**资源**成为了首要因素:
```
resource = router.add_resource('/path/{to}', name='name')
```
资源必须有一个路径（动态或静态），还可以有一个可选的名字。    
在路由器相关的地方这个名字也必须是唯一的。     
资源同时具有路由表。    
路由则相对于HTTP方法及其对应web处理器:
```
route = resource.add_route('GET', handler)
```
用户仍然可以用通配符来表示接受所有的HTTP方法（或许我们之后会添加个resource.add_wildcard(handler)的方法）。    
现在，名字是相对于资源来说的了，`app.router['name']`会返回资源实例，而不是aiohttp.web.Route对象了。   
资源具有.url()方法，所以`app.router['name'].url(parts={'a': 'b'}, query={'arg':'param'})`仍然可以使用。
改变之后允许重写静态文件处理以及部署嵌套应用。
拆散HTTP路径和方法真棒！


## 向后兼容方面
重构之后功能兼容之前99%已部署的内容。    
99%的意思就是所有的栗子和绝大多数现存代码不需要修改就可以工作，不过我们仍然在遵守着向后不兼容政策，只不过感觉不到。    
`app.router['name']`现在返回`aiohttp.web.BaseResource`实例而不是`aiohttp.web.Route`实例， 但资源具有相同的 `resource.url(...)`这个最常用的方法，所以最终用户也感觉不出变化。   
`route.match(...)`现在不在支持，请使用`aiohttp.web.AbstractResource.resolve()`代替。
`app.router.add_route(method, path, handler, name='name')`现在只是下面代码的快捷写法:
```
resource = app.router.add_resource(path, name=name)
route = resource.add_route(method, handler)
return route
```
`app.router.register_route(...)`仍然支持，每次调用都会创建aiohttp.web.ResourceAdapter对象（但已不赞成使用）。




