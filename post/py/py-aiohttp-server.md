# Preface
Notes of http://aiohttp.readthedocs.io/en/stable/web.html#aiohttp-web-middlewares

# server

## simple server
```
from aiohttp import web
async def hello(request): return web.Response(text="Hello, world")

app = web.Application()
app.router.add_get('/', hello)

web.run_app(app)
```

## CLI, Command Line Interface (CLI)
```
# package.module:init_func accepts command-line arguments : argv
def init_func(argv):
    app = web.Application()
    app.router.add_get("/", index_handler)
    return app

$ python -m aiohttp.web -H localhost -P 8080 package.module:init_func
```

## loop.create_server + app.make_handler
```
import logging; logging.basicConfig(level=logging.INFO)

import asyncio, os, json, time
from datetime import datetime

from aiohttp import web

def index(request):
    return web.Response(body=b'<h1>Awesome</h1>')

@asyncio.coroutine
def init(loop):
    app = web.Application(loop=loop)
    app.router.add_route('GET', '/', index)
    srv = yield from loop.create_server(app.make_handler(), '127.0.0.1', 9000)
    logging.info('server started at http://127.0.0.1:9000...')
    return srv

loop = asyncio.get_event_loop()
loop.run_until_complete(init(loop))
loop.run_forever()
```

# handler
accepts a `Request` instance as its only argument and returns a `StreamResponse` derived (e.g.  `Response`) instance

```
def handler(request):
    return web.Response()
```

A handler may also be a coroutine, in which case aiohttp.web will await the handler:
```
async def handler(request):
    return web.Response()
```

## register handler with router
 registering them with the Application.router
    ```
    app.router.add_get('/', handler)
    app.router.add_post('/post', post_handler)
    app.router.add_put('/put', put_handler)
    ```
`add_route()` also supports the wildcard HTTP method,:
    ```
    app.router.add_route('*', '/path', all_handler)
    ```

By default endpoints added with add_get() will accept HEAD requests, you can deny `HEAD` request:
```
app.router.add_get('/', handler, allow_head=False)
```

## Resources and Routes
1. Internally router is a list of resources(path).
2. Resource is an entry in route table which corresponds to requested URL.
3. Resource in turn has at least one route.(list of route)

We should know
1. UrlDispatcher.add_get() / UrlDispatcher.add_post() and family are plain shortcuts for `UrlDispatcher.add_route()`.
2. `UrlDispatcher.add_route()` in turn is just a shortcut for pair of `UrlDispatcher.add_resource() and Resource.add_route()`:

    resource = app.router.add_resource(path, name=name)
    route = resource.add_route(method, handler)
    return route

### All registered resources in a router
can be viewed using the UrlDispatcher.resources() method:

    for resource in app.router.resources():
        print(resource)

Similarly,

    for name, resource in app.router.named_resources().items():
        print(name, resource

### Alternative ways for registering routes
Code examples shown above use imperative style for adding new routes: they call app.router.add_get(...) etc.

There are two alternatives: route tables and route decorators.

####  route tables
Route tables look like Django way:
```
async def handle_get(request):
    ...
async def handle_post(request):
    ...
app.router.add_routes([web.get('/get', handle_get),
                       web.post('/post', handle_post),
```

### Routes decorators
are closer to Flask approach:
    ```
    routes = web.RouteTableDef()

    @routes.get('/get')
    async def handle_get(request):
        ...
    @routes.post('/post')
    async def handle_post(request):
        ...

    app.router.add_routes(routes)
    ```
## Variable Resources¶
```
async def variable_handler(request):
    return web.Response(
        text="Hello, {}".format(request.match_info['name']))

resource = app.router.add_resource('/{name}')
resource.add_route('GET', variable_handler)
```

You can also specify a custom regex in the form `{identifier:regex}`:

    resource = app.router.add_resource(r'/{name:\d+}')

### Reverse URL Constructing using Named Resources
Routes can also be given a name:

    resource = app.router.add_resource('/root', name='root')

Which can then be used to access and build a URL for that resource later (e.g. in a request handler):
    ```
    >>> request.app.router['root'].url_for().with_query({"a": "b", "c": "d"})
    URL('/root?a=b&c=d')
    ```
A more interesting example is building URLs for variable resources:
    ```
    app.router.add_resource(r'/{user}/info', name='user-info')
    ```
In this case you can also pass in the parts of the route:

    ```
    >>> request.app.router['user-info'].url_for(user='john_doe')\
    ...                                         .with_query("a=b")
    '/john_doe/info?a=b'
    ```

### Organizing Handlers in Classes
```
lass Handler:
    async def handle_greeting(self, request):
        name = request.match_info.get('name', "Anonymous")
        txt = "Hello, {}".format(name)
        return web.Response(text=txt)

handler = Handler()
app.router.add_get('/greet/{name}', handler.handle_greeting)
```

# static file handling
app.router.add_static('/prefix', path_to_static_folder)

# Response
## Response type
1. text: return web.Response(text="Hello, world")
2. body:     resp = web.Response(body=str(r).encode('utf-8'))
    resp.content_type = 'text/plain;charset=utf-8'

## Template Rendering
1. First, setup a jinja2 environment with a call to aiohttp_jinja2.setup():

    app = web.Application()
    aiohttp_jinja2.setup(app,
        loader=jinja2.FileSystemLoader('/path/to/templates/folder'))

2. Second, wrap your handlers with the aiohttp_jinja2.template() decorator:

    @aiohttp_jinja2.template('tmpl.jinja2')
    def handler(request):
        return {'name': 'Andrew', 'surname': 'Svetlov'}


## JSON Response
    def handler(request):
        data = {'some': 'data'}
        return web.json_response(data)

## WebSockets
aiohttp.web supports WebSockets out-of-the-box.

```
async def websocket_handler(request):

    ws = web.WebSocketResponse()
    await ws.prepare(request)

    async for msg in ws:
        if msg.type == aiohttp.WSMsgType.TEXT:
            if msg.data == 'close':
                await ws.close()
            else:
                await ws.send_str(msg.data + '/answer')
        elif msg.type == aiohttp.WSMsgType.ERROR:
            print('ws connection closed with exception %s' %
                  ws.exception())

    print('websocket connection closed')
    return ws

app.router.add_get('/ws', websocket_handler)
```

# middleware
```
from aiohttp import web

def test(request):
    print('Handler function called')
    return web.Response(text="Hello")

@web.middleware
async def middleware1(request, handler):
    print('Middleware 1 called')
    response = await handler(request)
    print('Middleware 1 finished')
    return response

@web.middleware
async def middleware2(app, handler):
    print('Middleware 2 called')
    response = await handler(request)
    print('Middleware 2 finished')
    return response

app = web.Application(middlewares=[middleware1, middleware2])
app.router.add_get('/', test)
web.run_app(app)

# output
Middleware 1 called
Middleware 2 called
Handler function called
Middleware 2 finished
Middleware 1 finished
```