---
layout: post
title: 设计模式（三）之策略模式
categories: DesignMode
description: 策略模式
keywords: 状态模式, Stragety,stragety
---
### 1.定义和使用场景

- 定义

  策略模式定义了一系列的算法，并将所有的算法封装起来，而且可以使他们相互替换。策略模式让算法独立于使用他的客户而独立变化。

- 使用场景

  1.针对同一需求的不同处理方式，仅仅是具体行为有差别

  2.需要安全地封装多种同一类型的时候

  3.出现同一抽象类有多个子类，而又需要使用大量的if else或者switch-case的时候。

### 2.角色

- Context  用于操作上下文的环境
- Stragety  策略的抽象
- ConcreteStragetyA ConcreteStragetyB  具体的策略实现类

### 3.策略模式的具体实现

​	以成都本地出租、公交、地铁为例来计算价格计算，先用常规写法来

```java
public class PriceCalculate {

    private static int BUS = 1;
    private static int SUBWAY = 2;

    public static void main(String[] args) {
        PriceCalculate calculate = new PriceCalculate();
        System.out.println("公交10公里的价格 == " + calculate.choice(BUS, 10));
        System.out.println("地铁10公里的价格 == " + calculate.choice(SUBWAY, 10));
    }


    private int choice(int type, int km) {
        if (type == BUS) {
            return busPrice();
        } else if (type == SUBWAY) {
            return subwayPrice(km);
        }
        return 0;
    }

    /**
     * bus价格
     *
     * @return 总是返回2元
     */
    private int busPrice() {
        return 2;
    }

    /**
     * 地铁价格
     */
    private int subwayPrice(int km) {
        if (km <= 4) {
            return 2;
        } else if (km <= 8) {
            return 3;
        } else if (km <= 12) {
            return 4;
        }
        return 5;
    }
}
```

​	这样的写法就存在一些问题，比如如果要添加新的交通工具，就需要在choice()方法里面添加新的判断，还需要在类里面添加新的计算方法。

​	如果改用策略模式来写就可以如下的方式

```java
//抽象一个价格计算的接口
public interface PriceCalculate {
    int priceCalculate(int km);
}
```

​	再实现公交和地铁的价格计算方式

```java
//公交
public class BusStrategy implements PriceCalculate {
    @Override
    public int priceCalculate(int km) {

        return 2;
    }
}
//地铁
public class SubwayStrategy implements PriceCalculate {
    @Override
    public int priceCalculate(int km) {
        if (km <= 4) {
            return 2;
        } else if (km <= 8) {
            return 3;
        } else if (km <= 12) {
            return 4;
        } else if (km <= 18) {
            return 6;
        }
        return 10;
    }
}
```

最后在调用类中具体使用

```java
public class Main {
    PriceCalculate calculate;

    public static void main(String[] args) {
        Main main = new Main();
        main.setCalculate(new BusStrategy());
        System.out.println("公交10公里的价格 == " + main.getPrice(10));
        main.setCalculate(new SubwayStrategy());
        System.out.println("地铁10公里的价格 == " + main.getPrice(10));
    }

    public void setCalculate(PriceCalculate priceCalculate) {
        this.calculate = priceCalculate;
    }

    public int getPrice(int km) {
        return calculate.priceCalculate(km);
    }
}
```

​	如果想在添加一个出租车的计价方式只需要再实现一个出租车的类就好了。

​	通过对比上述两种方式明显能看出区别，前一种方法虽然实现简单，但是各种计算放都杂糅在一起，逻辑不清晰，后续的维护也相对麻烦；而后者通过接口的方式试下各种计算方式，逻辑清晰，增加减少乘坐的方式都很简单。

最后附上相关代码地址：

[策略模式](https://github.com/JSnail/DesignMode/tree/master/DesignMode/out/production/DesignMode/strategy)