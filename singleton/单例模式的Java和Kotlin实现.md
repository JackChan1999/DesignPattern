## 饿汉式单例

java实现

```java
public class PlainOldSingleton {
    private static PlainOldSingleton INSTANCE = new PlainOldSingleton();

    private PlainOldSingleton(){
        System.out.println("PlainOldSingleton");
    }

    public static PlainOldSingleton getInstance(){
        return INSTANCE;
    }
}
```

kotlin实现

```kotlin
object PlainOldSingleton {

}
```

## 非线程安全的懒汉式单例

java实现

```java
public class LazyNotThreadSafe {
    private static LazyNotThreadSafe INSTANCE;

    private LazyNotThreadSafe(){}

    public static LazyNotThreadSafe getInstance(){
        if(INSTANCE == null){
            INSTANCE = new LazyNotThreadSafe();
        }
        return INSTANCE;
    }
}
```

kotlin实现

```kotlin
class LazyNotThreadSafe {

    companion object{
        val instance by lazy(LazyThreadSafetyMode.NONE) {
            LazyNotThreadSafe()
        }

        //下面是另一种等价的写法, 获取单例使用 get 方法
        private var instance2: LazyNotThreadSafe? = null

        fun get() : LazyNotThreadSafe {
            if(instance2 == null){
                instance2 = LazyNotThreadSafe()
            }
            return instance2!!
        }
    }
}
```

## 线程安全的懒汉式单例

java实现

```java
public class LazyThreadSafeSynchronized {
    private static LazyThreadSafeSynchronized INSTANCE;

    private LazyThreadSafeSynchronized(){}

    public static synchronized LazyThreadSafeSynchronized getInstance(){
        if(INSTANCE == null){
            INSTANCE = new LazyThreadSafeSynchronized();
        }
        return INSTANCE;
    }
}
```

kotlin实现

```kotlin
class LazyThreadSafeSynchronized private constructor() {
    companion object {
       private var instance: LazyThreadSafeSynchronized? = null

        @Synchronized
        fun get(): LazyThreadSafeSynchronized{
            if(instance == null) instance = LazyThreadSafeSynchronized()
            return instance!!
        }
    }
}
```

## DoubleCheck

java实现

```java
public class LazyThreadSafeDoubleCheck {
    private static volatile LazyThreadSafeDoubleCheck INSTANCE;

    private LazyThreadSafeDoubleCheck(){}

    public static LazyThreadSafeDoubleCheck getInstance(){
        if(INSTANCE == null){
            synchronized (LazyThreadSafeDoubleCheck.class){
                if(INSTANCE == null) {
                    //初始化时分为实例化和赋值两步, 尽管我们把这一步写成下面的语句,
                    // 但Java虚拟机并不保证其他线程『眼中』这两步的顺序究竟是怎么样的
                    INSTANCE = new LazyThreadSafeDoubleCheck();
                }
            }
        }
        return INSTANCE;
    }
}
```

kotlin实现

```kotlin
class LazyThreadSafeDoubleCheck private constructor(){
    companion object{
        val instance by lazy(mode = LazyThreadSafetyMode.SYNCHRONIZED){
            LazyThreadSafeDoubleCheck()
        }

        private @Volatile var instance2: LazyThreadSafeDoubleCheck? = null

        fun get(): LazyThreadSafeDoubleCheck {
            if(instance2 == null){
                synchronized(this){
                    if(instance2 == null)
                        instance2 = LazyThreadSafeDoubleCheck()
                }
            }
            return instance2!!
        }
    }
}
```

## 静态内部类单例

java实现

```java
public class LazyThreadSafeStaticInnerClass {

    private static class Holder{
        private static LazyThreadSafeStaticInnerClass INSTANCE = new LazyThreadSafeStaticInnerClass();
    }

    private LazyThreadSafeStaticInnerClass(){}

    public static LazyThreadSafeStaticInnerClass getInstance(){
        return Holder.INSTANCE;
    }
}
```

kotlin实现

```kotlin
class LazyThreadSafeStaticInnerObject private constructor(){
    companion object{
        fun getInstance() = Holder.instance
    }

    private object Holder{
        val instance = LazyThreadSafeStaticInnerObject()
    }
}
```