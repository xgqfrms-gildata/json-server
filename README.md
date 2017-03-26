# JSON Server [![](https://travis-ci.org/typicode/json-server.svg?branch=master)](https://travis-ci.org/typicode/json-server) [![](https://badge.fury.io/js/json-server.svg)](http://badge.fury.io/js/json-server) [![](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/typicode/json-server?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


获得一个完整的假 REST API  __零编码__  **不到30秒** (认真地, 严肃地)

Created with <3 for front-end developers who need a quick back-end for prototyping and mocking.

* [Egghead.io free video tutorial - Creating demo APIs with json-server](https://egghead.io/lessons/nodejs-creating-demo-apis-with-json-server)
* [JSONPlaceholder - Live running version](http://jsonplaceholder.typicode.com)

See also:
* :hotel: [hotel - Get local domains in seconds](https://github.com/typicode/hotel)
* :dog: [husky - Git hooks made easy](https://github.com/typicode/husky)

## 范例

Create a `db.json` file

```json
{
    "posts": [{
        "id": 1,
        "title": "json-server",
        "author": "typicode"
    }],
    "comments": [{
        "id": 1,
        "body": "some comment",
        "postId": 1
    }],
    "profile": {
        "name": "typicode"
    }
}
```

启动 JSON Server

```bash
$ json-server --watch db.json
```

现在如果你到浏览器打开 [http://localhost:3000/posts/1](http://localhost:3000/posts/1), 你会得到

```json
{ "id": 1, "title": "json-server", "author": "typicode" }
```

Also when doing requests, its good to know that
- If you make POST, PUT, PATCH or DELETE requests, changes will be automatically and safely saved to `db.json` using [lowdb](https://github.com/typicode/lowdb).
- Your request body JSON should be object enclosed, just like the GET output. (for example `{"name": "Foobar"}`)
- Id values are not mutable. Any `id` value in the body of your PUT or PATCH request wil be ignored. Only a value set in a POST request wil be respected, but only if not already taken.
- A POST, PUT or PATCH request should include a `Content-Type: application/json` header to use the JSON in the request body. Otherwise it will result in a 200 OK but without changes being made to the data.

## 安装

```bash
$ npm install -g json-server
```

## 路由

Based on the previous `db.json` file, here are all the default routes. You can also add [other routes](#add-routes) using `--routes`.

### 复数(多条)路由

```
GET    /posts
GET    /posts/1
POST   /posts
PUT    /posts/1
PATCH  /posts/1
DELETE /posts/1
```

### 单数路由

```
GET    /profile
POST   /profile
PUT    /profile
PATCH  /profile
```

### 过滤器

Use `.` to access deep properties

```
GET /posts?title=json-server&author=typicode
GET /posts?id=1&id=2
GET /comments?author.name=typicode
```

### 分页

Add `_page` and in the `Link` header you will get `first`, `prev`, `next` and `last` links

```
GET /posts?_page=7
```

_10 items are returned by default_

### 排序

Add `_sort` and `_order` (ascending order by default)

```
GET /posts?_sort=views&_order=DESC
GET /posts/1/comments?_sort=votes&_order=ASC
```

### 分片

Add `_start` and `_end` or `_limit` (an `X-Total-Count` header is included in the response)

```
GET /posts?_start=20&_end=30
GET /posts/1/comments?_start=20&_end=30
GET /posts/1/comments?_start=20&_limit=10
```

### 操作

Add `_gte` or `_lte` for getting a range

```
GET /posts?views_gte=10&views_lte=20
```

Add `_ne` to exclude a value

```
GET /posts?id_ne=1
```

Add `_like` to filter (RegExp supported)

```
GET /posts?title_like=server
```

### 全文搜索

Add `q`

```
GET /posts?q=internet
```

### 关系

To include children resources, add `_embed`

```
GET /posts?_embed=comments
GET /posts/1?_embed=comments
```

To include parent resource, add `_expand`

```
GET /comments?_expand=post
GET /comments/1?_expand=post
```

To get or create nested resources (by default one level, [add routes](#add-routes) for more)

```
GET  /posts/1/comments
POST /posts/1/comments
```

### 数据库

```
GET /db
```

### 首页

Returns default index file or serves `./public` directory

```
GET /
```

## 额外的

### 静态文件服务器

You can use JSON Server to serve your HTML, JS and CSS, simply create a `./public` directory
or use `--static` to set a different static files directory.

```bash
mkdir public
echo 'hello world' > public/index.html
json-server db.json
```

```bash
json-server db.json --static ./some-other-dir
```

### 可选择端口 

You can start JSON Server on other ports with the `--port` flag:

```bash
$ json-server --watch db.json --port 3004
```

### 从任何地方访问

You can access your fake API from anywhere using CORS and JSONP.

### 远程示模式

You can load remote schemas.

```bash
$ json-server http://example.com/file.json
$ json-server http://jsonplaceholder.typicode.com/db
```

### 生成随机数据

Using JS instead of a JSON file, you can create data programmatically.

```javascript
// index.js
module.exports = function() {
  var data = { users: [] }
  // Create 1000 users
  for (var i = 0; i < 1000; i++) {
    data.users.push({ id: i, name: 'user' + i })
  }
  return data
}
```

```bash
$ json-server index.js
```

__Tip__ use modules like [faker](https://github.com/Marak/faker.js), [casual](https://github.com/boo1ean/casual) or [chance](https://github.com/victorquinn/chancejs).

### 添加 routes

Create a `routes.json` file. Pay attention to start every route with /.

```json
{
  "/api/": "/",
  "/blog/:resource/:id/show": "/:resource/:id"
}
```

Start JSON Server with `--routes` option.

```bash
json-server db.json --routes routes.json
```

Now you can access resources using additional routes.

```bash
/api/posts
/api/posts/1
/blog/posts/1/show
```

### 添加 中间件

You can add your middlewares from the CLI using `--middlewares` option:

```js
// hello.js
module.exports = function (req, res, next) {
  res.header('X-Hello', 'World')
  next()
}
```

```bash
json-server db.json --middlewares ./hello.js
json-server db.json --middlewares ./first.js ./second.js
```

### CLI 用法

```
json-server [options] <source>

Options:
  --config, -c       Path to config file           [default: "json-server.json"]
  --port, -p         Set port                                    [default: 3000]
  --host, -H         Set host                               [default: "0.0.0.0"]
  --watch, -w        Watch file(s)                                     [boolean]
  --routes, -r       Path to routes file
  --middlewares, -m  Paths to middleware files                           [array]
  --static, -s       Set static files directory
  --read-only, --ro  Allow only GET requests                           [boolean]
  --no-cors, --nc    Disable Cross-Origin Resource Sharing             [boolean]
  --no-gzip, --ng    Disable GZIP Content-Encoding                     [boolean]
  --snapshots, -S    Set snapshots directory                      [default: "."]
  --delay, -d        Add delay to responses (ms)
  --id, -i           Set database id property (e.g. _id)         [default: "id"]
  --quiet, -q        Suppress log messages from output                 [boolean]
  --help, -h         Show help                                         [boolean]
  --version, -v      Show version number                               [boolean]

Examples:
  json-server db.json
  json-server file.js
  json-server http://example.com/db.json

https://github.com/typicode/json-server
```

You can also set options in a `json-server.json` configuration file.

```json
{
  "port": 3000
}
```

### 模块

If you need to add authentication, validation, or __any behavior__, you can use the project as a module in combination with other Express middlewares.

#### 简单的例子

```js
// server.js
var jsonServer = require('json-server')
var server = jsonServer.create()
var router = jsonServer.router('db.json')
var middlewares = jsonServer.defaults()

server.use(middlewares)
server.use(router)
server.listen(3000, function () {
  console.log('JSON Server is running')
})
```

```sh
$ node server.js
```

For an in-memory database, you can pass an object to `jsonServer.router()`.
Please note also that `jsonServer.router()` can be used in existing Express projects.

#### 自定义 routes例子

Let's say you want a route that echoes query parameters and another one that set a timestamp on every resource created.

```js
var jsonServer = require('json-server')
var server = jsonServer.create()
var router = jsonServer.router('db.json')
var middlewares = jsonServer.defaults()

// Set default middlewares (logger, static, cors and no-cache)
server.use(middlewares)

// Add custom routes before JSON Server router
server.get('/echo', function (req, res) {
  res.jsonp(req.query)
})

server.use(function (req, res, next) {
  if (req.method === 'POST') {
    req.body.createdAt = Date.now()
  }
  // Continue to JSON Server router
  next()
})

// Use default router
server.use(router)
server.listen(3000, function () {
  console.log('JSON Server is running')
})
```

#### 访问控制示例

```js
var jsonServer = require('json-server')
var server = jsonServer.create()
var router = jsonServer.router('db.json')
var middlewares = jsonServer.defaults()

server.use(middlewares)
server.use(function (req, res, next) {
 if (isAuthorized(req)) { // add your authorization logic here
   next() // continue to JSON Server router
 } else {
   res.sendStatus(401)
 }
})
server.use(router)
server.listen(3000, function () {
  console.log('JSON Server is running')
})
```

#### 自定义输出示例

To modify responses, overwrite `router.render` method:

```javascript
// In this example, returned resources will be wrapped in a body property
router.render = function (req, res) {
  res.jsonp({
   body: res.locals.data
  })
}
```

#### 重写器示例

To add rewrite rules, use `jsonServer.rewriter()`:

```javascript
// Add this before server.use(router)
server.use(jsonServer.rewriter({
  '/api/': '/',
  '/blog/:resource/:id/show': '/:resource/:id'
}))
```

#### Mounting JSON Server on another endpoint example 在另一个端点上安装JSON服务器示例

Alternatively, you can also mount the router on `/api`.

```javascript
server.use('/api', router)
```

### 部署

You can deploy JSON Server. For example, [JSONPlaceholder](http://jsonplaceholder.typicode.com) is an online fake API powered by JSON Server and running on Heroku.

## 链接

### 视频

* [Creating Demo APIs with json-server on egghead.io](https://egghead.io/lessons/nodejs-creating-demo-apis-with-json-server)

### 文章

* [Node Module Of The Week - json-server](http://nmotw.in/json-server/)
* [Mock up your REST API with JSON Server](http://www.betterpixels.co.uk/projects/2015/05/09/mock-up-your-rest-api-with-json-server/)
* [ng-admin: Add an AngularJS admin GUI to any RESTful API](http://marmelab.com/blog/2014/09/15/easy-backend-for-your-restful-api.html)
* [Fast prototyping using Restangular and Json-server](http://glebbahmutov.com/blog/fast-prototyping-using-restangular-and-json-server/)
* [Create a Mock REST API in Seconds for Prototyping your Frontend](https://coligo.io/create-mock-rest-api-with-json-server/)

### 第三方工具

* [Grunt JSON Server](https://github.com/tfiwm/grunt-json-server)
* [Docker JSON Server](https://github.com/clue/docker-json-server)
* [JSON Server GUI](https://github.com/naholyr/json-server-gui)
* [JSON file generator](https://github.com/dfsq/json-server-init)

##  许可协议

MIT - [Typicode](https://github.com/typicode)
