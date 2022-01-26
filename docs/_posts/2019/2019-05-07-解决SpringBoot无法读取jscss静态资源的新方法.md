---
layout: post
title: 解决SpringBoot无法读取js/css静态资源的新方法
date: 2019-05-07 11:36:00
author: 薛师兄
tags: SpringBoot
---
### 前言

作为依赖使用的SpringBoot工程很容易出现自身静态资源被主工程忽略的情况。但是作为依赖而存在的Controller方法却不会失效，我们知道，Spring MVC对于静态资源的处理也不外乎是路径匹配，读取资源封装到Response中响应给浏览器，所以，解决的途径就是自己写一个读取Classpath下静态文件并响应给客户端的方法。

对于ClassPath下文件的读取，最容易出现的就是IDE运行ok，打成jar包就无法访问了，该问题的原因还是在于getResources()不如getResourceAsStream()方法靠谱。

### 读取classpath文件

本就是SpringBoot的问题场景，何不用Spring现成的ClassPathResource类呢？

ReadClasspathFile.java

```java
public class ReadClasspathFile {
    public static String read(String classPath) throws IOException {
        ClassPathResource resource = new ClassPathResource(classPath);
        BufferedReader reader = new BufferedReader(new InputStreamReader(resource.getInputStream(),"UTF-8"));
        StringBuilder builder = new StringBuilder();
        String line;
        while ((line = reader.readLine())!=null){
            builder.append(line+"\n");
        }
        return builder.toString();
    }
}
```

上面的代码并不是特别规范，存在多处漏洞。比如没有关闭IO流，没有判断文件是否存在，没有考虑到使用缓存进行优化。

这里为什么考虑缓存呢？如果不加缓存，那么每次请求都涉及IO操作，开销也比较大。关于缓存的设计，这里使用WeakHashMap，最终代码如下：

```java
public class ReadClasspathFile {
    
    private static WeakHashMap<String, String> map = new WeakHashMap<>();

    public static String read(String classPath) {
        //考虑到数据的一致性，这里没有使用map的containsKey()
        String s = map.get(classPath);
        if (s != null) {
            return s;
        }
        //判空
        ClassPathResource resource = new ClassPathResource(classPath);
        if (!resource.exists()) {
            return null;
        }
        //读取
        StringBuilder builder = new StringBuilder();
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(resource.getInputStream(), "UTF-8"))) {
            String line;
            while ((line = reader.readLine()) != null) {
                builder.append(line).append("\n");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        //DCL双检查锁
        if (!map.containsKey(classPath)) {
            synchronized (ReadClasspathFile.class) {
                if (!map.containsKey(classPath)) {
                    map.put(classPath, builder.toString());
                }
            }
        }
        return builder.toString();
    }
}
```

但这样就完美了吗？其实不然。对于html/css等文本文件，这样看起来似乎并没有什么错误，但对于一些二进制文件，就会导致浏览器解码出错。为了万无一失，服务端应该完全做到向客户端返回原生二进制流，也就是字节数组。具体的解码应由浏览器进行判断并实行。

```java
public class ReadClasspathFile {

    private static WeakHashMap<String, byte[]> map = new WeakHashMap<>();

    public static byte[] read(String classPath) {
        //考虑到数据的一致性，这里没有使用map的containsKey()
        byte[] s = map.get(classPath);
        if (s != null) {
            return s;
        }
        //判空
        ClassPathResource resource = new ClassPathResource(classPath);
        if (!resource.exists()) {
            return null;
        }
        //读取
        ByteArrayOutputStream stream = new ByteArrayOutputStream();
        try (BufferedInputStream bufferedInputStream = new BufferedInputStream(resource.getInputStream());
             BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(stream)) {
            byte[] bytes = new byte[1024];
            int n;
            while ((n = bufferedInputStream.read(bytes))!=-1){
                bufferedOutputStream.write(bytes,0,n);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        //DCL双检查锁
        if (!map.containsKey(classPath)) {
            synchronized (ReadClasspathFile.class) {
                if (!map.containsKey(classPath)) {
                    map.put(classPath, stream.toByteArray());
                }
            }
        }
        return stream.toByteArray();
    }
}
```

### 自定义映射

接下来就是Controller层进行映射匹配响应了，这里利用Spring MVC取个巧，代码如下：

```java
@ResponseBody
@RequestMapping(value = "view/{path}.html",produces = {"text/html; charset=UTF-8"})
public String view_html(@PathVariable String path) throws IOException {
    return ReadClasspathFile.read("view/"+path+".html");
}

@ResponseBody
@RequestMapping(value = "view/{path}.js",produces = {"application/x-javascript; charset=UTF-8"})
public String view_js(@PathVariable String path) throws IOException {
    return ReadClasspathFile.read("view/"+path+".js");
}

@ResponseBody
@RequestMapping(value = "view/{path}.css",produces = {"text/css; charset=UTF-8"})
public String view_html(@PathVariable String path) throws IOException {
    return ReadClasspathFile.read("view/"+path+".css");
}

```

通过后戳（html、js）进行判断，以应对不同的Content-Type类型，静态资源的位置也显而易见，位于resources/view下。

但是，使用@PathVariable注解的这种方式不支持多级路径，也就是不支持包含“/”，为了支持匹配多级目录，我们只能放弃这种方案，使用另一种方案。

```java
@ResponseBody
@RequestMapping(value = "/view/**",method = RequestMethod.GET)
public void view_js(HttpServletResponse response, HttpServletRequest request) throws IOException {
    String uri = request.getRequestURI().trim();
    if (uri.endsWith(".js")){
        response.setContentType("application/javascript");
    }else if (uri.endsWith(".css")){
        response.setContentType("text/css");
    }else if (uri.endsWith(".ttf")||uri.endsWith(".woff")){
        response.setContentType("application/octet-stream");
    }else {
        String contentType = new MimetypesFileTypeMap().getContentType(uri);
        response.setContentType(contentType);
    }
    response.getWriter().print(ReadClasspathFile.read(uri));
}
```

将读取文件的静态方法更换为我们最新的返回字节流的方法，最终代码为：

```java
@RequestMapping(value = "/tree/**",method = RequestMethod.GET)
public void view_js(HttpServletResponse response, HttpServletRequest request) throws IOException {
    String uri = request.getRequestURI().trim();
    if (uri.endsWith(".js")){
        response.setContentType("application/javascript");
    }else if (uri.endsWith(".css")){
        response.setContentType("text/css");
    }else if (uri.endsWith(".woff")){
        response.setContentType("application/x-font-woff");
    }else if (uri.endsWith(".ttf")){
        response.setContentType("application/x-font-truetype");
    }else if (uri.endsWith(".html")){
        response.setContentType("text/html");
    }
    byte[] s = ReadClasspathFile.read(uri);
    response.getOutputStream().write(Optional.ofNullable(s).orElse("404".getBytes()));
}
```