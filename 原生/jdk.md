
# 1. Java lambda表达式 和 方法引用的区别

Java lambda 表达式是一个语法糖, 底层是通过内部类实现的, 把我们的方法实现包装在类中而已

区别: 

>1. lambda 表达式可以捕获 其所在的作用域的变量, 而方法引用只能通过参数的形式传过去
```java 

public static void main(String[] args) {

    int num = 123;
    Thread t = new Thread(() -> {
        // 在 lambda 表达式的内部可以方法作用域内的变量, 只要这个变量是 Effectively final
        // effectively final 也是 Jdk8 新增的一个功能: 只有这个变量声明并赋值了,
        // 在其所在的作用域内(这里就是 main 方法体内), 没有再被改变过, 他就是 effectively final的。
        // 相对应的如果他在后续被修改过, 他就不是 effectively final
        System.out.println(num);
    });
    t.start();
}

```
>2. lambda 只适合比较简单的逻辑, 如果逻辑很复杂，可读性会很差

* 如何选择：

逻辑简洁用 lambda, 否则选择方法引用。 《Effective Java》 中说到：
>1. 优先使用 lambda, 而不是匿名内部类
>2. 优先使用 方法引用，而不是 lambda 表达式


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