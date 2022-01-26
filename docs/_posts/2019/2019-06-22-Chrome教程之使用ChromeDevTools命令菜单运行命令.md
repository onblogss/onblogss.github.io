---
layout: post
title: Chrome教程之使用Chrome DevTools命令菜单运行命令
date: 2019-06-22 17:25:00
author: 薛师兄
tags: Chrome
---
## 1.模拟移动设备

点击 **Toggle Device Toolbar** 
![](./20190622Chrome教程之使用ChromeDevTools命令菜单运行命令/89419086.png)

## 2.限制网络流量和 CPU 占用率

要限制网络流量和 CPU 占用率，请从 **Throttle** 列表中选择 **Mid-tier mobile** 或 **Low-end mobile**。

![](./20190622Chrome教程之使用ChromeDevTools命令菜单运行命令/18588338.png)

**Mid-tier mobile** 可模拟快速 3G 网络，并限制 CPU 占用率，以使模拟性能比普通性能低 4 倍。 **Low-end mobile** 可模拟慢速 3G 网络，并限制 CPU 占用率，以使模拟性能比普通性能低 6 倍。 请记住，限制是相对于笔记本电脑或桌面设备的普通性能而言。

请注意，如果 **Device Toolbar** 布局较窄，则会隐藏 **Throttle** 列表。

### 2.1只限制 CPU 占用率

如果只限制 CPU 占用率而不限制网络流量，请转至 **Performance** 面板，点击 **Capture Settings** 

![](./20190622Chrome教程之使用ChromeDevTools命令菜单运行命令/88148178.png)

### 2.2只限制网络流量

如果只限制网络流量而不限制 CPU 占用率，请转至 **Network** 面板，然后从 **Throttle** 列表中选择 **Fast 3G**或 **Slow 3G**。

![](./20190622Chrome教程之使用ChromeDevTools命令菜单运行命令/40766294.png)

## 2.3.替换地理定位

要打开地理定位替换界面，请点击 **Customize and control DevTools** 

![](./20190622Chrome教程之使用ChromeDevTools命令菜单运行命令/54997730.png)

从 **Geolocation** 列表中选择其中一个预设，或选择 **Custom location** 以输入自己的坐标，或选择 **Location unavailable** 以测试您的页面在地理定位处于错误状态时的表现。

![](./20190622Chrome教程之使用ChromeDevTools命令菜单运行命令/52190236.png)