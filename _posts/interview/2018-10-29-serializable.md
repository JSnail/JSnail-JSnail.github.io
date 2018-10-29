---
layout: post
title: Serializable
categories: interview
description: Serializable
keywords: Serializable

---

### Serializable

​	Serializable是java提供的一种序列化接口，是一个空接口，目的是为对象提供标准的序列化和反序列化操作，实现Serializable非常的简单，只需要实现Serializable和serialVersionUID就好了，甚至SerialVersionUID实现也可以

```java
public class Test implements Serializable{
    private static final  long  serialVersionUID =123123213122L
       
    public int age;
    public String name;
       ...... 
}
```

Serializable序列化过程非常简单，基本上由系统自己动完成了，反序列化也非常简单，使用ObjectOutputStream和ObjectInputStream就能轻松的实现

```java
//序列化过程  
try {
            Test test = new Test();
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("test.txt"));
            objectOutputStream.writeObject(test);
            objectOutputStream.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
//反序列化过程
 try {
            ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream("test.txt"));
            Test test = (Test)inputStream.readObject();
            inputStream.close();
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

```

​	上文有提到不使用serialVersionUID也能实现序列化也能成功，那么serialVersionUID的作用是什么呢？当系统进行序列化的时候会把serialVersionUID写入文件或者其他什么当中，反序列化的时候就会去检测文件中的serialVersionUID，判断和当前类的值是否一致，如果一致则表示和当前类的版本是一致的，就能序列化成功；否则就说明当前类有发生变动，例如增减了一些成员变量什么的，这个时候就会反序列化失败出现crash。

​	这里就提现出serialVersionUID的作用了，当我们自定义他的值的时候，在升级程序增加成员变量之后也能反序列化成功，只不过需要多考虑一个情况就是，当之前存在的成员变量类型发生改变，或者类名发生改变的时候，虽然serialVersionUID一直，但是序列化也还是会失败。

​	

