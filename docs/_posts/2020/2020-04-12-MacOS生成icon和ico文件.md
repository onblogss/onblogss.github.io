---
layout: post
title: Mac OS 生成 icon 和 ico 文件
date: 2020-04-12 22:00:00
author: 薛师兄
tags: 
  - Electron
  - Mac
---

## 1. 生成 ICON 文件

Mac 电脑自带 iconutil 工具，如果你没有 Mac 环境，可以使用 iconutil 的 npm 包，[Github 地址](https://github.com/wtfaremyinitials/iconutil)。

首先，当前终端目录下我需要有一个 1024x1024 大小的图片，假设文件名为 app.png 。

执行命令行脚本：

```shell
mkdir icons.iconset &&

sips -z 16 16     app.png --out icons.iconset/icon_16x16.png &&
 
sips -z 32 32     app.png --out icons.iconset/icon_16x16@2x.png &&
 
sips -z 32 32     app.png --out icons.iconset/icon_32x32.png &&
 
sips -z 64 64     app.png --out icons.iconset/icon_32x32@2x.png &&

sips -z 64 64     app.png --out icons.iconset/icon_64x64.png &&
 
sips -z 128 128   app.png --out icons.iconset/icon_64x64@2x.png &&
 
sips -z 128 128   app.png --out icons.iconset/icon_128x128.png &&

sips -z 256 256   app.png --out icons.iconset/icon_128x128@2x.png &&
 
sips -z 256 256   app.png --out icons.iconset/icon_256x256.png &&
 
sips -z 512 512   app.png --out icons.iconset/icon_256x256@2x.png &&
 
sips -z 512 512   app.png --out icons.iconset/icon_512x512.png &&
 
sips -z 1024 1024   app.png --out icons.iconset/icon_512x512@2x.png &&

iconutil -c icns icons.iconset -o app.icns &&

rm -rf icons.iconset
```

你会在 app.png 旁边发现生成的 app.icns 文件。

## 2. 生成 ICO 文件

需要使用 npm 环境，安装 png-to-ico 工具：

```shell
npm install png-to-ico --save-dev
```

在 npm script 脚本中使用：

```shell
png-to-ico app.png > app.ico
```

可以将脚本写在 package.json 的 script 对象中：

```json
  "scripts": {
    "ico": "png-to-ico app.png > app.ico"
  }
```

调用方式：

```shell
npm run ico
```

或者在 node.js 程序中使用:

```js
const fs = require('fs');
const pngToIco = require('png-to-ico');

pngToIco('app.png')
  .then(buf => {
		fs.writeFileSync('app.ico', buf);
	})
	.catch(console.error);
```