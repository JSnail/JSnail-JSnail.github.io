---
layout: post
title: 设计模式（一）之单例模式
categories: DesignMode
description: 单例模式
keywords: 单例, singleton
---

### 1.定义和使用场景
- 定义 

	保证一个类有且只有一个对象，并能向全局提供实例

- 使用场景

	确保某一个类有且只有一个对象，避免过多的创建对象消耗资源，例如工具类 网络请求库等等。

### 2.实现的关键点

1. 构造函数为private 不对外开放
2. 通过一个静态的方法返回单例类的对象
3. 确保单例类的对象有且只能有一个，注意多线程并发的问题
4. 确保单例类在反序列化的时候不被重新创建对象

### 3.示例

例如一个公司有很多的员工，很多的经理，但是老板只有一个，老板这个对象就可以使用单例的模式来实现

	
	//老板
	public class Boss extends BaseClass{
    	private static Boss boss = new Boss();

    	private Boss() {
    	}

    	 public static Boss getInstance() {
        	return boss;
    	}
	}
	
	//经理
	public class Manager extends  BaseClass {
	}
	//员工
	public class Staff extends  BaseClass {
	}
	//公司类，主要用于组装数据
	
	public class Company {
    private ArrayList<BaseClass> data = new ArrayList<>();

    public void setData(BaseClass baseClass) {
        data.add(baseClass);
    }

    public void show() {
        for (BaseClass baseClass : data) {
            System.out.println(baseClass.toString());
        }
      }
	}
	
	//测试类
	    public static void main(String[] args) {

        Company company = new Company();
        Boss boss1 = Boss.getInstance();
        Boss boss2 = Boss.getInstance();

        Manager manager = new Manager();
        Manager manager1 = new Manager();

        Staff staff = new Staff();
        Staff staff1 = new Staff();

        company.setData(boss1);
        company.setData(boss2);
        company.setData(manager);
        company.setData(manager1);
        company.setData(staff);
        company.setData(staff1);

        company.show();
    }
    
    //结果
    singleton.Boss@45ee12a7
	singleton.Boss@45ee12a7
	singleton.Manager@330bedb4
	singleton.Manager@2503dbd3
	singleton.Staff@4b67cf4d
	singleton.Staff@7ea987ac

从上面的代码中可以看出，Boss对象只能通过Boss.getInstance()的方式拿到对象，而Boss这个对象是一个静态对象，而且是在声明的时候就初始化，每次拿到的都是同一个对象；而其他的经理、员工对象没做处理，每次通过new的方式得到的对象都是不同的。

### 4.其他的实现方式

1. 饿汉模式

	就是上述示例中Boss类使用的方式，在静态对象声明的时候就初始化完成。

2. 懒汉模式

	顾名思义就是在使用的时候才回去创建对象示例如下:

		public class Boss extends BaseClass{
		    private static Boss boss ;
		
		    private Boss() {
		    }
		
		    public static Boss getInstance() {
		        if (null == boss){
		            boss = new Boss();
		        }
		        return boss;
		    }
		}
		
这种实现方式解决了当单例只有在使用的时候才会被创建的问题，比起饿汉模式节约了一定的资源，但是在启动的时候要慢一点，而且存在线程安全的问题，为了解决线程安全这个问题可以使用synchronized进行加锁

	 public static synchronized Boss getInstance() {
        if (null == boss){
            boss = new Boss();
        }
        return boss;
    }
但是会造成一个问题，就是每次调用的时候都会进行同步，造成不必要的开销。

3. Double Cleck Look (DCL)双重加锁模式

	DCL模式的优点是能够在保证使用的时候才初始化，又能保证线程安全，而且在后续调用的情况下不进行同步。

		public static Boss getInstance() {
        if (null == boss) {
            synchronized (Boss.class) {
                if (null == boss) {
                    boss = new Boss();
                }
             }
          }
         return boss;
        }

上述单例模式基本上能满足需求，但是会在某些高并发的情况下出现加锁失效的问题。主要原因是java编译器运行乱序执行，而boss = new Boss()这句代码并不是原子性的操作，会出现先去拿到boss，但是还未分配到内存空间的情况，从而初选DCL失效的问题。解决办法可以在方法上加上volatile关键字禁止指令重排序以及内存屏障。

4. 静态内部内单例模式

		public class Boss extends BaseClass {

    	private Boss() {
    	}

    	public static Boss getInstance() {
        return BossHolder.boss;
    	}
		 private static class BossHolder {
        private static final Boss boss = new Boss();
    	}
	
5. 枚举单例

		public enum Boss{
			INSTANCE;
		}
　　枚举类实例的创建是线程安全的，并且保证在任何情况下都是单例的，而上面的几种实现方式会在反序列化。


	　　将一个实例对象通过序列化的方式写到本地，再通过反序列化的方式读回来，即使该对象的构造方法是私有的。如果想避免这种情况出现那么就需要添加如下代码。

		private Object readResolve() throw ObjectStreamException{
		   return instance;
		}

	也就是控制在反序列化的时候返回自己，而不是重新生成一个新的对象。枚举不存在这个情况。

	
	
