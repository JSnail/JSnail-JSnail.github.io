---
layout: post
title: String转化成Integer的方式及原理
categories: interview
description: String转化成Integer的方式及原理
keywords: String,Integer
---

### String转化成Integer的方式及原理

​	当我们想把一个string转换为int类型的时候需要使用如下方法

```java
public static int parseInt(String s) throws NumberFormatException {
    	//后面的10表示10进制
        return parseInt(s,10);
    }
```

可以看到他内部默认调用了parseInt方法，Integer.valueOf()在内部实际上也是调用的parseInt，只是多了一步使用缓存的判断，我们来看看这个方法里面有什么

```java
 public static int parseInt(String s, int radix)
                throws NumberFormatException
    {
        if (s == null) {
            throw new NumberFormatException("null");
        }

        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }
	
        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        int result = 0;
        //是否是负数
        boolean negative = false;
        int i = 0, len = s.length();
        int limit = -Integer.MAX_VALUE;
        int multmin;
        int digit;

        if (len > 0) {
            char firstChar = s.charAt(0);
            //判断第一个字符是否是小于0的ASCII码48
            if (firstChar < '0') { // Possible leading "+" or "-"
            //判断是否是负数
                if (firstChar == '-') {
                    negative = true;
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);

                if (len == 1) // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            //因为下方*了基准数，防止超过范围
            multmin = limit / radix;
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                //根据进制，获取他转换后的数字例如digit('3',10)返回10
                digit = Character.digit(s.charAt(i++),radix);
                //如果小于0表示为非数字，抛出异常
                if (digit < 0) {
                    throw NumberFormatException.forInputString(s);
                }
                if (result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                //result乘以基准数为得到位置
                //例如第一次的result为-1，第二次乘以10后为-10
                result *= radix;
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                //因为计算出来为负数，通过-的方式累加数字
                //一种和巧妙的方式，就是每次都提高一次位数
                //通过计算最后一位的方式来循环添加数据
                result -= digit;
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
        //通过上方的判断是否为负数取相反数
        return negative ? result : -result;
    }
```

