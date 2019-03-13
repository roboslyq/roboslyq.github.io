layout: post
title:  "SpringCloud应用系列(4.2) hystrix服务保护"
categories: springCloud
tags:  springCloud hystrix
author: roboslyq

* content
{:toc}
# 1.String源码解析

## trim方法

```java
 public String trim() {
        //默认结束位置
        int len = value.length;
        //默认起始位置
        int st = 0;
     	 //此处细节注意，自己代码也可以这样写。
        char[] val = value;    /* avoid getfield opcode */

        while ((st < len) && (val[st] <= ' ')) {
            st++;
        }
        while ((st < len) && (val[len - 1] <= ' ')) {
            len--;
        }
        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
    }
```

 请注意注释**avoid getfield opcode**。此代码可以提升性能，具体原因可以参考[String源码中的"avoid getfield opcode"](https://www.cnblogs.com/think-in-java/p/6130917.html)。简单总结就是："avoid getfield opcode"指在遍历实例的char数组的时候，将实例数组的引用赋值给一个本地引用，不需要频繁调用操作用码"getfield"，只需要在第一次对本地引用赋值的时候，调用一次getfield,接下来的遍历取值的时候，只需要将本地引用压入到栈顶。

## Formatter工具类

从`Jdk1.5`提供了一个工具类`java.util.Formatter`,用来格式化相关字符串。

## ValueOf操作

将其它类型转换为String，主要是利用了String的构造函数或者对象的toString()方法。

## toLowerCase 转小写

### Locale语言相关类

默认从用户环境变量中，获取语言相关信息。

```java
Locale.getDefault()
private static Locale initDefault() {
    String language, region, script, country, variant;
    language = AccessController.doPrivileged(
        new GetPropertyAction("user.language", "en"));
    // for compatibility, check for old user.region property
    region = AccessController.doPrivileged(
        new GetPropertyAction("user.region"));
    if (region != null) {
        // region can be of form country, country_variant, or _variant
        int i = region.indexOf('_');
        if (i >= 0) {
            country = region.substring(0, i);
            variant = region.substring(i + 1);
        } else {
            country = region;
            variant = "";
        }
        script = "";
    } else {
        script = AccessController.doPrivileged(
            new GetPropertyAction("user.script", ""));
        country = AccessController.doPrivileged(
            new GetPropertyAction("user.country", ""));
        variant = AccessController.doPrivileged(
            new GetPropertyAction("user.variant", ""));
    }

    return getInstance(language, script, country, variant, null);
}
```



## toUpperCase 转大写

## StringJoiner类，协助完成Joiner操作

## Split操作

## replace操作

## contain是含包含某个字符串

## StringBuffer



