---
layout: post
title: Apache Commons Codec的Base64加解密库
date: 2018-04-15 21:20:00
author: 薛师兄
tags:
  - Java
  - 密码学
---
下载地址：[http://commons.apache.org/proper/commons-codec/download_codec.cgi](http://commons.apache.org/proper/commons-codec/download_codec.cgi)

```java
import org.apache.commons.codec.binary.Base64;

/**
 * Created by Martin
 * 2018年4月9日 下午10:17:09
*/
public class JdkBase64 {

    public static void main(String[] args) {
        String key = "这是需要加密的文字";
        key = encode(key);
        System.out.println(key);
        key = decode(key);
        System.out.println(key);
    }

    /**
     * Base64加密
     * @param key
     * @return
     */
    public static String encode(String key){
         byte[] bt = key.getBytes();
         return (new Base64().encodeToString(bt));
    }

    /**
     * Baes64解密
     * @param key
     * @return
     */
    private static String decode(String key){
        byte[] bt = new Base64().decode(key);
        return new String(bt);
    }

}
```