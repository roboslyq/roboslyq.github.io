layout: post
title:  "SpringCloud应用系列(4.2) hystrix服务保护"
categories: springCloud
tags:  springCloud hystrix
author: roboslyq

* content
{:toc}
# 1.String本数据结构

在Java体系中，String变量的本质是字符数组，这与C语言类似。

```java
private final char value[];
```

> 所有有关与String的操作，基本是`value[]`数组进行操作。

# 2. String源码解析

## constructor

构造函数。String类为了兼容多种数据来源，包含了N多个构造函数，以适应不同的数据来源以便转换成String类型。参数主要是各种编码类型的byte数组和偏移量及长度。例如：

```java
//-----------   
public String(byte ascii[], int hibyte) {
        this(ascii, hibyte, 0, ascii.length);
    }
//------------
  public String(byte ascii[], int hibyte, int offset, int count) {
  }

```

## startsWith

常见的String操作其实是针对字符数组的操作。

比如说，判断字符串是否以某个字符串开头。

```java
public boolean startsWith(String prefix) {
        return startsWith(prefix, 0);
  }
public boolean startsWith(String prefix, int toffset) {
        char ta[] = value;
        int to = toffset;
        char pa[] = prefix.value;
        int po = 0;
        int pc = prefix.value.length;
        // Note: toffset might be near -1>>>1.
        if ((toffset < 0) || (toffset > value.length - pc)) {
            return false;
        }
        while (--pc >= 0) {
            if (ta[to++] != pa[po++]) {
                return false;
            }
        }
        return true;
    }
```

## endWith

与startWith思想一致

## HashCode

String的Hash值算法如下：

```java
/** Cache the hash code for the string */
private int hash; // Default to 0
public int hashCode() {
    //
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

> 1. hash值有缓存，缓存在hash变量中。默认为0，所以第一次获取hash需要计算。后续获取hash值不需要计算。
> 2. 为什么hashCode因子要用31？首先31是一个素数，然后31 * i == (i << 5) - i，可以使用位移和减法快速完成。

## indexOf

此操作通常用来判断某个特定字符串在当前字符串中的索引位置。常见操作原理是读取char[]数组：

```java
 public int indexOf(String str, int fromIndex) {
        return indexOf(value, 0, value.length,
                str.value, 0, str.value.length, fromIndex);
    }

static int indexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
            int fromIndex) {
    	//如果指定的起始位置大于源字符串，如果目标字符不为空，则源字符串肯定不包含目标字符串
    	//但如果目标字符串为空，则源字符串包含目标字符串，并且起始位置为源字符串的最后一位。
        if (fromIndex >= sourceCount) {
            return (targetCount == 0 ? sourceCount : -1);
        }
    	//如果起始位置小于0，调整为0
        if (fromIndex < 0) {
            fromIndex = 0;
        }
    	//如果目标字段为空，则返回fromIndex。即起始判断位置
        if (targetCount == 0) {
            return fromIndex;
        }
		//
        char first = target[targetOffset];
        int max = sourceOffset + (sourceCount - targetCount);

        for (int i = sourceOffset + fromIndex; i <= max; i++) {
            /* Look for first character. */
            if (source[i] != first) {
                while (++i <= max && source[i] != first);
            }

            /* Found first character, now look at the rest of v2 */
            if (i <= max) {
                int j = i + 1;
                int end = j + targetCount - 1;
                for (int k = targetOffset + 1; j < end && source[j]
                        == target[k]; j++, k++);

                if (j == end) {
                    /* Found whole string. */
                    return i - sourceOffset;
                }
            }
        }
        return -1;
    }
```





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



## ValueOf

将其它类型转换为String，主要是利用了String的构造函数或者对象的toString()方法。

## toLowerCase

## toUpperCase

## split

## replace

## contain

是含包含某个字符串

## intern



# 相关工具类

## StringBuffer

## StringJoiner

 协助完成Joiner操作

## Arrays

调用Arrays.copyof方法，此方法底层使用了System.arraycopy,而System.copy使用了native方法：

```java
public static char[] copyOf(char[] original, int newLength) {
        char[] copy = new char[newLength];
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
```

```java
 public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

## StringCoding

将其它类型转换`String`类型 。即String类型编码的工具类

## Locale

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

## Formatter

从`Jdk1.5`提供了一个工具类`java.util.Formatter`,用来格式化相关字符串。

## CaseInsensitiveComparator

