---
layout: post
title: java基础知识整理（一）
categories: Java
description: ==和equals和hashCode
keywords: equals，hashCode
---

### java中==和equals和hashCode的区别

#### 1.	==

  ‘==’ 用于数据类型的比较可分为2类：

1.	基本数据类型 

    用于基本数据类型之间的值的比较


2. 引用类型（类、接口、数组等等）

  当他们进行比较的时候，比较的是他们在内存中的内存地址，除非是同一个对象，他们的值永远为false。

  由于对象是放在堆中的，对象的引用或者说内存地址是存放在栈中，因此要比较堆中的对象，就需要使用equals()方法了，可以使用int 和integer的比较来判断，例如：

  ```java
  int a =1;
  Integer b1 =1;
  Integer b2 = new Integer(1);
  Integer b3 = Integer.valueOf(10);
  Integer b4 = 10;
  ```

以上代码的一些比较结果如下

```java
	a == b1;//true
	a == b2;//false
	b1 == b2;//false 
	b3 == b4;//true 
```

Integer变量和int变量比较时，只要两个变量的值是向等的，则结果为true（因为包装类Integer和基本数据类型int比较时，java会自动拆箱为int，然后进行比较，实际上就变为两个int变量的比较，所以a==b1为true，而 b3==b4结果为true是因为在java中有如下一段代码

```java
public static Integer valueOf(int i) {  
      assert IntegerCache.high >= 127;  
         if (i >= IntegerCache.low && i <=IntegerCache.high)  
              return IntegerCache.cache[i + (-IntegerCache.low)];  
     return new Integer(i);  
 } 
```

由源码可知java会缓存-127到127之间的值，当b4重新被赋值为10的时候，会直接从缓存里面获取，不会重新创建因此b3==b4的为true。


#### 2.	equals

1.	在没有重写父类的equals()方法的情况下，默认是调用objec类中的equals()方法，而此时比较的是两个对象之间的内存地址，判断的是是不是同一个对象，此时和==的比较是一样的

		public boolean equals(Object obj) {  
    	         return (this == obj);  
   ​			 } 

	.	在有重写父类的equals()的方法的时候就需要根据重写的方法具体去判断了
3. equals有4点需要注意

- 非空性：任何非空的值调用equals去比较null的时候一定为false

- 对称性：a.equals(b)为true，反之也为true

- 传递性：如果a.equals(b)=true, b.equals(c)=true,那么a.equals(c)=true 

- 一致性：当参与比较的对象没有变化的时候，比较的结果也不会变

  ### 3.	hashcode

  ```java
  public native int hashCode();  
  ```

object中的hashCode()方法就返回一个int值，他的主要作用是对象进行散列的时候作为一个key，因此生成的key是通过哈希算法，将数据依特定算法直接指定到一个地址上，同时也保证了每个值的不同。

在java中有list和set两种不同的集合，前者元素有序可重复，后者无序不可重复，这里就存在一个问题，怎么判断不能重复呢？如果是单纯的调用equals()方法，每添加一个元素就要去比较一次，当比较的体量足够大的时候，效率简直是惨不忍睹，于是hashcode就站了出来。

当集合添加一个新元素的时候，就去调用新元素的hashcode()，能通过生成的这个key去物理地址上去找是否有这个值，如果没找到就直接存在这个位置上，如果在这个位置上已经有元素了，就将这两个元素进行比较，如果相同就不存放，如果不相同就散列到其他位置，解决了元素的重复问题，也大大的降低了equals()的调用次数，提高了运行效率。

#### 4.	hashcode和equals的联系
- 同一对象多次调用hashCode()方法返回的值应该总是是一样的
- 如果a.equals(b)为true，那么他们的hashcode一定相同
- 如果a.equals(b)为false,那么他们的hashcode不一定相同，但是应该让他们产生不一样的hashcode，有利于提高性能
