原文出处：http://blog.csdn.net/yanbober/article/details/45338041

# **1. 概述**
建造者模式将客户端与包含多个组成部分的复杂对象的创建过程分离，客户端压根不用知道复杂对象的内部组成部分与装配方式，只需要知道所需建造者的类型即可。它关注如何一步一步创建一个的复杂对象，不同的具体建造者定义了不同的创建过程，且具体建造者相互独立，增加新的建造者非常方便，无须修改已有代码，系统具有较好的扩展性。

![builder建造者模式](http://img.blog.csdn.net/20160922213148310)

# 2. 建造者模式和抽象工厂模式的区别

其实，在建造者模式里有个指导者，由指导者来管理建造者，用户是与指导者联系的，指导者联系建造者最后得到产品。即建造模式可以强制实行一种分步骤进行的建造过程。而抽象工厂模式不具备最终的这个直接创建功能。建造者模式与工厂模式是极为相似的，总体上，建造者模式仅仅只比工厂模式多了一个“导演类”的角色。假如把这个导演类看做是最终调用的客户端，那么剩余的部分就可以看作是一个简单的工厂模式了。

与工厂模式相比，建造者模式一般用来创建更为复杂的对象，因为对象的创建过程更为复杂，因此将对象的创建过程独立出来组成一个新的导演类。

也就是说，工厂模式是将对象的全部创建过程封装在工厂类中，由工厂类向客户端提供最终的产品；而建造者模式中，建造者类一般只提供产品类中各个组件的建造，而将具体建造过程交付给导演类。由导演类负责将各个组件按照特定的规则组建为产品，然后将组建好的产品交付给客户端。

# **3. 四个要素**
概念： 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。建造者模式是一种对象创建型模式。

重点： 建造者模式结构重要核心模块：

- **Builder（抽象建造者）**
它为创建一个产品对象的各个部件指定抽象接口，在该接口中一般声明两类方法，一类方法是buildXXX()，它们用于创建复杂对象的各个部件；另一类方法是getXXX()，它们用于返回复杂对象。Builder既可以是抽象类，也可以是接口。

- **ConcreteBuilder（具体建造者）**
它实现了Builder接口，实现各个部件的具体构造和装配方法，定义并明确它所创建的复杂对象，也可以提供一个方法返回创建好的复杂产品对象。

- **Product（产品角色）**
它是被构建的复杂对象，包含多个组成部件，具体建造者ConcreteBuilder创建该产品的内部表示并定义它的装配过程。

- **Director（指挥者）**
指挥者又称为导演类，它负责安排复杂对象的建造次序，指挥者与抽象建造者之间存在关联关系，可以在其construct()建造方法中调用建造者对象的部件构造与装配方法，完成复杂对象的建造。客户端一般只需要与指挥者进行交互，在客户端确定具体建造者的类型，并实例化具体建造者对象，然后通过指挥者类的构造函数或者Setter方法将该对象传入指挥者类中。

**什么是复杂对象**
是指那些包含多个成员属性的对象，这些成员属性也称为部件或零件，如程序猿要会识字、会数学、会编程语言，会设计模式等等。

```java
package org.effectivejava.examples.chapter02.item02.builder;

public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int carbohydrate = 0;
        private int sodium = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}

```
# 4. 经典案例
## 4.1 AlertDialog

```java
AlertDialog dialog = new AlertDialog.Builder(this)
                .setIcon(R.mipmap.ic_launcher)
                .setTitle("标题")
                .setPositiveButton()
                .setNegativeButton()
                .create();
```

## 4.2 Notification

```java
Notification notification = new Notification.Builder(this)
                .setContentTitle("title")
                .setContentText("content")
                .setSmallIcon(R.mipmap.ic_launcher)
                .build();
```

## 4.3 Okhttp

```java
OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(10, TimeUnit.SECONDS)
                .readTimeout(5,TimeUnit.SECONDS)
                .writeTimeout(5,TimeUnit.SECONDS)
                .cache(new Cache(getCacheDir(),1024))
                .build();
```

```java
Request request = new Request.Builder()
                .url("url")
                .post(requestBody)
                .build();
```

## 4.4 Retrofit

```java
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("url")
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();
```