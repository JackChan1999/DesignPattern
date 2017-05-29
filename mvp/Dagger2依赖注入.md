---
typora-copy-images-to: img
---
Dagger2系列：https://alleniverson.gitbooks.io/android-opensource-framework/content/inject/

## 1. Dagger2

Dagger2是Dagger的升级版，是一个依赖注入框架，是解决Android或java中依赖注入的一个类库（DI类库）。

依赖注入是面向对象编程的一种设计模式，其目的是为了降低程序耦合，这个耦合就是类之间的依赖引起的。

依赖注入的几种方式

- 接口注入
- set()方法注入
- 构造方法注入
- Java注解

在Dagger2中用的就是最后一种注入方式，通过注解的方式，将依赖注入到宿主类中

### 1.1 什么是Dagger2?

Dagger是为Android和Java平台提供的在编译时进行依赖注入的框架。
编译时：编译时生成代码（rebulid），我们完成所需对象的注入。（假设使用反射，应该是什么时候起作用?）。

### 1.2 为什么使用Dagger2?

Dagger2解决了基于反射带来的开发和性能上的问题。

### 1.3 做什么工作？

Dagger2主要用于做界面和业务之间的隔离

### 1.4 引入配置

添加dagger2的依赖
```gradle
compile 'com.google.dagger:dagger:2.11'
```
编译时生成代码的插件配置（android-apt）

- project的gradle中添加

```gradle
buildscript {
		dependencies {
  			classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
		}
}
```
- apt插件的使用

modle的gradle中添加

```gradle
apply plugin: 'com.neenbedankt.android-apt'
```
- 关联Dagger2

```gradle
dependencies {
	apt 'com.google.dagger:dagger-compiler:2.11'
}
```
以上是gradle2.2以下的配置，由于android-apt在gradle2.2以上已经不再使用，所以gradle2.2以上的配置如下

Android Gradle

```gradle
// Add Dagger dependencies
dependencies {
  compile 'com.google.dagger:dagger:2.x'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.x'
}
```

If you're using classes in `dagger.android` you'll also want to include:

```gradle
compile 'com.google.dagger:dagger-android:2.x'
compile 'com.google.dagger:dagger-android-support:2.x' // if you use the support libraries
annotationProcessor 'com.google.dagger:dagger-android-processor:2.x'
```

### 1.5 操作步骤

![1495956020515](img/1495956020515.png)

```java
// 第一步
public class MainActivity extends AppCompatActivity{
	@Inject
	MainActivityPresenter presenter;
}

// 第二步

@Module
public class  MainActivityModule{
  MainActivity activity;
  
  public MainActivityModule(MainActivity activity){
  	this.activity = activity;  
  }
  
  @provides
  MainActivityPresenter provideMainActivityPresenter(){
    return new MainActivityPresenter(activity);
  }
}

// 第三步
// Component，建立Activity和Module之间的关系
// in(MainActivity activity)

@Component(module = MainActivityModule.class)

public interface MainActivityComponent{
  void inject(MainActivity activity);
}
// 第四步 build rebuild

// 第五步 注入presenter对象
DaggerMainActivityComponent component = (DaggerMainActivityComponent)DaggerMainActivityComponent.builder()
	.mainActivityModule(new MainActivityModule(this))
	.build();
component.in(this);
presenter.login();
```

## 2. 深入解析

在操作中会使用到@Inject、@Module、@Provides、@Conponent注解，那么他们分别在完成什么工作?

![](https://alleniverson.gitbooks.io/android-opensource-framework/content/inject/img/injection.jpg)

| 注解            | 功能说明                                     |
| :------------ | :--------------------------------------- |
| @Inject       | 使用在类的构造方法上或属性上，当看到某个类被@Inject标记时，就会到他的构造方法中，如果这个构造方法也被@Inject标记的话，就会自动初始化这个类，从而完成依赖注入 |
| @Conponent    | 用来将@Inject和@Module联系起来的桥梁，从@Module中获取依赖并将依赖注入给@Inject。一个接口或者抽象类，在这个接口中定义了一个inject()方法，rebuild一下项目，会生成一个以Dagger为前缀的Component类。有modules和dependencies两个属性分别表示Component关联的Module和依赖的Component |
| @Subcomponent | 子组件                                      |
| @Module       | 用来提供依赖，为那些没有构造函数的类提供依赖，比如第三方类库，系统类。里面定义一些用@Provides注解的以provide开头的方法，这些方法就是所提供的依赖，Dagger2会在该类中寻找实例化某个类所需要的依赖 |
| @Provides     | Module中的创建类实例方法用Provides进行标注。方法名可以是任意的有意义的名字，Component会根据返回值来提供依赖 |
| @Qualifier    | 通过自定义Qulifier，可以告诉Dagger2去需找具体的依赖提供者。 帮助我们去为相同接口的依赖创建“tags” |
| @Scope        | 在作用域内保持单例，可以直接理解为单例即可。同属于一个作用域的时候会公用一个相同的实例 |
| @MapKey       | 这个注解用在定义一些依赖集合                           |
| @Singleton    | @Singleton并不是我们设计模式中的单例模式，而是Dagger2中为了保持一个产品在一个Scope中只有一个对象的标签 |
| Lazy          | 懒加载，延迟实例化对象，在需要的时候，才会进行实例化               |
### 2.1 Inject

```java
//用这个标注标识是一个连接器
@Component()
public interface MainActivityComponent {
//这个连接器要注入的对象。这个inject标注的意思是，我后面的参数对象里面有标注为@Inject的属性，这个标注的属性是需要这个连接器注入进来的。也就是我们的使用方
void inject(MainActivity activity);
}
```

### 2.2 Component

Component需要引用到目标类的实例，Component会查找目标类中用Inject注解标注的属性，查找到相应的属性后会接着查找该属性对应的用Inject标注的构造函数，剩下的工作就是初始化该属性的实例并把实例进行赋值

一端连接目标类，另一端连接目标类依赖实例，它把目标类依赖实例注入到目标类中。Module是一个提供类实例的类，所以Module应该是属于Component的实例端的（连接各种目标类依赖实例的端），Component的新职责就是管理好Module，Component中的modules属性可以把Module加入Component，modules可以加入多个Module

定义好Component接口后，rebuild一下项目，Android Studio会自动为我们生成Component的实现类，该实现类以Dagger开头，以Component的接口名结尾

添加多个Module有两种方法

- 在Component中的modules属性中注入多个Module

   ```
   @Component(modules={ModuleA.class,ModuleB.class,...})
   public interface FruitComponent{
       ...
   }
   ```

- 在Component中的dependencies属性中注入多个依赖的Component

   ```
   @Component(dependencies={ComponentA.class,ComponentB.class,...}) 
   public interface FruitComponent{
       ...
   }

   ```

- 在Module中注入多个Module

   ```
   @Module(includes={ModuleA.class,ModuleB.class,...})
   public class FruitModule{
       ...
   }
   ```

#### 组织Component的方式

- 依赖方式，使用Component的dependencies属性
- 包含方式，使用@SubComponent
- 继承方式，extends

组件依赖

```java
@Component(
	dependencies = AppComponent.class,
	modules = SplashActivityModule.class
)
```

```java
@Singleton
@Component(modules = AndroidModule.class)
public interface ApplicationComponent {
    void inject(DemoApplication application);
    LocationManager getLocationManager();
    Context getContext();
}
```

```java
//标注 是一个模块 ，里面实现了一些提供的实例构造，后面使用Component来将这些连接起来。
@Module
public class AndroidModule {
    private final DemoApplication application;

    public AndroidModule(DemoApplication application) {
        this.application = application;
    }

    @Provides
    @Singleton
    Context ApplicationContext() {
        return application;
    }

    @Provides
    @Singleton
    LocationManager provideLocationManager() {
        return (LocationManager) application.getSystemService(LOCATION_SERVICE);
    }
}
```

```java
@PerActivity
@Component(dependencies = ApplicationComponent.class,modules ={MyModule2.class,ModuleB.class,ModuleA.class,MyModule.class})
public interface MainActivityComponent {
    void inject(MainActivity mainActivity);
    Set<String> strings();
    Map<String, Long> longsByString();

    Map<MyModule2.MyEnum, String> StringsByMyEnum();
}
```

### 2.3 Module

项目中使用到了第三方的类库，第三方类库又不能修改，所以根本不可能把Inject注解加入这些类中，这时我们的Inject就失效了。

Module通过Component的modules属性与Component进行关联，一个Component类可以包含多个Module类，用来提供依赖当我们调用Component注入实例时，Component会获取Module中@Provides标记的方法提供的实例，赋值给@Inject标记的属性，实现依赖注入。

Module里面的方法基本都是创建类实例的方法。

Component会首先从Module维度中查找类实例，若找到就用Module维度创建类实例，并停止查找Inject维度。否则才是从Inject维度查找类实例。所以创建类实例级别Module维度要高于Inject维度。

### 2.4 Provides

Module中的创建类实例方法用Provides进行标注，Component在搜索到目标类中用Inject注解标注的属性后，Component就会去Module中去查找用Provides标注的对应的创建类实例方法，这样就可以解决第三方类库用dagger2实现依赖注入了

### 2.5 Qualifier

限定符：解决依赖注入迷失问题

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface A {}
```

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface B {}
```

我们定义了两个注解，@A和@B，他们都是用@Qulifiier标注的

再看看我们的Module

```java
@Module
public class SimpleModule {
    @Provides
    @A
    Cooker provideCookerA(){
        return new Cooker("James","Espresso");
    }
    @Provides
    @B
    Cooker provideCookerB(){
        return new Cooker("Karry","Machiato");
    }
    }
```

再看看具体的使用

```java
public class ComplexMaker implements CoffeeMaker {
    Cooker cookerA;
    Cooker cookerB;
    @Inject
    public ComplexMaker(@A Cooker cookerA,@B Cooker cookerB){
        this.cookerA = cookerA;
        this.cookerB = cookerB;
    }
    @Override
    public String makeCoffee() {
        return cooker.make();
    }
    }

cookerA.make();//James make Espresso
cookerB.make();//Karry make Machiato
```
### 2.6 Scope作用域

- @Scope对某类注解后，其作用的核心应该是注入器的控制，说白了就是，在注入实例时，是控制注入器调用缓存的实例还是重新实例。
- 自定义注解限定注解作用域
- 更好的管理Component之间的组织方式，Component组织方式：依赖方式dependencies、包含方式SubComponent
- 更好的管理Component与Module之间的匹配关系

```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface PerActivity {
}
```

```java
@PerActivity
@Component(modules = {ActivityModule.class})
public interface ActivityComponent {

    Activity getActivity();
}
```
### 2.7 Singleton

@Singleton是@Scope的一个特例，或者是说是@Scope的一种实现方式。

### 2.8 SubComponent

子组件，子组件是继承和扩展父组件的对象的组件

声明子组件，使用@Subcomponent注解子组件，必须声明其xxComponent.Builder，否则编译时，会报错。

```java
@Subcomponent(modules=CommonModule.class)
public interface CommonComponnet{
	Test getTest();
  
   @Subcomponent.Builder
    interface Builder{
        Builder commonModule(CommonModule module);
        CommonComponnet build();
    }
}
```

```java
@PerActivity
@Subcomponent
public interface MainFragmentComponent {
    void inject(MainFragment mainFragment);
}
```
关于 @Subcomponent 的用法，它的作用和 dependencys 相似，这里表示 MainFragmentComponent是一个子组件，那么它的父组件是谁呢？ 提供了获取 MainFragmentComponent 的组件，如 MainComponent中的 `MainFragmentComponent mainFragmentComponent();`，是这样做的关联。

```java
@PerActivity
@Component(dependencies = AppComponent.class,modules = {MainModule.class, ActivityModule.class})
public interface MainComponent extends ActivityComponent{
    //对MainActivity进行依赖注入
    void inject(MainActivity mainActivity);
    MainFragmentComponent mainFragmentComponent();
}
```

```java
MainActivityPresenter presenter = new MainActivityPresenter(this);

MainActivity_MembersInjector

public void injectMembers(MainActivity instance) {
  if (instance == null) {
  ……  }
  instance.presenter = presenterProvider.get();
}
```

其实，Dagger2在使用这几个注解解决三个问题。

- 创建对象的代码放到那里去了?
- 创建好的对象给谁（指定接收者）?
- 如何将创建好的对象赋值给接收者?

指定创建对象的类（组件）和方法

- @Module：指定创建对象的类
- @Provides：指定创建对象的方法

初始时：

```java
public class MainActivityModule {
  public String createString(){
    return new String(“itheima”);
  }
}
```
添加注解后：
```java
@Moudle
public class MainActivityModule {
  @Provides
  public String createString(){
    return new String(“itheima”);
  }
}
```
指定接收者：创建好的对象需要赋值给指定的目标，我们需要通过@Inject注解告知Dagger2容器，把已经创建好的对象赋值给谁。
```java
public class MainActivity…..{
  @Inject
  String target;
}
```
将接收者和创建好的对象联系在一起：通过@Component来指定工作由哪个接口完成，通过这个接口我们可以看到组件和接收者。

添加子组件到父组件，有两种方式

- 通过@Component的dependencies属性依赖父组件


- 使用`@Module(subcomponents = SubComponent.class)`

### 2.9 Lazy

懒加载，延迟实例化对象，在需要的时候，才会进行实例化

```java
class GridingCoffeeMaker {
  @Inject Lazy<Grinder> lazyGrinder;

  public void brew() {
    while (needsGrinding()) {
      // Grinder created once on first call to .get() and cached.
      lazyGrinder.get().grind();
    }
  }
}
```

## 3. Sample

![](img/dagger2_sample.png)

### AppComponent

```java
@Singleton
@Component(modules = {AppModule.class})
public interface AppComponent {

    Context getContext();

    Navigator getNavigator();

    ToastUtil getToastUtil();
    //  Test test();
}
```

对应的module

```java
@Module
public class AppModule {

    Context context;

    public AppModule(Context context){
        this.context = context;
    }

    @Provides @Singleton
    public Context provideContext(){
        return context;
    }

    @Provides @Singleton
    public Navigator provideNavigator(){
        return new Navigator();
    }

    @Provides @Singleton
    public ToastUtil provideToastUtil(){
        return new ToastUtil(context);
    }

//    @Provides @Singleton
//    public Test provideTest(){
//        return new Test();
//    }
}
```

注入到Application

```java
public class App extends Application {

    AppComponent mAppComponent;

    @Override
    public void onCreate() {
        super.onCreate();

        mAppComponent = DaggerAppComponent.builder()
                .appModule(new AppModule(this))
                .build();
    }

    public AppComponent getAppComponent(){
        return mAppComponent;
    }
}
```

### ActivityComponent

```java
@PerActivity
@Component(modules = {ActivityModule.class})
public interface ActivityComponent {
    Activity getActivity();
}
```

对应的module

```java
@Module
public class ActivityModule {

    private final Activity activity;
    public ActivityModule(Activity activity){
        this.activity = activity;
    }

    @Provides @PerActivity
    public Activity provideActivity(){
        return activity;
    }
}
```

#### MainComponent

MainComponent继承了ActivityComponent，假如ActivityComponent中定义了创建类实例方法，则MainComponent中必须提供@Inject或@Provides对应的初始化类实例的方法

```java
@PerActivity
@Component(dependencies = AppComponent.class,modules = {MainModule.class, ActivityModule.class})
public interface MainComponent extends ActivityComponent{
    //对MainActivity进行依赖注入
    void inject(MainActivity mainActivity);

    MainFragmentComponent mainFragmentComponent();
}
```

对应的module

```java
@Module
public class MainModule {

    @Provides
    public GetUserData provideUserData(){
        return new GetUserData();
    }
}
```
#### MainFragmentComponent

```java
@PerActivity
@Subcomponent
public interface MainFragmentComponent {
    void inject(MainFragment mainFragment);
}
```
### BaseActivity

```java
public class BaseActivity extends AppCompatActivity {

    public AppComponent getAppComponent(){
        return ((App)getApplication()).getAppComponent();
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
}
```
### MainActivity

```java
public class MainActivity extends BaseActivity implements MainFragment
        .OnFragmentInteractionListener {

    private MainComponent mMainComponent;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mMainComponent = DaggerMainComponent.builder().appComponent(getAppComponent())
                .mainModule(new MainModule())
                .activityModule(new ActivityModule(this)).build();
        mMainComponent.inject(this);

        ActivityComponent activityComponent = DaggerActivityComponent.builder()
                .activityModule(new ActivityModule(this))
                .build();
        activityComponent.getActivity();
    }

    public MainComponent getMainComponent() {
        return mMainComponent;
    }

    @Override
    public void onFragmentInteraction(Uri uri) {

    }

}
```
### MainFragment

```java
public class MainFragment extends BaseFragment implements MainPresenter.IUserView{

    @Inject
    MainPresenter mainPresenter;
    @Inject
    ToastUtil toastUtil;

    @Inject
    MultiConstruct multiConstruct;

    private MainFragmentComponent mainFragmentComponent;

    private OnFragmentInteractionListener mListener;
    ...
    
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        if(getActivity() instanceof MainActivity){
            mainFragmentComponent = ((MainActivity)getActivity() ).getMainComponent().mainFragmentComponent();
            mainFragmentComponent.inject(this);
            mainPresenter.setUserView(this);
        }
    }
    ...
}
```