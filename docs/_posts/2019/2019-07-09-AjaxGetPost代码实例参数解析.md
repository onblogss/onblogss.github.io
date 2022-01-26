---
layout: post
title: $.Ajax、$.Get、$.Post代码实例参数解析
date: 2019-07-09 00:19:00
author: 薛师兄
tags: JavaScript
---
## $.ajax

**语法:**

```js
$.ajax({name:value, name:value, ... })
```

**示例:**

```js
$.ajax({
    url: "/testJson",
    type: "post",
    async: false, //布尔值，表示请求是否异步处理。默认是 true。
    data: JSON.stringify({username: username, password: password}), //data表示发送的数据
    contentType: "application/json;charset=UTF-8", //定义发送请求的数据格式为JSON字符串
    dataType: "json", //定义回调响应的数据格式为JSON字符串,该属性可以省略
    success: function (result,status,xhr) { //成功响应的结果
        if (result != null) {
            alert(result);
        }
    },
    error: function (xhr,status,error) {

    }
});
```

下面的表格中列出了可能的名称/值：

| 名称                         | 值/描述                                                      |
| :--------------------------- | :----------------------------------------------------------- |
| async                        | 布尔值，表示请求是否异步处理。默认是 true。                  |
| beforeSend(*xhr*)            | 发送请求前运行的函数。                                       |
| cache                        | 布尔值，表示浏览器是否缓存被请求页面。默认是 true。          |
| complete(*xhr,status*)       | 请求完成时运行的函数（在请求成功或失败之后均调用，即在 success 和 error 函数之后）。 |
| contentType                  | 发送数据到服务器时所使用的内容类型。默认是："application/x-www-form-urlencoded"。 |
| context                      | 为所有 AJAX 相关的回调函数规定 "this" 值。                   |
| data                         | 规定要发送到服务器的数据。                                   |
| dataFilter(*data*,*type*)    | 用于处理 XMLHttpRequest 原始响应数据的函数。                 |
| dataType                     | 预期的服务器响应的数据类型。<br />可能的类型：<br />"xml" - 一个 XML 文档<br />"html" - HTML 作为纯文本<br />"text" - 纯文本字符串<br />"script" - 以 JavaScript 运行响应，并以纯文本返回<br />"json" - 以 JSON 运行响应，并以 JavaScript 对象返回<br />"jsonp" - 使用 JSONP 加载一个 JSON 块，将添加一个 "?callback=?" 到 URL 来规定回调 |
| error(*xhr,status,error*)    | 如果请求失败要运行的函数。                                   |
| global                       | 布尔值，规定是否为请求触发全局 AJAX 事件处理程序。默认是 true。 |
| ifModified                   | 布尔值，规定是否仅在最后一次请求以来响应发生改变时才请求成功。默认是 false。 |
| jsonp                        | 在一个 jsonp 中重写回调函数的字符串。                        |
| jsonpCallback                | 在一个 jsonp 中规定回调函数的名称。                          |
| password                     | 规定在 HTTP 访问认证请求中使用的密码。                       |
| processData                  | 布尔值，规定通过请求发送的数据是否转换为查询字符串。默认是 true。 |
| scriptCharset                | 规定请求的字符集。                                           |
| success(*result,status,xhr*) | 当请求成功时运行的函数。<br /> result - 包含来自请求的结果数据<br />status - 包含请求的状态（"success"、"notmodified"、"error"、"timeout"、"parsererror"）<br />xhr - 包含 XMLHttpRequest 对象 |
| timeout                      | 设置本地的请求超时时间（以毫秒计）。                         |
| traditional                  | 布尔值，规定是否使用参数序列化的传统样式。                   |
| type                         | 规定请求的类型（GET 或 POST）。                              |
| url                          | 规定发送请求的 URL。默认是当前页面。                         |
| username                     | 规定在 HTTP 访问认证请求中使用的用户名。                     |
| xhr                          | 用于创建 XMLHttpRequest 对象的函数。                         |

## $.get

**语法:**

```js
$.get(URL,callback);
```

**实例:**

```js
$.get("/try/ajax/demo_test.php",function(data,status){
    alert(status);
});
```


## $.post

**语法:**

```js
$.post(*URL,data,callback*);
```

**示例:**

```js
$.post("/try/ajax/demo_test_post.php",
    {
        name:"菜鸟教程",
        url:"http://www.runoob.com"
    },
        function(data,status){
        alert("数据: \n" + data + "\n状态: " + status);
    }
);
```