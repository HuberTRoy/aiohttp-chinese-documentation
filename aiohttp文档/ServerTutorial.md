# 服务端指南

准备使用aiohttp但不知道如何开始？这里有一些小例子来快速熟悉下。接下来我们一起来试着开发一个小投票系统。

如果你想改进或与之对比学习，可以查看demo source 来获取全部源码。

准备好我们的开发环境
首先检查下python版本:
```
$ python -V
Python 3.5.0
```
我们需要python 3.5.0及以上版本。


假设你已经安装好aiohttp库了。你可以用以下命令来查询当前aiohttp库的版本。
```
$ python3 -c 'import aiohttp; print(aiohttp.__version__)'
2.0.5
```

项目结构与其他以python为基础的web项目大同小异:
```
.
├── README.rst
└── polls
    ├── Makefile
    ├── README.rst
    ├── aiohttpdemo_polls
    │   ├── __init__.py
    │   ├── __main__.py
    │   ├── db.py
    │   ├── main.py
    │   ├── routes.py
    │   ├── templates
    │   ├── utils.py
    │   └── views.py
    ├── config
    │   └── polls.yaml
    ├── images
    │   └── example.png
    ├── setup.py
    ├── sql
    │   ├── create_tables.sql
    │   ├── install.sh
    │   └── sample_data.sql
    └── static
        └── style.css
```
# 开始用aiohttp构建我们的第一个应用程序

该指南借鉴了Django投票系统指南。

## 创建应用程序
aiohttp的服务端程序都是 `aiohttp.web.Application`实例对象。用于创建信号，连接路由等。

使用下列代码可以创建一个应用程序:
```

from aiohttp import web


app = web.Application()
web.run_app(app, host='127.0.0.1', port=8080)
```

将其保存在`aiohttpdemo_polls/main.py`然后开启服务器:
`$ python3 main.py`

你会在命令行中看到如下输出:

```
======== Running on http://127.0.0.1:8080 ========
(Press CTRL+C to quit)
```
在浏览器中打开 http://127.0.0.1:8080 或在命令行中使用`curl`:
`$ curl -X GET localhost:8080`

啊咧，出现了404: Not Found. 呃...因为我们并没有创建路由和和展示页面。

## 创建视图
我们来一起创建第一个展示页面(视图)。我们先创建个文件`aiohttpdemo_polls/views.py`然后写入:
```
from aiohttp import web

async def index(request):
    return web.Response(text='Hello Aiohttp!')
```

`index`就是我们创建的展示页，然后我们创建个路由连接到这个展示页上。我们来把路由放在`aiohttpdemo_polls/routes.py`文件中（将路由表和模型分开写是很好的实践。创建实际项目时可能会有多个同类文件，这样分开放可以让自己很清楚。):

```
from views import index

def setup_routes(app):
    app.router.add_get('/', index)
```

我们还要在`main.py`中调用`setup_routes`。
```
from aiohttp import web
from routes import setup_routes


app = web.Application()
setup_routes(app)
web.run_app(app, host='127.0.0.1', port=8080)
```
然后我们重新开启服务器，现在我们从浏览器中访问:

```
$ curl -X GET localhost:8080
Hello Aiohttp!
```

啊哈！成功了！我们现在应该有了一个如下所示的目录结构:
```
.
├── ..
└── polls
    ├── aiohttpdemo_polls
    │   ├── main.py
    │   ├── routes.py
    │   └── views.py
```

## 使用配置文件
aiohttp不需要任何配置文件，也没有内置支持任何配置架构。
但考虑到这些事实:
1. 99%的服务器都有配置文件。
2. 其他的同类程序(除了以Python为基础的像Django和Flask的)都不会将配置文件作为源码的一部分。
        比如Nginx将配置文件保存在 /etc/nginx文件夹里。
        mongo则保存在 /etc/mongodb.conf里。
3. 使用配置文件是公认的好方法，在部署产品时可以预防一些小错误。

所以我们建议用以下途径(进行配置文件):
1. 将配置信息写在yaml文件中。(json或ini都可以，但yaml最好用。)
2. 在一个预先设定好的目录中加载yaml。
3. 拥有能通过命令行来设置配置文件的功能。如: ./run_app --config=/opt/config/app_cfg.yaml 
4. 对要加载的字典执行严格检测，以确保其数据类型符合预期。可以使用: trafaret, colander 或 JSON schema等库。

以下代码会加载配置文件并设置到应用程序中:
```
# load config from yaml file in current dir
conf = load_config(str(pathlib.Path('.') / 'config' / 'polls.yaml'))
app['config'] = conf
```

## 构建数据库
### 准备工作
这份指南中我们使用最新版的`PostgreSQL`数据库。 你可访问以下链接下载: http://www.postgresql.org/download/

### 数据库架构
我们使用`SQLAlchemy`来写数据库架构。我们只要创建两个简单的模块——`question`和`choice`:
```
import sqlalchemy as sa 

meta = sa.MetaData()

question - sq.Table(
    'question', meta,
    sa.Column('id', sa.Integer, nullable=False),
    sa.Column('question_text', sa.String(200), nullable=False),
    sa.Column('pub_date', sa.Date, nullable=False),
    # Indexes #
    sa.PrimaryKeyConstraint('id', name='question_id_pkey')
)

choice = sa.Table(
    'choice', meta,
    sa.Column('id', sa.Integer, nullable=False),
    sa.Column('question_id', sa.Integer, nullable=False),
    sa.Column('choice_text', sa.String(200), nullable=False),
    sa.Column('votes', server_default="0", nullable=False),
    # Indexes #
    sa.PrimayKeyConstraint('id', name='choice_id_pkey'),
    sa.ForeignKeyContraint(['question_id'], [question.c.id],
                            name='choice_question_id_fkey',
                            ondelete='CASCADE'),
)
```

你会看到如下数据库结构:

第一张表 question:
|question|
|id|
|question_text|
|pub_date|

第二张表 choice:
|choice|
|id|
|choice_text|
|votes|
|question_id|

### 创建连接引擎
为了从数据库中查询数据我们需要一个引擎实例对象。假设`conf`变量是一个带有连接信息的配置字典，`Postgre`s会使用异步的方式完成该操作:
```
async def init_pg(app):
    conf = app['config']
    engine = await aiopg.sa.create_engine(
        database=conf['database'],
        user=conf['user'],
        password=conf['password'],
        host=conf['host'],
        port=conf['host'],
        minsize=conf['minsize'],
        maxsize=conf['maxsize'])

    app['db'] = engine
```
最好将连接数据库的函数放在`on_startup`信号中:
```
app.on_startup.append(init_pg)
```

### 关闭数据库
程序退出时一块关闭所有的资源接口是一个很好的做法。
使用on_cleanup信号来关闭数据库接口:
```
async def close_pg(app):
    app['db'].close()
    await app['db'].wait_closed()

app.on_cleanup.append(close_pg)
```

## 使用模板

我们来添加些更有用的页面:
```
@aiohttp_jinja2.template('detail.html')
async def poll(request):
    async with request['db'].acquire() as conn:
        question_id = request.match_info['question_id']
        try:
            question, choices = await db.get_question(conn,
                                                      question_id)
        except db.RecordNotFound as e:
            raise web.HTTPNotFound(text=str(e))
        return {
            'question': question,
            'choices': choices
        }
```

编写页面时使用模板是很方便的。我们返回带有页面内容的字典，`aiohttp_jinja2.template`装饰器会用`jinja2`模板加载它。

当然我们要先安装下`aiohttp_jinja2`:
```
$ pip install aiohttp_jinja2

```
安装完成后我们使用时要适配下:
```
import aiohttp_jinja2
import jinja2

aiohttp_jinja2.setup(
    app, loader=jinja2.PackageLoader('aiohttpdemo_polls', 'templates'))
```

我们将其放在`polls/aiohttpdemo_polls/templates`文件夹中。

## 静态文件
每个web站点都有一些静态文件: 图片啦，JavaScript，CSS文件啦等等。
在生产环境中处理这些静态文件最好的方法是使用NGINX或CDN服务做反向代理。
但在开发环境中使用aiohttp服务器处理静态文件是很方便的。

只需要简单的调用一个信号即可:
```
app.router.add_static('/static/',
                      path=str(project_root / 'static'),
                      name='static')
```
project_root表示根目录。

## 使用中间件
中间件是每个web处理器必不可少的组件。它的作用是在处理器处理请求前预处理请求以及在得到响应后发送出去。

我们下面来实现一个用于显示漂亮的404和500页面的简单中间件。
```
def setup_middlewares(app):
    error_middleware = error_pages({404: handle_404,
                                    500: handle_500})
    app.middlewares.append(error_middleware)
```

中间件(middleware)本身是一个接受*应用程序（application）*和*后续处理器（next handler）*的加工厂。

中间件工厂返回一个与web处理器一样接受请求并返回响应的中间件处理器。

下面实现一个用于处理HTTP异常的中间件:
```
def error_pages(overrides):
    async def middleware(app, handler):
        async def middleware_handler(request):
            try:
                response = await handler(request)
                override = overrides.get(response.status)
                if override is None:
                    return response
                else:
                    return await override(request, response)
            except web.HTTPException as ex:
                override = overrides.get(ex.status)
                if override is None:
                    raise
                else:
                    return await override(request, ex)
        return middleware_handler
    return middleware
```


这些`overrides（handle_404和handle_500）`只是简单的用`Jinja2`模板渲染:
```
async def handle_404(request, response):
    response = aiohttp_jinja2.render_template('404.html',
                                              request,
                                              {})
    return response


async def handle_500(request, response):
    response = aiohttp_jinja2.render_template('500.html',
                                              request,
                                              {})
    return response
```

### 详情看 Middlewares.

