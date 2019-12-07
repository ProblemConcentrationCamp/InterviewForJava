# SingletonPattern(单例模式)

一个类只有一个实例，且该类能自行创建这个实例的一种模式

单例的 3 个特点
>1. 单例类只有一个实例对象
>2. 该单例对象必须由单例类自行创建
>3. 单例类对外提供一个访问该单例的全局访问点


## 单例的创建方式

1. 饿汉式
```java
/**
 * 饿汉式（静态变量）
 */
public class Singleton {

  private static Singleton singleton = new Singleton();

  private Singleton(){
  }
  public static Singleton getSingleton() {
    return singleton;
  }
}


/**
 * 饿汉式（静态常量）
 */
public class Singleton {

  private final static Singleton SINGLETON = new Singleton();

  private Singleton(){
  }

  public static Singleton getSingleton() {
    return SINGLETON;
  }
}

/**
 * 饿汉式（静态代码块）
 */
public class Singleton {

  private static Singleton singleton;

  static {
    singleton = new Singleton();
  }

  private Singleton(){
  }

  public static Singleton getSingleton() {
    return singleton;
  }
}

```

优点：   
写法简单, 在类装载的时候就完成实例化。避免了线程同步问题     
缺点：  
在类装载的时候就完成实例化, 如果系统不需要这个对象, 则会造成内存的浪费

2. 懒汉式
```java 

/**
 * 双重检测单例, 其他的实现方式，在并发下或多或少有一些问题
 */
public class Singleton {

  private volatile static Singleton singleton;

  static {
    singleton = new Singleton();
  }

  private Singleton(){
  }

  public static Singleton getSingleton() {
    if (singleton == null) {
      synchronized (Singleton.class) {
        if (singleton == null) {
          singleton = new Singleton();
        }
      }
    }
    return singleton;
  }
}
```
优点：  
线程安全；延迟加载；效率较高  
缺点：  
由于volatile关键字会屏蔽Java虚拟机所做的一些代码优化，略微的性能降低，但除非你的代码在并发场景比较复杂或者低于JDK6版本下使用，否则，这种方式一般是能够满足需求的


3. 静态内部类
```java
/**
 * Singleton 类被装载时并不会立即实例化, 只有在调用 getInstance 方法，才会装载 SingletonInstance 类，从而完成 Singleton 的实例化
 * 利用 JVM的 classloder 的机制来保证初始化 instance 时只有一个线程
 */
public class Singleton {

  private Singleton() {
  }

  private static class SingletonInstance {
    private final static Singleton SINGLETON = new Singleton();
  }

  public static Singleton getSingleton() {
    return SingletonInstance.SINGLETON;
  }
}
```

优点：  
避免了线程不安全，延迟加载，效率高。  
缺点：  
依旧不能解决在反序列化、反射、克隆时重新生成实例对象的问题。



4. 枚举模式
```java
public enum Singleton {
SINGLETON
}
```

优点：  
写法简单，不仅能避免多线程同步问题，而且还能防止反序列化、反射，克隆重新创建新的对象  
缺点：  
JDK 1.5之后才能使用。

5. 登记式单例--使用Map容器来管理单例模式(Spring的 Ioc 就是用的这种方式)
```java
public class SingletonManager {

  private static Map<String, Object> objectMap = new HashMap<>();

  private SingletonManager() {
  }

  public static void registerService(String key, Object instance) {
    if (!objectMap.containsKey(key)) {
      objectMap.put(key, instance);
    }
  }
  
  public static Object getService(String key) {
    return objectMap.get(key);
  }
}
```

优点：  
在程序的初始，将多种单例类型注入到一个统一的管理类中，统一管理多个实例，降低了用户的使用成本，也对用户隐藏了具体实现，降低了耦合度。  
缺点：  
繁琐