---
layout: post
title: 设计模式（二）之单例模式
categories: DesignMode
description: 状态模式
keywords: 状态模式, state
---
### 1.定义和使用场景
- 介绍

	状态模式实际上是把对象的行为封装起来，每一个对象都有一个相同的抽象状态基类，通过直接操作对象达到目的。
- 定义

	当一个对象的内在状态改变的时候运行改变其行为，这个对象看起来像	是改变了其类
- 使用场景

	1. 一个对象的行为取决于他的状态，并且他必须在运行时根据他的状 	态来改变他的行为。
	2. 代码中包含大量的if else

### 2.角色

- Context: 面向使用者的类，定义他需要的方法，维护state的对象以	及他的当前状态。
	 State: 抽象状态类或者状态接口，定义一个或者一组接口，表示该状态	下的行为。
- ConcreteStateA、ConcreteStateB: 具体状态类

### 3.示例

还是沿用上一篇的角色来说。先将工作分为带队研发和普通开发工作，经理只做技术攻关，不做普通的编码工作，普通员工进行编码，但是做不了技术攻关

- ConcreteManager 经理类 负责带队研发项目
- ConcreteStaff 员工类 负责具体编码
- WorkContent 工作内容类 
- WorkRole 工作角色类

```java
/**
   * 工作类容
   */
  public interface WorkContent {

      void solveProblems();

      void coding();

  }


  /**
   * 工作角色
   */
  public interface WorkRole {

      void isManager();

      void isStaff();
  }


```



```java
/**
 * 经理
 */
public class ConcreteManager implements WorkContent {

    @Override
    public void solveProblems() {
        System.out.println("进行技术攻关");
    }

    @Override
    public void coding() {

    }
}
```

  ```java
  /**
   * 普通员工
   */
  public class ConcreteStaff implements WorkContent {
      @Override
      public void solveProblems() {

      }

      @Override
      public void coding() {
          System.out.println("进行日常代码编写");
      }
  }
  ```

  ​	

  ```java
  /**
   * 公司 -- 角色逻辑控制类
   */
  public class CompanyController implements WorkRole {

      private WorkContent workContent;

      @Override
      public void isManager() {
          setWorkContent(new ConcreteManager());
          System.out.println("我是经理");
      }

      @Override
      public void isStaff() {
          setWorkContent(new ConcreteStaff());
          System.out.println("我是普通coder");
      }

      private void setWorkContent(WorkContent workContent) {
          this.workContent = workContent;
      }

      public void solveProblems() {
          workContent.solveProblems();
      }

      public void coding() {
          workContent.coding();
      }
  }
  ```

  ```java
  public class Main {
      public static void main(String[] args) {
          CompanyController company = new CompanyController();
          //设置经理
          company.isManager();
          company.solveProblems();
          company.coding();//不会生效

          //设置员工
          company.isStaff();
          company.solveProblems();//不会生效
          company.coding();

      }
  }
  ```

  	通过以上代码可以看到，我们创建了2个接口，一个用于表示工作中需要做的所有工作，并且有他的实现类分别为ConcreteManager和ConcreteStaff；另外一个接口是角色类，相当于普通判断中的if else。当当前传入的角色是经理的时候，只有solveProblems()方法有效果，而当传入的是普通员工的角色的时候就只有 coding()方法起作用。同一个操作比如是coding()，在经理状态下不执行，在员工状态下才执行，也就是操作对象内部的属性决定了外部操作的结果。

​	状态模式就将这些行为封装成一个对象，在进行操作的时候直接传入需要操作的对象，从而达到不同状态操作不同对象的目的，去除了很多的if else。

最后附上相关代码的Github地址

[状态模式](https://github.com/JSnail/DesignMode/tree/master/DesignMode/out/production/DesignMode/state)