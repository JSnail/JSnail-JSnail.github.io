---
layout: post
title: 接口和抽象类
categories: interview
description: 接口和抽象类
keywords: 接口,抽象类,interface,abstract
---

### 接口和抽象类

1. 接口(interface)

   接口类会产生一个完全抽象的类，但是它能被向上转型为多种基类的类型，接口有以下几种特点

   - 接口中的可以包含成员变量，但是他们默认为public static  final，其他的修饰符编译器会提示错误。
   - 接口不能implements接口，但是可以extend
   - 在java8以前接口中不能实现方法，但是在java 8之后可以用static或者default修饰
   - 接口不能实例化，只能通过接口的实现类创建实例
   - 接口支持多继承

2. 抽象类(abstract)

   抽象类使用abstract关键字来修饰。如果一个类中有被abstract修饰的抽象方法，该类一定要被定义为抽象类。使用抽象类有以下几点需要注意：

   - 抽象类不能通过new去创建实例
   - 抽象类中可以没有抽象方法
   - 如果继承一个抽象类，必须实现抽象类中的所有抽象方法，如果不想实现那么子类也一定要是抽象方法。




