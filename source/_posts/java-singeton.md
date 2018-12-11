title: "Java设计模式修炼之道之单例模式"
date: 2015-07-07 15:09:32
tags:
- Java
- 单例
- Singeton
---


单例模式顾名思义就是要保证某个类只被实例化一次，该模式常常用于窗口管理器、文件系统等代表那些本质上唯一的系统组件。由于要保证该类只能被实例化一次，所以就要求是该类自己创建自己的实例，因为如果能够别的类有能力来创建该类的实例的话就不能保证只有一个实例。为了使外界能够使用这个唯一的实例，单例类必须向外界提供自己的实例。创建单例的方式有以下几种:

<!-- more -->

### 饿汉式单例类

``` java
public class EagerSingleton {
	private static final EagerSingleton instance = new EagerSingleton();
	// 私有构造函数，不可省略，需要确保别的类不能通过构造函数创建该类的实例
	private EagerSingleton() {}
	public static EagerSingleton getInstance() {
		return instance;
	}
}

```


这种方式在类被加载时初始化实例，构造方法是private，所以外界不能直接创建该类的实例。这种方式创建单例类的优缺点如下:
优点:相对于后面介绍的懒汉式单例类来说节省了时间
缺点:
* 牺牲了空间，因为类一加载就需要创建单例
* 不能算是饿汉式特有的缺点，但是却是需要注意的是防止某些客户端通过反射机制设置构造方法的setAccessible(true),从而拥有可以直接创建实例的能力。测试程序如下:


``` java
public class EagerSingleton {
    private static final EagerSingleton instance=new EagerSingleton();
    //私有构造函数，不可省略，需要确保别的类不能通过构造函数创建该类的实例
    private EagerSingleton(){}
    public static EagerSingleton getInstance(){
                    return instance;
    }
    public static void main(String[] args) throws IllegalArgumentException, InstantiationException, IllegalAccessException,InvocationTargetException {
                    EagerSingleton es1=EagerSingleton.getInstance();
                    EagerSingleton es2=EagerSingleton.getInstance();
                    System.out.println(es1==es2);
                    for (java.lang.reflect.Constructor<?>c:es1.getClass().getDeclaredConstructors()) {
                                   c.setAccessible(true);
                                   EagerSingleton es3=(EagerSingleton) c.newInstance();
                                   System.out.println(es2==es3);
                    }
    }
}

```


可以看到输出结果如下：

```sh
true
false

```

这说明es3和es2是引用的不同的对象。

解决该问题的一个思路如下:

``` java
public class EagerSingleton {
	private static int count=0;
	private static final EagerSingleton instance=new EagerSingleton();
	//私有构造函数，不可省略，需要确保别的类不能通过构造函数创建该类的实例
	private EagerSingleton(){
		if(count==1){
			throw new RuntimeException("只能初始化一次");
		}
		count++;
	}
	public static EagerSingleton getInstance(){
		return instance;
	}
}

```


### 懒汉式单例类

介绍懒汉式单例类之前先介绍几种使用懒汉式单例类的几种常见的不恰当的例子：

* 如下这种懒汉式单例是错误频率比较高的一种，其错误的原因是该单例类只能在单线程的环境中正确使用。如果在多线程的环境中，如果线程A和线程B同时进入getInstance中，如果其中一个线程判断instance==null，进入了if的代码块正在或者还没有开始实例化LazySingleton或者实例化结束尚未把实例引用赋给instance，此时线程B就会再次进入if代码块，造成系统中有超过一个的单例类实例


``` java
public class LazySingleton {
	private static LazySingleton instance = null;
	private LazySingleton() {
	}
	public static LazySingleton getInstance() {
		if (instance == null) {
			instance = new LazySingleton();
		}
		return instance;
	}
}


```


* 同步化getInstance方法，这样做可以保证是单例的，但是由于每一次获取实例都只能有一个方法进入getInstance方法，所以效率相对比较低。但是总的来说这是一个正确的单例模式实现方式。

``` java
public static synchronized LazySingleton getInstance()  

```

* 所谓的"双重检查加锁"，这种方式的代码如下，可以说很大程度上这种方式被一些人误认为是解决第2条效率低的比较好的方式，因为只有在第一次获取实例的时候才会进入加锁的代码块，以后就再不用加锁了。可是需要注意的是这种双重检查加锁的机制在Java中是行不通的，原因如下:
对象初始化和引用赋值的顺序是不可预料的
这句话怎么理解呢，在这里new LazySingleton()的过程中会先为对象分配内存空间和对象属性的赋值，之后就可将引用地址传递给instance变量，可是这个时候对象的初始化过程有可能并没有完成，这个时候如果另外一个线程进入getInstance就会得到一个并没有完成初始化的状态不正确的对象，从而造成崩毁，这个地方需要特别注意。


``` java
public class LazySingleton {
	private static LazySingleton instance=null;
	private LazySingleton(){}
	public static LazySingleton getInstance(){
		if(instance==null){
			synchronized (LazySingleton.class) {
				if(instance==null){
					instance=new LazySingleton();
				}
			}
		}
		return instance;
	}
}


```


### 类级内部类式单例类

上面的懒汉式和饿汉式实现方式各有利弊，类级内部类单例类可以说兼顾了两者的优点，可谓是单例模式最好的实现方式之一。
先来看看什么是类级内部类，类级内部类是指由static修饰的成员式内部类，相当于类的static成分，它的对象与外部类间不存在依赖关系，可以直接创建。类级内部类可以定义静态方法，静态方法中只能够引用外部类中的静态成员方法或者成员变量。类级内部类相当于其外部类的成员，只有在第一次被使用的时候才会被加载。

``` java
public class StaticInnerClassSingleton {
	private StaticInnerClassSingleton() {}
	private static class SingletonHolder{
		//静态初始化，由JVM来保证线程安全
		private static StaticInnerClassSingleton instance=new StaticInnerClassSingleton();
	}
	public static StaticInnerClassSingleton getInstance(){
		return SingletonHolder.instance;
	}
}

```


上面的代码中同步已经由JVM执行了，JVM在以下情况下会进行同步控制。

* 由静态初始化器(静态字段上或static{}块中的初始化器)初始化数据时
* 访问final字段时
* 在创建线程之前创建对象时
* 线程可以看见它将要处理的对象时


这种单例模式只有在第一次调用getInstance的时候读取SingletonHolder.instance时初始化，同时由于是SingletonHolder的静态字段，所以JVM会保证它的线程安全性。


### 枚举式单例类

按照《高效Java 第二版》中的说法：单元素的枚举类型已经成为实现Singleton的最佳方法。用枚举来实现单例非常简单，只需要编写一个包含单个元素的枚举类型即可。


``` java
public enum Singleton {
    /**
     * 定义一个枚举的元素，它就代表了Singleton的一个实例。
     */
    uniqueInstance;
    /**
     * 单例可以有自己的操作
     */
    public void singletonOperation(){
        //功能处理
    }
}

```

使用枚举来实现单实例控制会更加简洁，而且无偿地提供了序列化机制，并由JVM从根本上提供保障，绝对防止多次实例化，是更简洁、高效、安全的实现单例的方式。