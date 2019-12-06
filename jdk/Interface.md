# 2. 接口里面能否有具体的实现

1. static
```java 
public interface MyInteface {

    static String sayHello() {
        return "hello";
    }
}
```

2. default
```java
public interface MyInteface {

    default String sayHi() {
        return "hi";
    }
}
```
