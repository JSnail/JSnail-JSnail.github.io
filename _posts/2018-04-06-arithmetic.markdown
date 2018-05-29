---
layout:     post
title:      "一些有趣的算法题"
subtitle:  "一些有趣的算法题（持续更新）"
date:       2018-04-05 04:00:10
author:     "Snail"
---
##1 数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。 

###解法一：

可以设置一个常量count=0，因为需要找的数字超过一半，所以可以再设置一个常量result=数组[0],循环去比较每一个数字，找到相同的就++，反之--。

如果能不需要判断原数据里面的值是否真的超过一半，就能减少很多的判断，只需要判断count是否小于0即可


```java
  private static int find(int[] numbers) {

    int length = numbers.length;
    if (length == 0)
        return 0;

    int result = numbers[0], count = 0;
    for (int i = 1; i < length; i++) {
        if (numbers[i] == result)
            count++;
        else
            count--;
        if (count == -1) {
            result = numbers[i];
            count = 0;
        }
    }
    
    return result;
}
```
因为需要查找的数据超过一半count无论怎么减到最后一定是对应的是出现次数最多的那个值，如果还需要判断是否超过一半也很简单，在后面追加一个判断即可


```java
 private static int find(int[] numbers) {

    int length = numbers.length;
    if (length == 0)
        return 0;

    int result = numbers[0], count = 0;
    for (int i = 1; i < length; i++) {
        if (numbers[i] == result)
            count++;
        else
            count--;
        if (count == -1) {
            result = numbers[i];
            count = 0;
        }
    }

    count = 0;

    for (int number : numbers) {
        if (number == result) {
            count++;
        }
    }

    if (2 * count > length)
        return result;

    return 0;
}
```

个人觉得是最优解法了，时间复杂度为O(N)
###解法二：
可以考虑排序来解决，排序后
	
​	