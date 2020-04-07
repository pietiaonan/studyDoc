1. 最常用且简单的方式，饿汉式

```java
public class Singleton{
	private static Singleton INSTANCE = new Singleton();
	private Singleton(){};
	public static Singleton getInstance(){
		return INSTANCE;
	}
}

```

2. 注重效率而且多线程安全情况下，懒汉式  懒加载
   双重判断加volatile 

```java
public class Singleton{
    private static volatile Singleton INSTANCE ;
    private Singleton(){};
    public static Singleton getInstance(){
        if(INSTANCE==null){
            synchronized (Singleton.class){
                if(INSTANCE==null){
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}

```
3. 静态内部类的方式，使用时加载，而且保证线程安全
```java
public class Singleton{
    private Singleton(){};
    public static Singleton getInstance(){
        return Inner.INSTANCE;
    }
    private static class Inner{
        private static Singleton INSTANCE = new Singleton();
    }
}
```
4. effiective java 推荐，简单直接，且jvm保证线程安全， **而且enum无构造方法，保证无法通过反射方式获取新的实例，以上三种不能保证  ,java 通过编译器和 JVM 联手来防止enum 产生超过一个class：不能利用 new、clone()、de-serialization、以及 Reflection API 来产生enum 的 instance**



```java
public enum  Singleton{
    INSTANCE;
    /**
     * other method
     */
    Singleton getInstance(){
        return INSTANCE;
    }
}
```

注意

- **1.4及更早之前的版本 volatile加双重判断的方式会家锁失效，不安全**
- **1.2及之前的版本，单例的对象会被jvm垃圾回收，造成可能单例产生的对象不是同一个**

为什么单例不能被回收

- 虚拟机栈（栈桢中的本地变量表）中的引用的对象。
- **方法区中的类静态属性引用的对象。**
- 方法区中的常量引用的对象。
- 本地方法栈中JNI的引用的对象。

java中单例模式创建的对象被自己类中的静态属性所引用，符合第二条，因此，单例对象不会被jvm垃圾收集。

vm卸载类的判定条件如下：

- **该类所有的实例都已经被回收，也就是java堆中不存在该类的任何实例。**
- 加载该类的ClassLoader已经被回收。
- 该类对应的java.lang.Class对象没有任何地方被引用，无法在任何地方通过反射访问该类的方法。

只有三个条件都满足，jvm才会在垃圾收集的时候卸载类。显然，单例的类不满足条件一