---
layout: post
title: 详解JSOUP的Select选择器语法
date: 2018-07-26 19:55:00
author: 薛师兄
tags:
  - Java
  - Jsoup
---
本文参考：[JSOUP中文文档](http://www.open-open.com/jsoup/selector-syntax.htm)

## 问题

你想使用类似于CSS或jQuery的语法来查找和操作元素。

## 方法

可以使用`Element.select(String selector)` 和 `Elements.select(String selector)` 方法实现：

```javascript
//从本地加载html文件
File input = new File("/tmp/input.html");
Document doc = Jsoup.parse(input, "UTF-8", "http://example.com/");//编码以及HTML页面URL前戳

Elements links = doc.select("a[href]"); //带有href属性的a元素
Elements pngs = doc.select("img[src$=.png]"); //扩展名为.png的图片

Element masthead = doc.select("div.masthead").first(); //class等于masthead的div标签

Elements resultLinks = doc.select("h3.r > a"); //在h3元素之后的a元素
```

## 说明

jsoup elements对象支持类似于[CSS](http://www.w3.org/TR/2009/PR-css3-selectors-20091215/) (或[jquery](http://jquery.com/))的选择器语法，来实现非常强大和灵活的查找功能。.

这个`select` 方法在`Document`, `Element`,或`Elements`对象中都可以使用。且是上下文相关的，因此可实现指定元素的过滤，或者链式选择访问。

Select方法将返回一个`Elements`集合，并提供一组方法来抽取和处理结果。

### Selector选择器概述

*  `tagname`: 通过标签查找元素，比如：`a`
*  `ns|tag`: 通过标签在命名空间查找元素，比如：可以用 `fb|name` 语法来查找 `<fb:name>` 元素
*  `#id`: 通过ID查找元素，比如：`#logo`
*  `.class`: 通过class名称查找元素，比如：`.masthead`
*  `[attribute]`: 利用属性查找元素，比如：`[href]`
*  `[^attr]`: 利用属性名前缀来查找元素，比如：可以用`[^data-]` 来查找带有HTML5 Dataset属性的元素
*  `[attr=value]`: 利用属性值来查找元素，比如：`[width=500]`
*  `[attr^=value]`, `[attr$=value]`, `[attr*=value]`: 利用匹配属性值开头、结尾或包含属性值来查找元素，比如：`[href*=/path/]`
*  `[attr~=regex]`: 利用属性值匹配正则表达式来查找元素，比如： `img[src~=(?i)\.(png|jpe?g)]`
*  `*`: 这个符号将匹配所有元素

### Selector选择器组合使用

*  `el#id`: 元素+ID，比如： `div#logo`
*  `el.class`: 元素+class，比如： `div.masthead`
*  `el[attr]`: 元素+class，比如： `a[href]`
*  任意组合，比如：`a[href].highlight`
*  `ancestor child`: 查找某个元素下子元素，比如：可以用`.body p` 查找在&rdquo;body&rdquo;元素下的所有 `p`元素
*  `parent > child`: 查找某个父元素下的直接子元素，比如：可以用`div.content > p` 查找 `p` 元素，也可以用`body > *` 查找body标签下所有直接子元素
*  `siblingA + siblingB`: 查找在A元素之前第一个同级元素B，比如：`div.head + div`
*  `siblingA ~ siblingX`: 查找A元素之前的同级X元素，比如：`h1 ~ p`
*  `el, el, el`:多个选择器组合，查找匹配任一选择器的唯一元素，例如：`div.masthead, div.logo`

### 伪选择器selectors

*  `:lt(n)`: 查找哪些元素的同级索引值（它的位置在DOM树中是相对于它的父节点）小于n，比如：`td:lt(3)` 表示小于三列的元素
*  `:gt(n)`:查找哪些元素的同级索引值大于`n``，比如`： `div p:gt(2)`表示哪些div中有包含2个以上的p元素
*  `:eq(n)`: 查找哪些元素的同级索引值与`n`相等，比如：`form input:eq(1)`表示包含一个input标签的Form元素
*  `:has(seletor)`: 查找匹配选择器包含元素的元素，比如：`div:has(p)`表示哪些div包含了p元素
*  `:not(selector)`: 查找与选择器不匹配的元素，比如： `div:not(.logo)` 表示不包含 class=logo 元素的所有 div 列表
*  `:contains(text)`: 查找包含给定文本的元素，搜索不区分大不写，比如： `p:contains(jsoup)`
*  `:containsOwn(text)`: 查找直接包含给定文本的元素
*  `:matches(regex)`: 查找哪些元素的文本匹配指定的正则表达式，比如：`div:matches((?i)login)`
*  `:matchesOwn(regex)`: 查找自身包含文本匹配指定正则表达式的元素
*  注意：上述伪选择器索引是从0开始的，也就是说第一个元素索引值为0，第二个元素index为1等

可以查看`Selector` API参考来了解更详细的内容

### 如何选择多个class值的元素

示例：`<ul class="ul-ss-3 jb-xx-ks">`

方法:

```javascript
Elements select = document.select(".ul-ss-3").select(".jb-xx-bw");
```

或者

```javascript
Elements select = document.getElementsByClass("ul-ss-3 jb-xx-bw");
```

### Selector API文档

官方API原文：[Selector (jsoup Java HTML Parser 1.11.3 API)](https://jsoup.org/apidocs/org/jsoup/select/Selector.html)

|Pattern|Matches|Example|
|---|---|---|
|`*`|任何元素|`*`|
|`tag`|具有给定标签名的元素|`div`|
|`*|E`|类型E在任何名称空间中的元素。|
|`ns|E`|类型E在名称空间中的元素。|
|`#id`|具有&ldquo;ID&rdquo;属性ID的元素|`div#wrap`, `#logo`|
|`.class`|类名为&ldquo;class&rdquo;的元素|`div.left`, `.result`|
|`[attr]`|属性为&ldquo;attr&rdquo;(任何值)的元素|`a[href]`, `[title]`|
|`[^attrPrefix]`|属性名称以&ldquo;attrPrefix&rdquo;开头的元素。使用HTML5数据集查找元素|`[^data-]`, `div[^data-]`|
|`[attr=val]`|元素的属性为&ldquo;attr&rdquo;，值为&ldquo;val&rdquo;|`img[width=500]`, `a[rel=nofollow]`|
|`[attr="val"]`|元素的属性为&ldquo;attr&rdquo;，值为&ldquo;val&rdquo;|`span[hello="Cleveland"][goodbye="Columbus"]`, `a[rel="nofollow"]`|
|`[attr^=valPrefix]`|元素的属性为&ldquo;attr&rdquo;，值以&ldquo;valPrefix&rdquo;开头|`a[href^=http:]`|
|`[attr$=valSuffix]`|元素的属性为&ldquo;attr&rdquo;，值以&ldquo;valfix&rdquo;结尾|`img[src$=.png]`|
|`[attr*=valContaining]`|元素的属性为&ldquo;attr&rdquo;，包含属性值&ldquo;valcontains&rdquo;|`a[href*=/search/]`|
|`[attr~=*regex*]`|元素具有名为&ldquo;attr&rdquo;的属性，值与正则表达式匹配|`img[src~=(?i)\.(png|
||以上可以按任何顺序合并。|`div.header[title]`|

### 关系选择器Combinators

|Pattern|Matches|Example|
|---|---|---|
|`E F`|由E元素衍生而来的F元素|`div a`, `.logo h1`|
|`E > F`|F是E的直接子结点|`ol > li`|
|`E + F`|一个F元素，紧接在E的前面|`li + li`, `div.head + div`|
|`E ~ F`|在F元素前面加上E|`h1 ~ p`|
|`E, F, G`|所有匹配元素E F G|`a[href], div, h3`|

### Pseudo selectors

|Pattern|Matches|Example|
|---|---|---|
|`:lt(*n*)`|其同胞指数小于n的元素|`td:lt(3)` 找到每一行的前3个单元格|
|`:gt(*n*)`|其同胞指数大于n的元素|`td:gt(1)` 在跳过前两个单元后查找单元格|
|`:eq(*n*)`|其同胞指数等于n的元素|`td:eq(0)` 找到每一行的第一个单元格|
|`:has(*selector*)`|包含至少一个与选择器匹配的元素的元素|`div:has(p)` 查找包含p元素的div|
|`:not(*selector*)`|不匹配选择器的元素。参见`Elements.not(String)`|`div:not(.logo)` 查找所有没有&ldquo;logo&rdquo;类的divs。 `div:not(:has(div))` 查找不包含div的div。|
|`:contains(*text*)`|包含指定文本的元素。搜索是大小写不敏感的。文本可以出现在找到的元素中，也可以出现在它的任何后代元素中。|`p:contains(jsoup)` 查找包含&ldquo;jsoup&rdquo;文本的p元素。|
|`:matches(*regex*)`|其文本与指定正则表达式匹配的元素。文本可以出现在找到的元素中，也可以出现在它的任何后代元素中。|`td:matches(\\d+)` 查找包含数字的表单元格。 `div:matches((?i)login)` 找到包含文本的div，不敏感的情况。|
|`:containsOwn(*text*)`|直接包含指定文本的元素。搜索是大小写不敏感的。文本必须出现在找到的元素中，而不是它的任何后代元素中。|`p:containsOwn(jsoup)` 查找具有自己的文本&ldquo;jsoup&rdquo;的p元素。|
|`:matchesOwn(*regex*)`|元素，其自身的文本与指定的正则表达式匹配。文本必须出现在找到的元素中，而不是它的任何后代元素中。|`td:matchesOwn(\\d+)` 查找直接包含数字的表单元格。 `div:matchesOwn((?i)login)` 找到包含文本的div，不敏感的情况。|
|`:containsData(*data*)`|包含指定数据的元素。`script`和`style`的内容元素,和`comment`节点(等)被认为是数据节点,而不是文本节点。搜索是大小写不敏感的。数据可能出现在已找到的元素中，也可能出现在其任何子代中。|`script:contains(jsoup)` 查找包含数据&ldquo;jsoup&rdquo;的脚本元素。|
||上述可按任何顺序与其他选择器组合|`.light:contains(name):eq(0)`|
|`:matchText`|将文本节点视为元素，因此允许您匹配并选择文本节点。注意，使用此选择器将修改DOM，因此您可能希望在使用之前克隆文档。|`p:matchText:firstChild` 与输入 `<p>One<br />Two</p>` 将返回一个 [`PseudoTextElement`](../../../org/jsoup/nodes/PseudoTextElement.html) 与文本 &ldquo;`One`&ldquo;.|

### Structural pseudo selectors

|Pattern|Matches|Example|
|---|---|---|
|`:root`|元素是文档的根。在HTML中，这是 `html` 元素|`:root`|
|`:nth-child(*a*n+*b*)`|在文档树中有`*a*n+*b*-1`兄妹的元素，对于任何正整数或零值的n，并具有父元素。对于a和b大于零的值，这有效地将元素的子元素划分为元素的组(最后一个组取其余的部分)，并选择每个组的第*b*th元素。例如，这允许选择器处理表中的其他行，并可以用于在一个4周期中替换段落文本的颜色。 a和b的值必须是整数(正数、负数或零)。元素的第一个子元素的索引是1。除此之外, `:nth-child()` 可以采用奇数和偶数作为参数。 奇数与2n+1的意义相同，偶数与2n的意义相同。|`tr:nth-child(2n+1)` 查找表中的每一行。 `:nth-child(10n-1)` 第9，第19，第29，等等，元素。 `li:nth-child(5)` the 5h li|
|`:nth-last-child(*a*n+*b*)`|在文档树中后面有`*a*n+*b*-1`兄弟元素。否则像`:nth-child()`|`tr:nth-last-child(-n+2)` 表的最后两行|
|`:nth-of-type(*a*n+*b*)`|伪类表示法表示一个元素，该元素具有`*a*n+*b*-1`个兄弟元素，在文档树前具有相同的扩展元素名，对于任何0或正整数值为n，并且具有父元素|`img:nth-of-type(2n+1)`|
|`:nth-last-of-type(*a*n+*b*)`|伪类表示法表示一个元素，该元素具有`*a*n+*b*-1`个兄弟元素，在文档树中，对于任何0或正整数值为n的元素，该元素具有一个父元素|`img:nth-last-of-type(2n+1)`|
|`:first-child`|元素是其他元素的第一个子元素。|`div > p:first-child`|
|`:last-child`|其他元素的最后一个子元素。|`ol > li:last-child`|
|`:first-of-type`|在父元素的子元素列表中是其类型的第一个兄弟元素|`dl dt:first-of-type`|
|`:last-of-type`|元素，它是其父元素的子元素列表中类型的最后一个兄弟元素。|`tr > td:last-of-type`|
|`:only-child`|具有父元素且父元素没有其他元素子元素的元素||
|`:only-of-type`|具有父元素的元素，其父元素没有具有相同展开元素名称的其他元素子元素||
|`:empty`|没有孩子的元素。||