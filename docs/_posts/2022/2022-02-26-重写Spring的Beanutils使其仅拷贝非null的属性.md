---
layout: post
title: 重写Spring的Beanutils使其仅拷贝非null的属性
date: 2022-02-26 23:38:00
author: 薛师兄
tags:
- Spring
---

有时候，在更新某条数据的时候，我们会先从数据库中读取到这条数据，得到BeanA。前端会把更新的字段带过来，即参数BeanB，BeanB中仅含有部分变更的属性，为了不让BeanB中的空属性污染BeanA，可以逐行对BeanB的属性进行判空，但这种代码很枯燥，于是想到了使用Bean拷贝相关工具类。

比如Spring自带的Beanutils工具类，但它会把BeanB中的属性完封不动的拷贝过去，所以需要对其代码进行些许改造。

代码如下，复制即可使用。

```java
import org.springframework.beans.BeanUtils;
import org.springframework.beans.BeansException;
import org.springframework.beans.FatalBeanException;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;
import org.springframework.util.ClassUtils;

import java.beans.PropertyDescriptor;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.util.Arrays;
import java.util.List;

/**
 * 重写Spring的Beanutils：当源对象的值为null时，不注入该值
 */
public class BeanUtil extends BeanUtils {

    private static void copyProperties(Object source, Object target, @Nullable Class<?> editable,
                                       @Nullable String... ignoreProperties) throws BeansException {

        Assert.notNull(source, "Source must not be null");
        Assert.notNull(target, "Target must not be null");

        Class<?> actualEditable = target.getClass();
        if (editable != null) {
            if (!editable.isInstance(target)) {
                throw new IllegalArgumentException("Target class [" + target.getClass().getName() +
                        "] not assignable to Editable class [" + editable.getName() + "]");
            }
            actualEditable = editable;
        }
        PropertyDescriptor[] targetPds = getPropertyDescriptors(actualEditable);
        List<String> ignoreList = (ignoreProperties != null ? Arrays.asList(ignoreProperties) : null);

        for (PropertyDescriptor targetPd : targetPds) {
            Method writeMethod = targetPd.getWriteMethod();
            if (writeMethod != null && (ignoreList == null || !ignoreList.contains(targetPd.getName()))) {
                PropertyDescriptor sourcePd = getPropertyDescriptor(source.getClass(), targetPd.getName());
                if (sourcePd != null) {
                    Method readMethod = sourcePd.getReadMethod();
                    if (readMethod != null &&
                            ClassUtils.isAssignable(writeMethod.getParameterTypes()[0], readMethod.getReturnType())) {
                        try {
                            if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers())) {
                                readMethod.setAccessible(true);
                            }
                            Object value = readMethod.invoke(source);
                            if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
                                writeMethod.setAccessible(true);
                            }
                            // 加一个if语句，只有不等于null才拷贝
                            if (value != null) {
                                writeMethod.invoke(target, value);
                            }
                        } catch (Throwable ex) {
                            throw new FatalBeanException(
                                    "Could not copy property '" + targetPd.getName() + "' from source to target", ex);
                        }
                    }
                }
            }
        }
    }

}
```
