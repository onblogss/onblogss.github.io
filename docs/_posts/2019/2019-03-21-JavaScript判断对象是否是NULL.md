---
layout: post
title: JavaScript判断对象是否是NULL
date: 2019-03-21 20:09:00
author: 薛师兄
tags: JavaScript
---
这个方法是我踩了很多坑之后找到的，对数组等类型的对象都很好使，果断收藏！

```java
function isEmpty(obj) {
// 检验 undefined 和 null
    if (!obj && obj !== 0 && obj !== '') {
        return true;
    }
    if (Array.prototype.isPrototypeOf(obj) && obj.length === 0) {
        return true;
    }
    if (Object.prototype.isPrototypeOf(obj) && Object.keys(obj).length === 0) {
        return true;
    }
    return false;
}

```