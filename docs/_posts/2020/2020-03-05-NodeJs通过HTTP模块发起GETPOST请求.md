---
layout: post
title: NodeJs通过HTTP模块发起GET/POST请求
date: 2020-03-05 14:49:00
author: 薛师兄
tags: nodejs
---

Node.js 的 http 模块和 https 模块在使用差不多，只是换个名称，本文以 http 模块为例。http 的 get / post 等请求都是以 `http.request()` 方法进行的，所以搞懂该方法就搞懂了 Node.js 的 http 模块。 

`http.request()` 方法有两种形参，区别在于是否添加 url 参数。

- http.request(options[, callback])

- http.request(url\[, options][, callback])

url 可以是字符串或 [URL](http://nodejs.cn/s/5dwq7G) 对象。 如果 `url` 是一个字符串，则会自动使用  url.URL() 解析它。 如果它是一个 [URL](http://nodejs.cn/s/5dwq7G) 对象，则会自动转换为普通的 options 对象。

如果同时指定了 url 和 options，则对象会被合并，其中 options 属性优先。

**官方文档**：[http://nodejs.cn/api/http.html#http_http_request_url_options_callback](http://nodejs.cn/api/http.html#http_http_request_url_options_callback)

## POST

```js
const http = require('http')
const querystring = require('querystring')
const zlib = require('zlib')

// 用于将对象转换成query字符串
const postData = querystring.stringify({
                                           'msg': '你好世界',
                                           'date': '2020-3-1'
                                       });
console.log(`postData: ${postData}`)
const options = {
    hostname: 'nodejs.cn',
    port: 80,
    path: '/upload',
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Content-Length': Buffer.byteLength(postData)
    }
};

const request = http.request(options, (response) => {
    if (response.headers['content-encoding'] === 'gzip') {
        console.log('解决返回数据使用gzip进行压缩')
        let gzip = zlib.createGunzip();
        response.pipe(gzip);
        response = gzip;
    }

    console.log(`状态码: ${response.statusCode}`);
    console.log(`响应头: ${JSON.stringify(response.headers)}`);

    response.setEncoding('utf8');

    let body = ''
    response.on('data', (chunk) => {
        body+=chunk;
    });
    response.on('end', () => {
        console.log(`响应主体: ${body}`);
    });
    
});

request.on('error', (e) => {
    console.error(`请求遇到问题: ${e.message}`);
});

// 将数据写入请求主体。
request.write(postData);
request.end();
```

## GET

GET方法除了使用 `http.request()` ，还可以使用 `http.get() `方法。

- http.get(options[, callback])

- http.get(url\[, options][, callback])

由于大多数请求都是没有主体的 GET 请求，因此 Node.js 提供了这个便捷的方法。 这个方法与 http.request() 的唯一区别是它将方法设置为 GET 并自动调用 request.end()。

如果没有添加 response 事件处理函数，则响应将会被完全地丢弃。 如果添加了 response 事件处理函数，则**必须消费完响应对象中的数据**，消费方式包括：每当有 readable 事件时调用 `response.read()`、添加 'data' 事件处理函数、通过调用 `.resume()` 方法。在消费完数据之前，不会触发 end 事件。 此外，在读取数据之前，它将会占用内存，这最终可能导致进程内存不足的错误。

与 request 对象不同，如果响应过早地关闭，则 response 对象不会触发 'error' 事件而是触发 'aborted' 事件。

```js
const http = require('http')
const querystring = require('querystring')

const getData = querystring.stringify({
                                            'time': '2019-03-22 14:54:55',
                                            'page': 2
                                        })

http.get(`http://nodejs.cn/?${getData}`, (response) => {
    
    console.log(`状态码: ${response.statusCode}`);
    console.log(`响应头: ${JSON.stringify(response.headers)}`);
    
    if (response.statusCode!==200){
        // 如果不想读取数据一定记得手动消费哦
        response.resume();
        return;
    }
    
    response.setEncoding('utf8');
    let body = '';
    response.on('data', (chunk) => {
        body += chunk;
    });
    response.on('end', () => {
        console.log(`响应主体: ${body}`);
    });

}).on('error', (e) => {
    console.error(`请求出现问题: ${e.message}`);
});
```

## Options

options 对象包括如下属性：

- `agent` \<http.Agent>| \<boolean> 控制 [`Agent`](http://nodejs.cn/s/HRCnER) 的行为。可能的值有：
  - `undefined` (默认): 对此主机和端口使用 [`http.globalAgent`](http://nodejs.cn/s/g7BhW2)。
  - `Agent` 对象: 显式地使用传入的 `Agent`。
  - `false`: 使用新建的具有默认值的 `Agent`。
- `auth` \<string> 基本的身份验证，即 `'user:password'`，用于计算授权请求头。
- `createConnection` \<Function> 当 `agent` 选项未被使用时，用来为请求生成套接字或流的函数。这可用于避免创建自定义的 `Agent` 类以覆盖默认的 `createConnection` 函数。详见 [`agent.createConnection()`](http://nodejs.cn/s/nH3X12)。任何[双工流](http://nodejs.cn/s/2iRabr)都是有效的返回值。
- `defaultPort` \<number> 协议的默认端口。 如果使用 `Agent`，则默认值为 `agent.defaultPort`，否则为 `undefined`。
- `family` \<number> 当解析 `host` 或 `hostname` 时使用的 IP 地址族。有效值为 `4` 或 `6`。如果没有指定，则同时使用 IP v4 和 v6。
- `headers` \<object> 包含请求头的对象。
- `host` \<string> 请求发送至的服务器的域名或 IP 地址。**默认值:** `'localhost'`。
- `hostname`\<string>  `host` 的别名。为了支持 [`url.parse()`](http://nodejs.cn/s/b28B2A)，如果同时指定 `host` 和 `hostname`，则使用 `hostname`。
- `insecureHTTPParser` \<boolean> 使用不安全的 HTTP 解析器，当为 `true` 时接受无效的 HTTP 请求头。应避免使用不安全的解析器。有关更多信息，参阅 [`--insecure-http-parser`](http://nodejs.cn/s/5Bnm43)。**默认值:** `false`。
- `localAddress`\<string> 为网络连接绑定的本地接口。
- `lookup` \<Function> 自定义的查找函数。 **默认值:** [`dns.lookup()`](http://nodejs.cn/s/LJLsTL)。
- `method`\<string> 一个字符串，指定 HTTP 请求的方法。**默认值:** `'GET'`。
- `path` \<string> 请求的路径。应包括查询字符串（如果有）。例如 `'/index.html?page=12'`。当请求的路径包含非法的字符时，则抛出异常。目前只有空格被拒绝，但未来可能会有所变化。**默认值:** `'/'`。
- `port` \<number> 远程服务器的端口。**默认值:** `defaultPort`（如果有设置）或 `80`。
- `protocol` \<string> 使用的协议。**默认值:** `'http:'`。
- `setHost` \<boolean>: 指定是否自动添加 `Host` 请求头。**默认值:** `true`。
- `socketPath` \<string> Unix 域套接字。如果指定了 `host` 或 `port` 之一（它们指定了 TCP 套接字），则不能使用此选项。
- `timeout` \<number> : 指定套接字超时的数值，以毫秒为单位。这会在套接字被连接之前设置超时。