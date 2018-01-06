---
layout: post
title: 设计模式笔记(2) -- 模板方法模式，适配器模式
category: OO Design
---
在上文，我们知道了设计模式的三个基本原则并学到了第一个模式-策略模式。而在程序设计中，还有很多很多模式需要我们探索。如果我们回忆一下的话，我们引出策略模式的例子是：母类有很多子类且需要针对不同子类设计不同的算法。不同的设计模式就是前人为了解决不同的问题而总结出的解决方案。曾经逛论坛时看到一个人猛烈抨击设<!-- more -->计模式认为其毫无意义，因为自己以前毫无接触设计模式，凭看和写了多年代码现在写出的代码结构也很好，浏览设计模式书后发现自己有些代码甚至比书本上写的漂亮。但这个例子实在不能支持他的观点。因为好的代码里面就已经蕴含了好的设计思想，他其实是在另一个层面学习设计模式，而设计模式这门学问则是系统的集中的学习，直接学习可以起到事半功倍的效果。不过此人的观点也给我们提了一个醒：模式是死的，我们在实际写代码时需要遵循的只有问题的需求，比如有时情况下不用模式甚至会比较好，而在设计一个系统时，**设计模式只是一个指导方针**，起到一个启发我们思维的帮助作用，我们需要关注的应该更是其背后蕴含的设计思想以及问题本身，这样我们甚至可以写出比教科书还要好的解决方案。

好了，下面进入正题，这次会介绍两个比较简单的模式，模板方法模式和适配器模式(才不是想偷懒才挑简单的讲>д<).

## 模板方法模式
假设我们现在开一家店，在给客户提供咖啡，茶以及其他饮料。现在的各个饮料的类是这样的:
```java
public class Coffee{
  void prepareCoffee(){
    boilWater();
    addCoffeeBins();
    pourInCup();
    addSugerAndMilk();
  }
  public void boilWater(){
    //do something
  }
  public void addCoffeeBins(){
    //do something
  }
  public void pourInCup(){
    //do something
  }
  public void addSugerAndMilk(){
    //do something
  }
  //Other func...
}

public class Tea{
  void prepareCoffee(){
    boilWater();
    addTeaLeaf();
    pourInCup();
    addLemon();
  }
  public void boilWater(){
    //do something
  }
  public void addTeaLeaf(){
    //do something
  }
  public void pourInCup(){
    //do something
  }
  public void addLemon(){
    //do something
  }
  //Other func...
}

//Other Beverages
```
明显可以看出，这些类存在重复代码，饮料制作的过程实在是太过于相似了，除此之外，还有一些函数也很相似(甚至就一样，我问你烧水还有什么烧得花样么。。。)。由设计模式第一条原则"合并重复及相似"，我们可以提取二者公有的部分做成一个超类，对于一成不变的部分，如烧水`boilWater()`，倒饮料`pourInCup()`这些，我们甚至可以把它们直接写到父类中，代码变成这样:
```java
public class CaffeineBeverage{
  //注意这里是个final方法
  final void prepareBeverage(){
    boilWater();
    brew();
    pourInCup();
    addCondiment();
  }
  public void boilWater(){
    //do something
  }
  public void pourInCup(){
    //do something
  }

  abstract public void brew();
  abstract public void addCondiment();
  //Other func...
}

public class Tea extends CaffeineBeverage{
  public void brew(){
    //do something
  }
  public void addCondiment(){
    //do something
  }
  //Other func...
}

public class Coffee extends CaffeineBeverage{
  public void brew(){
    //do something
  }
  public void addCondiment(){
    //do something
  }
  //Other func...
}
```
好的，恭喜你学会一个新的模式-模板方法模式！(这么快？！)因为这确实是一个比较简单的模式，不过它还有一些东西可以发掘，而且我也相信部分人肯定对这个处理过程有些疑问。这里先给出此模式的定义:
>**模板方法模式在一个方法中定义了一个算法的骨架，将一些实施步骤延迟到子类中。模板方法使得子类在不改变算法结构的情况下，重新定义一些算法的某些步骤。**

现在回答一些疑问：

1. 为什么在模板方法模式中模板方法要设置为final?
   刚才大家也在感叹为何如此之快，简单的运用一条原则就做出一个模式。其实*整个过程中体现模板方法模式的只有`final void prepareBeverage()`这个`final`方法*。这个函数用来固定的给一类算法给出算法骨架，而我们肯定不希望算法骨架可以被随意改变，所以把它设为了final。另外，如果我们给其他算法也做类似行为制作算法骨架(甚至和此类共同继承某个更抽象的类)，这样不同算法骨架是在同一级，有利于改善代码结构。当然是不是final还是要具体问题具体分析，实际应用还是要由需求而定灵活变通，不要太死板
2. 这个问题不和上次讲的策略模式很像么？上次在策略模式中也说了应该把算法抽离出来成接口然后使用组合，现在这种做法正是使用了上次我们摒弃的做法，违反了"多用组合，少用继承"的设计原则！
   没错！模板方法模式使用了继承，这是因为在这种设计问题中，我们所需要耦合的算法骨架一般很少，绝大多数情况下基本只需要一个算法骨架。而在策略模式中，我们需要耦合的算法较多而每个算法内包含的步骤较少。此外，策略模式也有它的缺点，创建出了额外的类花了更大的开销，也不利于代码解读。所以在这种情况下，继承是一个更方便的举动。而对于算法骨架包含的小算法，当他们过多时我们就可以使用策略模式来改善它们。后面的将学到的工厂模式就是二者的绝佳组合。模板方法模式和策略模式的这个矛盾也给了我们一个"在耦合组件时，什么时候该使用继承，什么时候该使用组合"的一个指导。

### 钩子
现在，我们关注此模式的核心`final void prepareBeverage()`，介绍一下"钩子"。
```java
public class CaffeineBeverage{
  //注意这里是个final方法
  final void prepareBeverage(){
    boilWater();
    brew();
    pourInCup();
    if(CustomerWant()) addCondiment();
    SomePotentialStep();
  }
  boolean CustomerWant(){return true;}
  void SomePotentialStep(){};
  //...
}
```
钩子就是声明在抽象类中的非抽象方法，但只有空的或默认的实现。在模板方法模式中，它的作用是让声明为`final`的模板方法可以有一些预定的，可选的或额外的步骤，可以提高其弹性，预定的方法则可降低子类的负担。在此例中，类似`boilWater()`和`CustomerWant()`的方法就是钩子。

### 好莱坞原则
既然模板方法模式是一个关于继承的的模式，那我们也谈一下设计模式中关于继承的一个设计原则
>**好莱坞原则:别调用(打电话给)我们，我们会调用(打电话给)你**

好莱坞原则主要是为了解决"依赖腐败"的问题。当高层组件依赖底层组件，底层组件又依赖高层组件，中心的，边缘的，各种层次的组件都互相依赖时，依赖腐败就发生了。在这种情况下，没人能轻易的搞懂系统如何设计。

在好莱坞原则下，底层组件通过钩子挂钩到高层组件中，而高层组件使用多态决定调用何种底层组件。子类依赖于父类，父类则不依赖子类。高层组件对底层组件的态度就是"别调用我们，我们会调用你"。这个原则实际上就是提醒我们在设计系统时要注意避免产生环状依赖，各个模块要层次分明。比如在模板方法模式中模板类就是通过final方法来让模板类处于同一级，改善代码结构。

### 模板方法模式的应用
刚接触Android编程的一些人可能和我有一样的感触:为啥我们在继承某个组件类之后，只需要写一小部分函数，组件就可以运行？这就是模板方法模式的好处，算法框架以及很多钩子的默认实现都已经在父类中实现，我们只需要完成自己的部分就可以了。比如下面代码
```java
public class MainActivity extends AppCompatActivity implements NavigationView.OnNavigationItemSelectedListener{
  @Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //something
}

@Override
public void onBackPressed() {
  //something
}

@Override
public boolean onNavigationItemSelected(MenuItem item) {
    // something
}

@Override
public boolean onCreateOptionsMenu(Menu menu) {
  //something
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
  //something
  }
}
```

## 适配器模式
过几个月，我就要去欧洲工作一趟，师兄对我说:"走之前买插座几个转接头"，因为欧洲那边的插座标准与中国不一样。在面向对象程序设计中，适配器(Adapter)就是用来兼容新旧接口的"转接头"。

适配器模式也是一个很简单的模式，通过下面的例子即可说明。假设我们有鸭子，鸡这两个类，由于未知原因，你需要用鸡来冒充鸭子。那么鸭子适配器写法如下:
```java
public interface Duck{
  public void quark();
  public void fly();
}

public interface Chicken{
  //Chicken can gobble but can't quark
  public void gobble();
  //Chicken can glide but can't fly
  public void glide();
}

//Adapter
public ChickenAdapter implements Duck{
  Chicken chicken;
  public ChickenAdapter(Chicken chicken){
    this.chicken = chicken;
  }
  public void quark(){
    //use gobble faking quark
    chicken.gobble();
  }
  public void fly(){
    //glide 5 times to fake fly
    for (int i = 0; i < 5; i++ ) {
      chicken.glide();
    }
  }
}
```
使用时只需要使用类似如下代码即可。
```java
Chicken chicken = new Chicken();
Duck fakeDuck = new ChickenAdapter(chicken);
```
这个模式是不是很简单呢？通过一个继承目标接口的类，在里面写入转换请求的方法，这样就可以把新的请求转化为旧的请求了。下面给出适配器模式的正式定义:
>**适配器模式将一个类的接口，转换成客户期望的另一个接口，适配器让原本接口不兼容的类可以合作无间**

另外，在C++中，除了使用组合的方式构造适配器外，我们还可以使用C++的特性"多重继承"来构造适配器。他的优点是多重继承对于被适配者独有的适配者却无法提供的方法不需要重新写代码，而且必要时，还可以覆盖被适配者的行为。此外，多重继承还可以进行双向适配。而缺点是显而易见的，多重继承隐式的给代码增加了很大复杂度，某些情况下会埋下意想不到的很难排查的错误(有兴趣可以搜下"c++多重继承"，排名第一的居然不是讲它是什么，而是在它说怎么坑，如何处理坑。。。这也是Java创建接口抛弃多重继承的原因)，所以此方法要慎用。

### 外观模式
适配器模式还有一个拓展，就是外观模式。外观模式是什么呢？举个例子，我们在使用空调时只需要通过遥控器的几个按钮设定温度，模式等，空调内部的压缩机，风扇会自动调整功率，转速等，我们只需要使用遥控器提供给我们的几个接口就可以。外观模式就是把多个接口重新包装成几个简单的接口来供我们使用，下面是它的正式定义
>**外观模式提供一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用**

下面举一个例子来说明
```java
public class Car {
  Engine engine;
  DashBoard dashBoard;
  //Others...

  public Car(){
    //Some step
  }

  public void start(Key key){
    Doors doors = new Doors();
    boolean authorized = key.turns();
    if (authorized) {
      engine.start();
      doors.lock();
      updateDashBoard();
    }
  }

  public void updateDashBoard(){
    //do something
  }
}
```
在此例中，车辆的使用者只需要接触`Car`这个类提供的函数就可以了，并不需要管理引擎，仪表盘等是如何工作的。

在外观模式的设计中，有这样一条原则需要遵守，那就是最小知识原则，定义如下:
>**最小知识原则:只和你的密友谈话，不要让过多的类耦合在一起**

此原则提供了一个方针，就任何对像而言，在该对象的方法内，我们应该调用属于以下范围的方法:
- 该对象本身
- 被当作方法的参数而传入的对象
- 此方法直接创建或实例化的任何对像 (此条和上一条结合告诉我们，如果在方法中某对象是通过调用其他对象的方法得到的，不要调用此对象的方法！)
- 对象的任何组件

简单地说，就是最好不要出现诸如`ObjectA a = b.Getxx().getyy()`之流的代码或其变体。如在上例`Car`类的`start()`方法中，我们可以调用Car包含的对象对象`engine`的方法，对象自己的方法`updateDashBoard`，方法调用的对象`key`的方法，以及直接创建的对象`Door`的方法。但我们最好不要做出如下行为
```java
Glass doorGlass = Door.getGlass();  //此对象不是直接创建而是通过调用其他对象的方法得到！
doorGlass.close();  //违反了最小设计原则！
```
当然，最小设计原则只是在我们设计系统里的一个强烈建议，为了解决具体问题方便偶尔违反，只要合情合理也没什么问题，最明显的违反此原则的例子就是`System.out.println()`这个java最常用输出语句。

## 总结
- 设计模式原则

  1.最小知识原则:只和你的密友谈话，不要让过多的类耦合在一起

  2.好莱坞原则:别调用(打电话给)我们，我们会调用(打电话给)你
- 模板方法模式: 在一个方法中定义了一个算法的骨架，将一些实施步骤延迟到子类中。模板方法使得子类在不改变算法结构的情况下，重新定义一些算法的某些步骤。
- 适配器模式: 将一个类的接口，转换成客户期望的另一个接口，适配器让原本接口不兼容的类可以合作无间.
- 外观模式: 提供一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用
- 运用设计模式的建议: 设计一个系统时，设计模式只是一个指导方针，我们要灵活变通，具体问题具体对待。
