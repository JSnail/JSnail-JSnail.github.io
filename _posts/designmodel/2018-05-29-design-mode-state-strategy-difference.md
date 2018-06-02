---
layout: post
title: 设计模式之状态模式和策略模式的区别
categories: DesignMode
description: 状态模式和策略模式的区别
keywords: 状态模式, 策略模式,state,dtrategy

---

### 1.前言

​	前面说了策略模式和状态模式两种设计模式，他们的行为有些类似，因此单独开一片来分析下两者的关系

### 2.分析

​	从本质上来看，两种设计模式都是干去耦合这件事，就是把要做什么，和具体做什么拆分开来。举个伪代码的🌰

```java
//经理每天要干的主要事情
class 经理{
    
   void 吃饭(){}
    
   void 上班(){}
    
   void  睡觉(){}
    
}
```

但是工作是有细分的，然后拆分为如下的方式

```java
class 经理{
    
   void 吃饭(){}
    
   void service(){}
    
   void android(){}
    
   void ios(){}
        
   void 睡觉(){}
}
```

把工作细分下来之后发现方法变多了，再分析发现具体做什么事取决于工作的类型，然后我们就可以抽象出一个接口表示工作这件事

```java
interface 工作{
    void 干啥();
}
//然后我们根据不同的工作类型去做实现对应的类

class Service implements 工作{
   void 干啥(){
        //写接口
    }
}

class Android implements 工作{
   void 干啥(){
        //写android架构
    }
}
class Ios implements 工作{
   void 干啥(){
        //写IOS架构
    }
}
```

以上可以看出工作只需要知道调用干啥就好了，不用去管是什么工作，然后我们就能把经理的工作的方法改一下

```java
class 经理{
    
   void 吃饭(){}
   
   void 上班(工作 要做的工作){
        要做的工作.干啥();
    }
        
   void 睡觉(){}
}
```

然后经理到公司了，开始工作了

```java
main(){
    经理  经理 = new 经理();
    //上午
    Service service = new Service();
    经理.工作(service);
    //下午
    Android android  = new Android();
    经理.工作(android);
    //加班
    Ios ios = new Ios();
    经理.工作(ios);
    
}
```

 	以上的就是策略模式，完全依赖于传入的对象进行相关的操作，划重点！是完全依赖于外部传入的对象！

​	但是如果传入的对象本身就有呢，经理是在是沉迷于工作，做啥都离不开工作，然后他一天的事情就变成这样了

```java
class 经理{
    
   void 吃饭(工作 吃饭时候的工作){
        吃饭时候的工作.看资料();
    }
    
   void 上班(工作 上班时候的工作){
        上班时候的工作.写代码();
    }
    
   void 睡觉(工作 睡觉之前的工作){
        睡觉之前的工作.整理();
    }
}


class 经理{
    工作 工作 = new Ios();
    
   void 吃饭(){
        工作.看资料();
    }
    
  void 上班(){
    	工作.写代码();
    }
    
  void  睡觉(){
       	工作.整理();
    }
}
```

但是要修改工作的内容呢，添加一个方法就好了

```java
void 工作切换(工作 传入的工作){
    工作= 传入的工作;
}
```

然后最后的调用就成了

```java
void main(){
    经理 经理 = new 经理();
    //早上
    经理.吃饭();
    //下午
    Android android = new Android();
    经理.工作切换(android);
    经理.工作();
}
```

这就叫状态模式

最后总结下，状态模式就像是有了每天要做的任务，不关心是谁传进来的任务；而策略模式是我只提供一个入口给你，你要做啥你自己传。一个在内部操作，一个是依赖外部的传入。