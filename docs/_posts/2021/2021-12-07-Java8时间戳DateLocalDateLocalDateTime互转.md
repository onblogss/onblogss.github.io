---
layout: post
title: Java8时间戳、Date、LocalDate、LocalDateTime互转
date: 2021-11-19 10:11:00
author: 薛师兄
tags:
- Date
---

> 注：Date 为 java.util.Date

## 时间戳：毫秒与秒

```java
// 毫秒转秒
long second = System.currentTimeMillis() / 1000;
// 秒转毫秒
long millisecond = second * 1000;
```

## 时间戳与Date

```java
// Date转时间戳
long millisecond = new Date().getTime();
// 时间戳转Date
Date date = new Date(millisecond);
```

## 时间戳与LocalDate

Java8新增的 LocalDate、Instant、ZoneOffset 类都是在java.time包下的。

```java
// 时间戳转LocalDate
LocalDate localDate = Instant.ofEpochMilli(System.currentTimeMillis()).atZone(ZoneOffset.ofHours(8)).toLocalDate();
// LocalDate转时间戳
long millisecond = LocalDate.now().atStartOfDay(ZoneOffset.ofHours(8)).toInstant().toEpochMilli();
```

## 时间戳与LocalDateTime

```java
// 时间戳转LocalDateTime
LocalDateTime localDateTime = Instant.ofEpochMilli(System.currentTimeMillis()).atZone(ZoneOffset.ofHours(8)).toLocalDateTime();
// LocalDateTime转时间戳
long millisecond = LocalDateTime.now().toInstant(ZoneOffset.ofHours(8)).toEpochMilli();
```

## Date与LocalDate

```java
// Date转LocalDate
LocalDate localDate = new Date().toInstant().atZone(ZoneOffset.ofHours(8)).toLocalDate();
// LocalDate转Date
Date date = Date.from(LocalDate.now().atStartOfDay(ZoneOffset.ofHours(8)).toInstant());
```

## Date与LocalDateTime

```java
// Date转LocalDateTime
LocalDateTime localDateTime = new Date().toInstant().atZone(ZoneOffset.ofHours(8)).toLocalDateTime();
// LocalDateTime转Date
Date date = Date.from(LocalDateTime.now().atZone(ZoneOffset.ofHours(8)).toInstant());
```

## LocalDate与LocalDateTime

```java
// LocalDate转LocalDateTime 时分秒默认为0点整
LocalDateTime localDateTime = LocalDate.now().atStartOfDay();
// LocalDateTime转LocalDate
LocalDate localDate = LocalDateTime.now().toLocalDate();
```