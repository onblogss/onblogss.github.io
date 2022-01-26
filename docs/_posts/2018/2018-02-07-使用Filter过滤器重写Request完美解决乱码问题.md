---
layout: post
title: 使用Filter过滤器+重写Request完美解决乱码问题
date: 2018-02-07 23:31:00
author: 薛师兄
tags: Java
---
### 一：原理

>  1.对于Post方式提交的数据，我们可以通过直接设置request和response的编码方式来解决乱码问题；但是Get方式提交的数据，那么就需要编码再解码的方式解决乱码问题。
>  2.我们一般对于GET请求获取参数有两个常用方法：   request.getParameter（String name）是获得相应名的数据，如果有重复的名，则返回第一个的值 . 接收一般变量 ，如text类型。   request.getParameterValues(String name)是获得如checkbox类（名字相同，但值有多个）的数据。 接收数组变量 ，如checkobx类型 。
>  3.所以我们的思路就是重写上述这两个方法，达到解决GET取参乱码问题。

### 二：代码

```java
package com.example.filter;

import java.io.IOException;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;

/**
 * 指定过滤器的名称以及过滤的内容
 * 单个页面可用urlPatterns="/advice"
 * 多个页面可用urlPatterns={"/advice","/advice"}
 *
 */
@WebFilter(filterName="filter", urlPatterns="/*")
public class FilterImpl implements javax.servlet.Filter {

    private String encoding="UTF-8";

    public FilterImpl() {

    }

    public void destroy() {

    }

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {   

        //1 解决中文乱码问题：1)只能解决POST乱码2)响应乱码
        request.setCharacterEncoding(encoding);
        response.setContentType("text/html;charset="+encoding);  

        //2 创建自定义的Request对象 ：解决get乱码
        MyRequest myRequest = new MyRequest((HttpServletRequest) request);  

        //3 放行  
        chain.doFilter(myRequest, response);  

    }

    public void init(FilterConfig fConfig) throws ServletException {

    }

}
```

> 继承HttpServletRequestWrapper类 ， 重写getParameter()和 getParameterValues()方法

```java
package com.example.filter;

import java.io.UnsupportedEncodingException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
/**
 * 继承HttpServletRequestWrapper
 * 重写getParameter()和 getParameterValues()方法
 */
public class MyRequest extends HttpServletRequestWrapper {

    public MyRequest(HttpServletRequest request) {
        super(request);
    }

    @Override
    public String getParameter(String name) {
        String value = super.getParameter(name);
        if (super.getMethod().equalsIgnoreCase("GET")&&value!=null) {
            try {
                value = new String(value.getBytes("ISO-8859-1"), "utf-8");
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }
        return value;
    }

    @Override
    public String[] getParameterValues(String name) {
        String[] values = super.getParameterValues(name);
        if (super.getMethod().equalsIgnoreCase("GET")&&values!=null) {
            try {
                int i=0;
                for (String value : values) {
                    values[i++] = new String(value.getBytes("ISO-8859-1"), "utf-8");
                }   
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }
        return values;
    }

}
```

>  本篇博客借鉴于： 
>  http:// blog.csdn.net/melissa_heixiu/article/details/52705457 
>  http:// blog.csdn.net/csdn_gia/article/details/54094426 
>  可以看到，第一篇博客的思想是利用的req.getParameterMap()来获取到单个或多个参数；第二篇博客较简单，重写了request.getParameter()。 
>  笔者比着葫芦画瓢，增加了request.getParameterValues()方法，并解决了参数为空导致的自定义Request类空指针异常。