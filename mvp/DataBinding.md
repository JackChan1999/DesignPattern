---
typora-copy-images-to: img
---
## 1. DataBinding

为什么使用databinding——解决数据传输问题

### 开启DataBinding

由于AndroidStudio已经默认集成了DataBinding，所以我们只需要将开关打开即可。在应用的build.grandle中添加开启配置

```gradle
android {
    ....
    dataBinding {
        enabled = true
    }
}
```

## 2. Data Binding 中的布局文件入门

### 2.1 编写 data binding 表达式
DataBinding 的布局文件与以前的布局文件有一点不同。它以一个 layout 标签作为根节点，里面包含一个 data 标签与 view 标签。view 标签的内容就是不使用 data binding 时的普通布局文件内容。例子如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```
在 data 标签中定义的 user 变量描述了一个可以在布局中使用的属性
```xml
<variable name="user" type="com.example.User"/>
```
在布局文件中写在属性值里的表达式使用 “@{}” 的语法。在这里，TextView 的文本被设置为 user 中的 firstName 属性。
```xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}"/>
```

### 2.2 数据对象
假设你有一个 plain-old Java object(POJO) 的 User 对象。
```java
public class User {
   public final String firstName;
   public final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
}
```
这个类型的对象拥有不可改变的数据(immutable)。在应用中，写一次之后永不变动数据的对象很常见。这里也可以使用 JavaBeans 对象：
```java
public class User {
   private final String firstName;
   private final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
   public String getFirstName() {
       return this.firstName;
   }
   public String getLastName() {
       return this.lastName;
   }
}
```
从 data binding 的角度看，这两个类是等价的。用于 TextView 的`android:text`属性的表达式`@{user.firstName}`，对于 POJO 对象会读取 firstName 字段，对于 JavaBeans 对象会调用 getFirstName() 方法。此外，如果 user 中有 firstName() 方法存在的话，也可以使用@{user.firstName}表达式调用。

### 2.3 绑定数据
1、数据绑定工具在编译时会基于布局文件生成一个 Binding 类。默认情况下，这个类的名字将基于布局文件的名字产生，把布局文件的名字转换成帕斯卡命名形式并在名字后面接上”Binding”。例如，上面的那个布局文件叫 main_activity.xml，所以会生成一个 MainActivityBinding 类。这个类包含了布局文件中所有的绑定关系（user 变量和user表达式），并且会根据绑定表达式给布局文件中的 View 赋值。

在 inflate 一个布局的时候创建 binding 的方法如下：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```
就这么简单！运行应用，你会发现测试信息已经显示在界面中了。

2、你也可以通过以下这种方式绑定view：
```java
MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater());
```
MainActivityBinding.inflate() 会填充 MainActivityBinding 对应的布局，并创建 MainActivityBinding 对象，把布局和MainActivityBinding对象绑定起来。

3、如果在 ListView 或者 RecyclerView 的 adapter 中使用 data binding，可以这样写：
```java
ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
//or
ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
```

### 2.4 事件处理
数据绑定允许你编写表达式来处理view分发的事件（比如 onClick）。事件属性名字取决于监听器方法名字。例如[View.OnLongClickListener](https://developer.android.com/reference/android/view/View.OnLongClickListener.html)有[onLongClick()](https://developer.android.com/reference/android/view/View.OnLongClickListener.html#onLongClick(android.view.View))的方法，因此这个事件的属性是android:onLongClick
。处理事件有两种方法：

- 方法引用绑定：在您的表达式中，您可以引用符合监听器方法签名的方法。当表达式的值为方法引用时，Data Binding会创建一个侦听器中封装方法引用和方法所有者对象，并在目标视图上设置该侦听器。如果表达式的值为null，数据绑定则不会创建侦听器，而是设置一个空侦听器。

- Lisenter 绑定：如果事件处理表达式中包含lambda表达式。数据绑定一直会创建一个监听器，设置给视图。当事件分发时，侦听器才会计算lambda表达式的值。

#### 方法引用绑定

事件可以直接绑定到处理器的方法上，类似于`android：onClick`可以分配一个Activity中的方法。与`View＃onClick`属性相比，方法绑定一个主要优点是表达式在编译时就处理了，因此如果该方法不存在或其签名不正确，您会收到一个编译时错误。 

方法引用绑定和监听器绑定之间的主要区别是，包裹方法引用的监听器是在数据绑定时创建的，包裹监听器绑定是在触发事件时创建的。如果您喜欢在事件发生时执行表达式，则应使用监听器绑定。 

要将事件分配给其处理程序，使用正常的绑定表达式，该值是要调用的方法名称。例如，如果数据对象有两个方法：

```java
public class MyHandlers {
    public void onClickFriend(View view) { ... }
}
```
像下面这样，绑定表达式可以为视图分配一个点击监听器：
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.Handlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```
请注意，表达式中的方法签名必须与监听器对象中的方法签名完全匹配。

#### Lisenter 绑定

Lisenter 绑定是在应用程序运行中事件发生时才绑定表达式。它们和方法引用绑定类似，但Listener 绑定允许运行时的任意数据绑定表达式。此功能适用于版本2.0及更高版本的Android Gradle插件。 

在方法引用绑定中，方法的参数必须与事件侦听器的参数匹配。在Listener 绑定中，只要返回值必须与Lisenter的预期返回值匹配（除非它期望void）。

例如，您可以有一个presenter类，它具有以下方法：
```java
public class Presenter {
    public void onSaveClick(Task task){}
}
```
然后，您可以将点击事件绑定到您的类中，如下所示：
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
  <data>
      <variable name="task" type="com.android.example.Task" />
      <variable name="presenter" type="com.android.example.Presenter" />
  </data>
  <LinearLayout 
    android:layout_width="match_parent" 
    android:layout_height="match_parent">
      <Button 
        android:layout_width="wrap_content" 
        android:layout_height="wrap_content"
        android:onClick="@{() -> presenter.onSaveClick(task)}" />
  </LinearLayout>
</layout>
```
侦听器由只允许由作为表达式的根元素的lambda表达式表示。当在表达式中使用回调时，数据绑定自动创建必要的侦听器并且为事件注册。当视图触发事件时，数据绑定将执行给定的表达式。与在正则绑定表达式中一样，在执行这些侦听器表达式时，仍然会获得数据绑定的空值和线程安全性。

注意，在上面的示例中，我们没有在ambda表达式中定义传递到 onClick（android.view.View）的视图参数。侦听器绑定为侦听器参数提供两个选择：忽略方法的所有参数和命名所有参数。如果您喜欢命名参数，可以在表达式中使用它们。例如，上面的表达式可以写成：
```
android:onClick="@{(view) -> presenter.onSaveClick(task)}"
```
如果你想要使用表达式中的参数，可以在这样使用：
```java
public class Presenter {
    public void onSaveClick(View view, Task task){}
}
```
```
android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```
您也可以使用具有多个参数的lambda表达式：
```java
public class Presenter {
    public void onCompletedChanged(Task task, boolean completed){}
}
```
```xml
<CheckBox 
    android:layout_width="wrap_content" 
    android:layout_height="wrap_content"
    android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```
如果正在侦听的事件返回类型不是void的值，则表达式也必须返回相同类型的值。例如，如果你想监听长点击事件，你的表达式应该返回布尔值。
```java
public class Presenter {
    public boolean onLongClick(View view, Task task){}
}
```
```
android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
```
如果由于空对象而无法计算表达式，数据绑定将返回该类型的默认Java值。例如，引用类型为null，int为0，boolean为false等。 如果需要使用带谓词（例如三元）的表达式，则可以使用void作为符号。
```
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```
#### 避免复杂侦听器

Listener表达式非常强大，可以使您的代码非常容易阅读。另一方面，包含复杂表达式的 Listener 又会使您的布局难以阅读和难以维护。这些表达式应该像从UI传递可用数据到回调方法一样简单。您应该在从侦听器表达式调用的回调方法中实现任何业务逻辑。 

一些专门的点击事件处理程序存在，他们需要一个属性，而不是android:onClick避免冲突。已创建以下属性以避免此类冲突。

| Class        | Listener Setter                          | Attribute             |
| :----------- | :--------------------------------------- | :-------------------- |
| SearchView   | setOnSearchClickListener(View.OnClickListener) | android:onSearchClick |
| ZoomControls | setOnZoomInClickListener(View.OnClickListener) | android:onZoomIn      |
| ZoomControls | setOnZoomOutClickListener(View.OnClickListener) | android:onZoomOut     |

##  3. Data Binding 中的布局文件进阶

###  import
1.1 data标签内可以有0个或者多个 import 标签。你可以在布局文件中像使用 Java 一样导入引用。
```xml
<data>
    <import type="android.view.View"/>
</data>
```
导入的类有两种用处，一种是用来定义变量，一种是在表达式中访问类中的静态方法和属性
现在 View 可以被在表达式中这样引用：
```xml
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```
1.2 当类名发生冲突时，可以使用 alias：
```xml
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```
现在，Vista 可以用来引用 com.example.real.estate.View ，与 View 在布局文件中同时使用。

1.3 导入的类型可以用于变量的类型引用和表达式中：
```xml
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List<User>"/>
</data>
```
> 注意：Android Studio 还没有对导入提供自动补全的支持。你的应用还是可以被正常编译，要解决这个问题，你可以在变量定义中使用完整的包名。

```xml
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```
1.4 导入的类也可以在表达式中使用静态属性/方法:

```xml
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```