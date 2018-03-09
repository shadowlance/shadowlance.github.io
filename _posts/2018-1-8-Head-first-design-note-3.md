---
layout: post
title: 设计模式笔记(3) -- 观察者模式，单件模式
category: OO Design
---
经过前两节的热身，现在我们接触设计模式中重要的两个模式，观察者模式和工厂模式。<!-- more -->

## 观察者模式
在设计一个系统时，我们经常会接触一个这样的一个场景:我们需要监控一些状态的实时变化，以便及时做出反应和处理。经过前几章的经验，将所监视的观察者和信息发布者都想像为对象，对于信息发布者和不同观察者的沟通，我们可以创建上级超类/接口通过多态完成。以一个气象站和气象站提供的几个天气布告板为例，系统设计代码可以如下:
```java
public class WeatherStation {
  //我们希望观察者可以随时变动并且不被钉死在WeatherStation类中，所以用个ArrayList来控制
  private ArrayList observers;
  private float temperature;
  private float pressure;
  private float humidity;

  public WeatherStation(){
    this.observers = new ArrayList();
  }

  public void AddObserver(Observer o){
    observers.add(o);
  }

  public void notifyObservers(){
    for (Observer o : observers ) {
      o.update(temperature,pressure,humidity);
    }
  }

  public void removeObserver(Observer o){
    int i = observers.indexOf(o);
    if (i >=0) {
      observers.remove(i);
    }
  }

  public void StationUpdated(){
    notifyObserver();
  }

  public float getTemperature(){
    return temperature;
  }
  //...
}

public interface Observer {
  public void update(float emperature, float pressure, float humidity);
}

public class GeneralDisplay implements Observer{
 private float temperature;
 private float pressure;
 private float humidity;

 public void update(float emperature, float pressure, float humidity){
   //...
 }

 public void display(){
   //...
 }
}

public class ForecastDisplay implements Observer{
  //...
}
```
这就是一个观察者模式的雏形，在完善它之前，先给出此模式的定义:
>**观察者模式定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新**

我们现在的代码已经可以完成我们的预定任务，但系统将来可能需要更多的需求，这时我们就要更新我们的代码让其更强大。一般我们会遇到的更新需求有这两个:

- 不再是多对一监视。而是多个观察者，多个信息发布者，多对多监视
- 发布者可以主动选择是否给观察者信息而不是被动的信息一更新就自动传，观察者也可以主动查询获得最新信息

针对第一点的信息发布者方面，我们可以让信息发布者继承某个类/或接口。选择是类还是接口的优缺点相信大家已经清楚，根据实际情况而定。而对观察者来说，上面代码的`Observer`接口的`update()`肯定不能那么写了，我们可以考虑把整个信息发布者的对象传过来。这样我们也可以针对不同的信息发布者进行细微的操作。

对于第二点，我们可以考虑在信息发布者总的超类中加入一个`willPush`的boolean值并改写`StationUpdated()`的函数来操纵是否发布信息。而对于接口，只能在子类中分别加入这些实现了。而在第一点的解决方案中我们在`update()`函数中传入了的是整个信息发布者的对象，所以如果观察者想主动查询获得最新信息，可以直接调用信息发布者的各种函数直接获得信息。下面我们以信息发布者应用接口的方法为例:
```java
public interface Subject{
  public void AddObserver(Observer o);
  public void notifyObservers();
  public void removeObserver(Observer o);
}

public class WeatherStation implements Subject{
  //For others, all same as before
  boolean willPush;

  public void StationUpdated(){
    willPush = (statement == something) ? true : false;
    if (willPush) {
      notifyObserver();
    }
  }

  public void notifyObservers(){
    for (Observer o : observers ) {
      o.update(this);
    }
  }
  //...
}

public interface Observer {
  public void update(Subject subject);
}

public class GeneralDisplay implements Observer{
  //...

  GeneralDisplay(Subject weatherstation, Subject stationA){
    weatherstation.AddObserver(this);
    stationA.AddObserver(this);
    //...
  }

  public void update(Subject subject){
    if (subject instanceof WeatherStation) {
      this.temperature = subject.getTemperature();
      //...
    }else if (subject instanceof StationA) {
      //...
    }
    //...
  }
  //...
}
```
在观察者模式中，我们改变信息发布者或观察者任一方，并不会影响另一方，因为二者是松耦合的，所以只要它们的接口仍被遵守，我们就可以自由改变它们。这也是我们设计模式中的一个原则:
>**为了交互对象的松耦合设计而努力**

## 单件模式
在某些时候，我们的程序的某些部分只需要一个部件。比如程序中控制联网的httpCilent。对于这种情况，最简单的方法就是设计一个类里面全是static方法和对象来解决。但全部都是static显得类过于呆板，而且static对象必须在程序一开始就创建好，如果这些对象创建十分耗时，那么启动等待将会折磨疯你的用户。并且static创建后将一直存在，而客户有可能根本不会用到这部分资源。比如客户使用程序的离线功能时根本不会使用到httpCilent，这造成了程序资源的极大浪费。所以我们必须使用一种新的方法来保证单一部件。

在想如何保证单一部件之前，我们先回顾下对象是如何创造的。要创建一个对象时，我们使用new关键字。后面调用对象的公有构造方法。也就是说如果我们要保证只有一个对象的话，我们有两种选择:
- 阻止new。
- 阻止构造方法构造。

对于第一种选择，我们需要在类内存储一个static变量来记录此类是否创建，然后再每次new之前检查此变量。这种方法可以看出有明显弊端。首先每次创建都需要写检查代码，这么做的话简直要把人逼疯。其次对于多线程并发场景。这种方法将瞬间失效。
所以我们现在就得考虑如何实现第二种方法。阻止构造方法构造意味着类将没有公有的构造方法。这样所面临最大的问题就是没有公有构造方法类该如何构造。解决它的方法就是调用即使没有产生对象也可以调用的方法:static方法！解决代码示例如下：
```java
public class Singleton {
    private static Singleton uniqueInstance;//利用一个静态变量保留对所创建的唯一对象的引用

    //私有构造器，通过类内部的静态函数调用
    private Singleton() {
      //do something
    }

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }

    // Do other thing
}
```
这样，我们就可以使用`Singleton.getInstance()`来获得创建的唯一实例了。这种代码方法也被称为单件模式。
>**单件模式确保一个类只有一个实例，并提供一个全局访问点**

现在，我们必须面对一个新的问题:多线程并发下如何保证。最简单的方法是把函数`public static Singleton getInstance()` 加上`synchronized`修饰加上线程锁变成`public static Singleton getInstance()`来保证多线程执行此函数时顺序执行它。但实际上多线程可能出问题的阶段只有创建对象阶段。创建uniqueInstance变量之后`synchronized`修饰只会起到降低性能的作用。

再或者，我们可以一开始就创建好单件类
```java
public class Singleton {
    private static Singleton uniqueInstance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return uniqueInstance;
    }
}
```
但这样和最开始全写static方法对象比较除了不全static不那么呆板外没有其他改进。

幸好，在Java5之后，引入了volatile修饰符，这个修饰符可以让任何线程对变量的修改直接更新到主线程。而任何线程对变量的读取都是读取主线程的值。如果想了解此修饰符的详情，可以参考下[这篇博客](http://cmsblogs.com/?p=2092)。使用volatile修饰符后，代码示例如下:
```java
public class Singleton {
    private static volatile Singleton uniqueInstance;//利用一个静态变量保留对所创建的唯一对象的引用

    //私有构造器，通过类内部的静态函数调用
    private Singleton() {
      //do something
    }

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
          synchronized(Singleton.class){
            if (uniqueInstance == null){
              uniqueInstance = new Singleton();
            }
          }
        }
        return uniqueInstance;
    }

    // Do other thing
}
```
这就是著名的双重检查锁(DCL)。通过此方法，synchronized的性能降低问题得以解决。

## 总结
- 设计模式原则

  为了交互对象的松耦合设计而努力
- 观察者模式: 观察者模式定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。
- 单件模式: 单件模式确保一个类只有一个实例，并提供一个全局访问点。
