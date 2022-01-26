---
layout: post
title: Jsoup+FastJson制作新闻数据接口-Demo
date: 2017-12-30 21:51:00
author: 薛师兄
tags:
  - Jsoup
  - Java
  - FastJson
---
>经常用到 编写出来直接拿来用 

这个适合在服务端结合servlet来做接口：需要下载jsoup+fastjson两个包.
 
Jsoup使用手册：[http://www.open-open.com/jsoup/selector-syntax.htm](http://www.open-open.com/jsoup/selector-syntax.htm) 

fastJson使用手册：[https://www.w3cschool.cn/fastjson/fastjson-ex2.html](https://www.w3cschool.cn/fastjson/fastjson-ex2.html) 
在这里我就不重复写了，看官方API最靠谱！

```java
package com.example.jsoup;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import com.alibaba.fastjson.JSON;

public class JsoupTest {
    /**
     * Jsoup解析网页实例
     * 
     * @param i（页数）
     * @return
     */
    public static String getNew(int i) {
        String url = "http://www.cnmo.com/news/all_" + i + ".html";
        List<NewBean> list_bean = new ArrayList<>();
        NewBean newbean;
        try {
            Document doc = Jsoup.connect(url).get();
            // 获取class等于Newcon的div标签
            Element contents = doc.select("div.Newcon").first();
            Elements content = contents.getElementsByClass("Newcon-list");
            for (Element element : content) {
                Elements linka = element.getElementsByTag("a");
                String linkHref = linka.get(0).attr("href");
                String linkText = linka.get(0).text();
                Elements linkimg = element.getElementsByTag("img");
                String linkSrc = linkimg.get(0).attr("src");
                Elements linkp = element.getElementsByTag("p");
                String linktxt = linkp.get(0).text();
                // 这里把内部类修饰为static所以直接new
                newbean = new NewBean(linkText, linktxt, linkSrc, linkHref);
                list_bean.add(newbean);
            }
            // 使用了阿里的fastJson，其它json框架也可以，true是格式化
            String json = JSON.toJSONString(list_bean, true);
            return json;
        } catch (IOException e) {
            // e.printStackTrace();
            return null;
        }
    }
    /**
    *测试方法
    */
    public static void main(String[] args) {
        System.out.print(getNew(1));
    }

    public static class NewBean {
        private String title;
        private String content;
        private String imgUrl;
        private String urlA;

        public NewBean() {

        }

        public NewBean(String title, String content,
         String imgUrl, String urlA) {
            super();
            this.title = title;
            this.content = content;
            this.imgUrl = imgUrl;
            this.urlA = urlA;
        }

        public String getTitle() {
            return title;
        }

        public void setTitle(String title) {
            this.title = title;
        }

        public String getContent() {
            return content;
        }

        public void setContent(String content) {
            this.content = content;
        }

        public String getImgUrl() {
            return imgUrl;
        }

        public void setImgUrl(String imgUrl) {
            this.imgUrl = imgUrl;
        }

        public String getUrlA() {
            return urlA;
        }

        public void setUrlA(String urlA) {
            this.urlA = urlA;
        }

        @Override
        public String toString() {
            return "NewBean：[title=" + title + ", content=" + content + ", imgUrl=" + imgUrl + "urlA" + urlA + "]";
        }
    }

}
```


看着内部类写满了构造方法和getter setter方法，有没有觉得没有必要写了呢？答案是必须写，因为我们使用的是fastJson： 默认的构造函数一定要写，不然是无法解析的。

（对于fastjson 严格按照JavaBean的规范来，有一点不对就无法解析，这里一定要切记，每一个实体类的属性的get , set 方法必须写，且方法第四个字母必须大写，最好使用Eclipse的source->genreal setters and getters 生成get,set 方法，切记，切记 ）。对于Gson 这里就没有这么多的要求了，但最好还是按照JavaBean来写，避免一些位置的错误。（参考[http://blog.csdn.net/wx_962464/article/details/37612861](http://blog.csdn.net/wx_962464/article/details/37612861)）

---

项目到这里就已经可以发布到服务器上了，但是我们在客户端还需要解析json数据，这里可以参考我写的另一篇博客

下面我只写一下拿到json字符串之后要做的事：

```java
package com.example.jsoup;

import java.util.List;
import com.alibaba.fastjson.JSON;
import com.example.jsoup.JsoupTest.NewBean;

public class Demo {
    public static void main(String[] args) {
        String json = JsoupTest.getNew(1);
        List<NewBean> List = JSON.parseArray(json, NewBean.class);
        for (NewBean bean : List) {
            System.out.println(bean.toString());
        }
    }
}
```

**这里我说一下fastjson的使用心得：** 

JSONArray：相当于List 

JSONObject：相当于Map

```java
//如果不把内部类修饰为static，这句不好通过：
List<NewBean> List = JSON.parseArray(json, NewBean.class);
```