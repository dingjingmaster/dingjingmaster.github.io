# 设计模式


## 设计模式简介

- 设计模式是一套被反复使用的、多数人知晓的、经过分类编目的、代码设计经验的总结。
- 使用设计模式是为了重用代码、让代码更容易被他人理解、保证代码可靠性。

### 设计模式的设计原则

- 对接口编程而不是对实现编程
- 优先使用对象组合而不是继承

### 设计模式的类型

目前总共有23 种设计模式。这些模式可以分为三大类：
- 创建型模式（Creational Patterns）
- 结构型模式（Structural Patterns）
- 行为型模式（Behavioral Patterns）
- 当然，我们还会讨论另一类设计模式：J2EE 设计模式。

<style>table th:first-of-type{width:390px;}</style>
|模式&描述|包括|
|:----|:----|
|`创建型模式`<br/><br/>这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用`new`运算符直接实例化对象。这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活|工厂模式（Factory Pattern）<br/>抽象工厂模式（Abstract Factory Pattern）<br/>单例模式（Singleton Pattern）<br/>建造者模式（Builder Pattern）<br/>原型模式（Prototype Pattern）|
|`结构型模式`<br/><br/>这些设计模式关注类和对象的组合。继承的概念被用来组合接口和定义组合对象获得新功能的方式|适配器模式(Adapter Pattern)<br/>桥接模式(Bridge Pattern)<br/>过滤器模式(Filter、Criteria Pattern)<br/>组合模式(Composite Pattern)<br/>装饰器模式(Decorator Pattern)<br/>外观模式(Facade Pattern)<br/>享元模式(Flyweight Pattern)<br/>代理模式(Proxy Pattern)|
|`行为型模式`<br/><br/>这些设计模式特别关注对象之间的通信|责任链模式(Chain of Responsibility Pattern)<br/>命令模式(Command Pattern)<br/>解释器模式(Interpreter Pattern)<br/>迭代器模式(Iterator Pattern)<br/>中介者模式(Mediator Pattern)<br/>备忘录模式(Memento Pattern)<br/>观察者模式(Observer Pattern)<br/>状态模式(State Pattern)<br/>空对象模式(Null Object Pattern)<br/>策略模式(Strategy Pattern)<br/>模板模式(Template Pattern)<br/>访问者模式(Visitor Pattern)<br/>|
|`J2EE 模式`<br/><br/>这些设计模式特别关注表示层。这些模式是由 Sun Java Center 鉴定的|MVC 模式(MVC Pattern)<br/>业务代表模式(Business Delegate Pattern)<br/>组合实体模式(Composite Entity Pattern)<br/>数据访问对象模式(Data Access Object Pattern)<br/>前端控制器模式(Front Controller Pattern)<br/>拦截过滤器模式(Intercepting Filter Pattern)<br/>服务定位器模式(Service Locator Pattern)<br/>传输对象模式(Transfer Object Pattern)|


<div align=center><img src='/pic/c&c++/dp-0.jpg'/></div>

### 设计模式六大原则
1. 开闭原则: **对扩展开放，对修改关闭**
2. 里氏代换原则: **任何基类可以出现的地方，子类一定可以出现**(LSP 是继承复用的基石，只有当派生类可以替换掉基类，且软件单位的功能不受到影响时，基类才能真正被复用，而派生类也能够在基类的基础上增加新的行为，里氏代换原则是对开闭原则的补充)。
3. 依赖倒转原则: **针对接口编程，依赖于抽象而不依赖于具体**(这个原则是开闭原则的基础)
4. 接口隔离原则: **使用多个隔离的接口，比使用单个接口要好**(降低类之间的依赖/耦合度)
5. 迪米特法则，又称最少知道原则: **一个实体应当尽量少地与其他实体之间发生相互作用，使得系统功能模块相对独立**
6. 合成复用原则: **尽量使用合成/聚合的方式，而不是使用继承**

## 设计模式

### 工厂模式

> 在工厂模式中，我们在创建对象时**不会对客户端暴露创建逻辑**，并且是**通过使用一个共同的接口来指向新创建的对象**

#### 意图

定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行

#### 主要解决
主要解决接口选择的问题

#### 优点

1. 一个调用者想创建一个对象，只要知道其名称就可以了
2. 扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以
3. 屏蔽产品的具体实现，调用者只关心产品的接口。

#### 缺点

每次增加一个产品时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。这并不是什么好事。

#### 使用场景

1. 日志记录器：记录可能记录到本地硬盘、系统事件、远程服务器等，用户可以选择记录日志到什么地方
2. 数据库访问：当用户不知道最后系统采用哪一类数据库，以及数据库可能有变化时
3. 设计一个连接服务器的框架，需要三个协议，"POP3"、"IMAP"、"HTTP"，可以把这三个作为产品类，共同实现一个接口。

> 作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度

#### 实现

<div align=center><img src='/pic/c&c++/dp-f-0.jpg'/></div>
<center>工厂模式</center>

#### 例子 
1. 创建一个接口 `Shape.java`
```java

public interface Shape {
   void draw();
}
```

2. 创建实现接口的实体类
```java
// Rectangle.java
public class Rectangle implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Rectangle::draw() method.");
   }
}
```

```java
// Square.java

public class Square implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Square::draw() method.");
   }
}
```

```java
// Circle.java

public class Circle implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Circle::draw() method.");
   }
}
```

3. 创建一个工厂`ShapeFactory.java`

```java

public class ShapeFactory {

   //使用 getShape 方法获取形状类型的对象
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();
      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();
      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }
      return null;
   }
}
```

5. 使用工厂
```java

public class FactoryPatternDemo {

   public static void main(String[] args) {
      ShapeFactory shapeFactory = new ShapeFactory();

      //获取 Circle 的对象，并调用它的 draw 方法
      Shape shape1 = shapeFactory.getShape("CIRCLE");

      //调用 Circle 的 draw 方法
      shape1.draw();

      //获取 Rectangle 的对象，并调用它的 draw 方法
      Shape shape2 = shapeFactory.getShape("RECTANGLE");

      //调用 Rectangle 的 draw 方法
      shape2.draw();

      //获取 Square 的对象，并调用它的 draw 方法
      Shape shape3 = shapeFactory.getShape("SQUARE");

      //调用 Square 的 draw 方法
      shape3.draw();
   }
}
```

### 抽象工厂模式

抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

#### 意图
提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类

#### 主要解决
主要解决接口选择的问题

#### 何时使用
系统的产品有多于一个的产品族，而系统只消费其中某一族的产品

#### 优点
当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象

#### 缺点
产品族扩展非常困难，要增加一个系列的某一产品，既要在抽象的 Creator 里加代码，又要在具体的里面加代码

#### 使用场景
1. QQ 换皮肤，一整套一起换
2. 生成不同操作系统的程序

> 产品族难扩展，产品等级易扩展

#### 实现
<div align=center><img src='/pic/c&c++/dp-af-0.jpg'/></div>
<center>抽象工厂模式</center>

#### 例子
1. 创建一个接口
```java
// Shape.java

public interface Shape {
   void draw();
}
```

2. 创建实现接口的实体类
```java
// Rectangle.java


public class Rectangle implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Rectangle::draw() method.");
   }
}

```

```java
// Square.java


public class Square implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Square::draw() method.");
   }
}

```

```java
// Circle.java

public class Circle implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Circle::draw() method.");
   }
}
```

3. 创建颜色的接口
```java
// Color.java

public interface Color {
   void fill();
}
```

4. 创建实现接口的实体类

```java
// Red.java

public class Red implements Color {

   @Override
   public void fill() {
      System.out.println("Inside Red::fill() method.");
   }
}
```

```java
// Green.java

public class Green implements Color {

   @Override
   public void fill() {
      System.out.println("Inside Green::fill() method.");
   }
}
```

```java
// Blue.java

public class Blue implements Color {

   @Override
   public void fill() {
      System.out.println("Inside Blue::fill() method.");
   }
}

```

5. 为 Color 和 Shape 对象创建抽象类来获取工厂

```java
// AbstractFactory.java

public abstract class AbstractFactory {
   public abstract Color getColor(String color);
   public abstract Shape getShape(String shape);
}
```

6. 创建扩展了 AbstractFactory 的工厂类，基于给定的信息生成实体类的对象

```java
// ShapeFactory.java


public class ShapeFactory extends AbstractFactory {

   @Override
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();
      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();
      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }
      return null;
   }

   @Override
   public Color getColor(String color) {
      return null;
   }
}
```

```java
// ColorFactory.java


public class ColorFactory extends AbstractFactory {

   @Override
   public Shape getShape(String shapeType){
      return null;
   }

   @Override
   public Color getColor(String color) {
      if(color == null){
         return null;
      }
      if(color.equalsIgnoreCase("RED")){
         return new Red();
      } else if(color.equalsIgnoreCase("GREEN")){
         return new Green();
      } else if(color.equalsIgnoreCase("BLUE")){
         return new Blue();
      }
      return null;
   }
}
```

7. 创建一个工厂创建器/生成器类，通过传递形状颜色信息来获取工厂
```java
// FactoryProducer.java

public class FactoryProducer {
   public static AbstractFactory getFactory(String choice){
      if(choice.equalsIgnoreCase("SHAPE")){
         return new ShapeFactory();
      } else if(choice.equalsIgnoreCase("COLOR")){
         return new ColorFactory();
      }
      return null;
   }
}
```

8. 使用 FactoryProducer 来获取 AbstractFactory，通过传递类型信息来获取实体类的对象
```java
// AbstractFactoryPatternDemo.java

public class AbstractFactoryPatternDemo {
   public static void main(String[] args) {

      //获取形状工厂
      AbstractFactory shapeFactory = FactoryProducer.getFactory("SHAPE");

      //获取形状为 Circle 的对象
      Shape shape1 = shapeFactory.getShape("CIRCLE");

      //调用 Circle 的 draw 方法
      shape1.draw();

      //获取形状为 Rectangle 的对象
      Shape shape2 = shapeFactory.getShape("RECTANGLE");

      //调用 Rectangle 的 draw 方法
      shape2.draw();

      //获取形状为 Square 的对象
      Shape shape3 = shapeFactory.getShape("SQUARE");

      //调用 Square 的 draw 方法
      shape3.draw();

      //获取颜色工厂
      AbstractFactory colorFactory = FactoryProducer.getFactory("COLOR");

      //获取颜色为 Red 的对象
      Color color1 = colorFactory.getColor("RED");

      //调用 Red 的 fill 方法
      color1.fill();

      //获取颜色为 Green 的对象
      Color color2 = colorFactory.getColor("GREEN");

      //调用 Green 的 fill 方法
      color2.fill();

      //获取颜色为 Blue 的对象
      Color color3 = colorFactory.getColor("BLUE");

      //调用 Blue 的 fill 方法
      color3.fill();
   }
}
```

### 单例模式
这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象

#### 意图
保证一个类仅有一个实例，并提供一个访问它的全局访问点

#### 主要解决
一个全局使用的类频繁地创建与销毁

#### 何时使用
当您想控制实例数目，节省系统资源的时候

#### 优点
1. 在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如管理学院首页页面缓存）
2. 避免对资源的多重占用（比如写文件操作）

#### 缺点
没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化

#### 使用场景
1. 要求生产唯一序列号
2. WEB 中的计数器，不用每次刷新都在数据库里加一次，用单例先缓存起来
3. 创建的一个对象需要消耗的资源过多，比如 I/O 与数据库的连接等

#### 实现
<div align=center><img src='/pic/c&c++/dp-s.jpg'/></div>
<center>单例模式</center>

#### 例子
1. 创建一个 Singleton 类

```java
// SingleObject.java


public class SingleObject {

   //创建 SingleObject 的一个对象
   private static SingleObject instance = new SingleObject();

   //让构造函数为 private，这样该类就不会被实例化
   private SingleObject(){}

   //获取唯一可用的对象
   public static SingleObject getInstance(){
      return instance;
   }

   public void showMessage(){
      System.out.println("Hello World!");
   }
}
```

2. 从 singleton 类获取唯一的对象
```java
// SingletonPatternDemo.java

public class SingletonPatternDemo {
   public static void main(String[] args) {

      //不合法的构造函数
      //编译时错误：构造函数 SingleObject() 是不可见的
      //SingleObject object = new SingleObject();

      //获取唯一可用的对象
      SingleObject object = SingleObject.getInstance();

      //显示消息
      object.showMessage();
   }
}
```

#### 单例的几种实现方式
1. 懒汉式，线程不安全
```java

public class Singleton {
    private static Singleton instance;
    private Singleton (){}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

2. 懒汉式，线程安全
```java

public class Singleton {
    private static Singleton instance;
    private Singleton (){}
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

3. 饿汉式，线程安全
```java

public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton (){}
    public static Singleton getInstance() {
    return instance;
    }
}
```

### 构造者模式

建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象

#### 意图

将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示

#### 主要解决
主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定

#### 何时使用
一些基本部件不会变，而其组合经常变化的时候

#### 优点
1. 建造者独立，易扩展
2. 便于控制细节风险

#### 缺点
1. 产品必须有共同点，范围有限制
2. 如内部变化复杂，会有很多的建造类

#### 使用场景
1. 需要生成的对象具有复杂的内部结构
2. 需要生成的对象内部属性本身相互依赖

#### 实现

<div align=center><img src='/pic/c&c++/dp-b.svg'/></div>
<center>构造者模式</center>

#### 例子

1. 创建一个表示食物条目和食物包装的接口
```java
// Item.java

public interface Item {
   public String name();
   public Packing packing();
   public float price();
}
```

```java
// Packing.java

public interface Packing {
   public String pack();
}
```

2. 创建实现 Packing 接口的实体类

```java
// Wrapper.java

public class Wrapper implements Packing {

   @Override
   public String pack() {
      return "Wrapper";
   }
}
```

```java
// Bottle.java

public class Bottle implements Packing {

   @Override
   public String pack() {
      return "Bottle";
   }
}
```

3. 创建实现 Item 接口的抽象类，该类提供了默认的功能
```java
// Burger.java

public abstract class Burger implements Item {

   @Override
   public Packing packing() {
      return new Wrapper();
   }

   @Override
   public abstract float price();
}
```

```java
// ColdDrink.java

public abstract class ColdDrink implements Item {

    @Override
    public Packing packing() {
       return new Bottle();
    }

    @Override
    public abstract float price();
}
```

4. 创建扩展了 Burger 和 ColdDrink 的实体类
```java
// VegBurger.java

public class VegBurger extends Burger {

   @Override
   public float price() {
      return 25.0f;
   }

   @Override
   public String name() {
      return "Veg Burger";
   }
}
```

```java
// ChickenBurger.java

public class ChickenBurger extends Burger {

   @Override
   public float price() {
      return 50.5f;
   }

   @Override
   public String name() {
      return "Chicken Burger";
   }
}
```

```java
// Coke.java

public class Coke extends ColdDrink {

   @Override
   public float price() {
      return 30.0f;
   }

   @Override
   public String name() {
      return "Coke";
   }
}
```

```java
// Pepsi.java

public class Pepsi extends ColdDrink {

   @Override
   public float price() {
      return 35.0f;
   }

   @Override
   public String name() {
      return "Pepsi";
   }
}
```

5. 创建一个 Meal 类，带有上面定义的 Item 对象
```Meal.java

import java.util.ArrayList;
import java.util.List;

public class Meal {
   private List<Item> items = new ArrayList<Item>();

   public void addItem(Item item){
      items.add(item);
   }

   public float getCost(){
      float cost = 0.0f;
      for (Item item : items) {
         cost += item.price();
      }
      return cost;
   }

   public void showItems(){
      for (Item item : items) {
         System.out.print("Item : "+item.name());
         System.out.print(", Packing : "+item.packing().pack());
         System.out.println(", Price : "+item.price());
      }
   }
}
```

6. 创建一个 MealBuilder 类，实际的 builder 类负责创建 Meal 对象
```java
// MealBuilder.java


public class MealBuilder {

   public Meal prepareVegMeal (){
      Meal meal = new Meal();
      meal.addItem(new VegBurger());
      meal.addItem(new Coke());
      return meal;
   }

   public Meal prepareNonVegMeal (){
      Meal meal = new Meal();
      meal.addItem(new ChickenBurger());
      meal.addItem(new Pepsi());
      return meal;
   }
}
```

7. BuilderPatternDemo
```java
// BuilderPatternDemo.java

public class BuilderPatternDemo {
   public static void main(String[] args) {
      MealBuilder mealBuilder = new MealBuilder();

      Meal vegMeal = mealBuilder.prepareVegMeal();
      System.out.println("Veg Meal");
      vegMeal.showItems();
      System.out.println("Total Cost: " +vegMeal.getCost());

      Meal nonVegMeal = mealBuilder.prepareNonVegMeal();
      System.out.println("\n\nNon-Veg Meal");
      nonVegMeal.showItems();
      System.out.println("Total Cost: " +nonVegMeal.getCost());
   }
}
```

### 原型模式
原型模式（Prototype Pattern）是用于创建重复的对象，同时又能保证性能
<br/>

这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。

#### 意图
用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象

#### 主要解决
在运行期建立和删除原型

#### 何时使用
1. 当一个系统应该独立于它的产品创建，构成和表示时
2. 当要实例化的类是在运行时刻指定时，例如，通过动态装载
3. 为了避免创建一个与产品类层次平行的工厂类层次时
4. 当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些。

#### 优点
1. 性能提高
2. 逃避构造函数的约束

#### 缺点
1. 配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候
2. 必须实现 Cloneable 接口

#### 使用场景

1. 资源优化场景
2. 类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等
3. 性能和安全要求的场景 
4. 通过`new`产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式
5. 一个对象多个修改者的场景
6. 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用
7. 在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone 的方法创建一个对象，然后由工厂方法提供给调用者。原型模式已经与 Java 融为浑然一体，大家可以随手拿来使用。 
> 注意：与通过对一个类进行实例化来构造新对象不同的是，原型模式是通过拷贝一个现有对象生成新对象的。浅拷贝实现 Cloneable，重写，深拷贝是通过实现 Serializable 读取二进制流

#### 实现

<div align=center><img src='/pic/c&c++/dp-pt.png'/></div>
<center>原型模式</center>

#### 例子
1. 创建一个实现了 Cloneable 接口的抽象类
```java
// Shape.java


public abstract class Shape implements Cloneable {

   private String id;
   protected String type;

   abstract void draw();

   public String getType(){
      return type;
   }

   public String getId() {
      return id;
   }

   public void setId(String id) {
      this.id = id;
   }

   public Object clone() {
      Object clone = null;
      try {
         clone = super.clone();
      } catch (CloneNotSupportedException e) {
         e.printStackTrace();
      }
      return clone;
   }
}
```

2. 创建扩展了上面抽象类的实体类

```java
// Rectangle.java

public class Rectangle extends Shape {

   public Rectangle(){
     type = "Rectangle";
   }

   @Override
   public void draw() {
      System.out.println("Inside Rectangle::draw() method.");
   }
}
```

```java
// Square.java


public class Square extends Shape {

   public Square(){
     type = "Square";
   }

   @Override
   public void draw() {
      System.out.println("Inside Square::draw() method.");
   }
}
```

```java
// Circle.java

public class Circle extends Shape {

   public Circle(){
     type = "Circle";
   }

   @Override
   public void draw() {
      System.out.println("Inside Circle::draw() method.");
   }
}
```

3. 创建一个类，从数据库获取实体类，并把它们存储在一个 Hashtable 中
```java
// ShapeCache.java

import java.util.Hashtable;

public class ShapeCache {

   private static Hashtable<String, Shape> shapeMap
      = new Hashtable<String, Shape>();

   public static Shape getShape(String shapeId) {
      Shape cachedShape = shapeMap.get(shapeId);
      return (Shape) cachedShape.clone();
   }

   // 对每种形状都运行数据库查询，并创建该形状
   // shapeMap.put(shapeKey, shape);
   // 例如，我们要添加三种形状
   public static void loadCache() {
      Circle circle = new Circle();
      circle.setId("1");
      shapeMap.put(circle.getId(),circle);

      Square square = new Square();
      square.setId("2");
      shapeMap.put(square.getId(),square);

      Rectangle rectangle = new Rectangle();
      rectangle.setId("3");
      shapeMap.put(rectangle.getId(),rectangle);
   }
}
```

4. PrototypePatternDemo 使用 ShapeCache 类来获取存储在 Hashtable 中的形状的克隆
```java
// PrototypePatternDemo.java

public class PrototypePatternDemo {
   public static void main(String[] args) {
      ShapeCache.loadCache();

      Shape clonedShape = (Shape) ShapeCache.getShape("1");
      System.out.println("Shape : " + clonedShape.getType());

      Shape clonedShape2 = (Shape) ShapeCache.getShape("2");
      System.out.println("Shape : " + clonedShape2.getType());

      Shape clonedShape3 = (Shape) ShapeCache.getShape("3");
      System.out.println("Shape : " + clonedShape3.getType());
   }
}
```

### 适配器模式
适配器模式（Adapter Pattern）是作为两个不兼容的接口之间的桥梁。这种类型的设计模式属于结构型模式，它结合了两个独立接口的功能。
<br/>
这种模式涉及到一个单一的类，该类负责加入独立的或不兼容的接口功能。举个真实的例子，读卡器是作为内存卡和笔记本之间的适配器。您将内存卡插入读卡器，再将读卡器插入笔记本，这样就可以通过笔记本来读取内存卡。

#### 意图
将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

#### 主要解决
主要解决在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。

#### 何时使用
1. 系统需要使用现有的类，而此类的接口不符合系统的需要
2. 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口
3. 通过接口转换，将一个类插入另一个类系中。（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。） 

#### 优点
1. 可以让任何两个没有关联的类一起运行
2. 提高了类的复用
3. 增加了类的透明度
4. 灵活性好

#### 缺点
1. 过多地使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是 A 接口，其实内部被适配成了 B 接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构
2. 由于 JAVA 至多继承一个类，所以至多只能适配一个适配者类，而且目标类必须是抽象类。 

#### 使用场景
有动机地修改一个正常运行的系统的接口，这时应该考虑使用适配器模式

> 注意：**适配器不是在详细设计时添加的**，而是解决正在服役的项目的问题

#### 实现

<div align=center><img src='/pic/c&c++/dp-adp.png'/></div>
<center>适配器模式</center>

#### 例子

1. 为媒体播放器和更高级的媒体播放器创建接口

```java
// MediaPlayer.java

public interface MediaPlayer {
   public void play(String audioType, String fileName);
}
```

```java
// AdvancedMediaPlayer.java

public interface AdvancedMediaPlayer {
   public void playVlc(String fileName);
   public void playMp4(String fileName);
}
```

2. 创建实现了 AdvancedMediaPlayer 接口的实体类
```java
// VlcPlayer.java

public class VlcPlayer implements AdvancedMediaPlayer{
   @Override
   public void playVlc(String fileName) {
      System.out.println("Playing vlc file. Name: "+ fileName);
   }

   @Override
   public void playMp4(String fileName) {
      //什么也不做
   }
}
```

```java
// Mp4Player.java

public class Mp4Player implements AdvancedMediaPlayer{

   @Override
   public void playVlc(String fileName) {
      //什么也不做
   }

   @Override
   public void playMp4(String fileName) {
      System.out.println("Playing mp4 file. Name: "+ fileName);
   }
}
```

3. 创建实现了 MediaPlayer 接口的适配器类
```java
// MediaAdapter.java

public class MediaAdapter implements MediaPlayer {

   AdvancedMediaPlayer advancedMusicPlayer;

   public MediaAdapter(String audioType){
      if(audioType.equalsIgnoreCase("vlc") ){
         advancedMusicPlayer = new VlcPlayer();
      } else if (audioType.equalsIgnoreCase("mp4")){
         advancedMusicPlayer = new Mp4Player();
      }
   }

   @Override
   public void play(String audioType, String fileName) {
      if(audioType.equalsIgnoreCase("vlc")){
         advancedMusicPlayer.playVlc(fileName);
      }else if(audioType.equalsIgnoreCase("mp4")){
         advancedMusicPlayer.playMp4(fileName);
      }
   }
}
```

4. 创建实现了 MediaPlayer 接口的实体类
```java
// AudioPlayer.java


public class AudioPlayer implements MediaPlayer {
   MediaAdapter mediaAdapter;

   @Override
   public void play(String audioType, String fileName) {

      //播放 mp3 音乐文件的内置支持
      if(audioType.equalsIgnoreCase("mp3")){
         System.out.println("Playing mp3 file. Name: "+ fileName);
      }
      //mediaAdapter 提供了播放其他文件格式的支持
      else if(audioType.equalsIgnoreCase("vlc")
         || audioType.equalsIgnoreCase("mp4")){
         mediaAdapter = new MediaAdapter(audioType);
         mediaAdapter.play(audioType, fileName);
      }
      else{
         System.out.println("Invalid media. "+
            audioType + " format not supported");
      }
   }
}
```

5. 使用 AudioPlayer 来播放不同类型的音频格式
```java
// AdapterPatternDemo.java

public class AdapterPatternDemo {
   public static void main(String[] args) {
      AudioPlayer audioPlayer = new AudioPlayer();

      audioPlayer.play("mp3", "beyond the horizon.mp3");
      audioPlayer.play("mp4", "alone.mp4");
      audioPlayer.play("vlc", "far far away.vlc");
      audioPlayer.play("avi", "mind me.avi");
   }
}
```

### 桥接模式

桥接（Bridge）是用于把抽象化与实现化解耦，使得二者可以独立变化。这种类型的设计模式属于结构型模式，它通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦。
<br/>
这种模式涉及到一个作为桥接的接口，使得实体类的功能独立于接口实现类。这两种类型的类可被结构化改变而互不影响。

#### 意图
将抽象部分与实现部分分离，使它们都可以独立的变化

#### 主要解决
在有多种可能会变化的情况下，用继承会造成类爆炸问题，扩展起来不灵活

#### 何时使用
实现系统可能有多个角度分类，每一种角度都可能变化

#### 应用实例
墙上的开关，可以看到的开关是抽象的，不用管里面具体怎么实现的。

#### 优点
1. 抽象和实现的分离
2. 优秀的扩展能力
3. 实现细节对客户透明

#### 缺点
桥接模式的引入会增加系统的理解与设计难度，由于聚合关联关系建立在抽象层，要求开发者针对抽象进行设计与编程

#### 使用场景
1. 如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系
2. 对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用
3. 一个类存在两个独立变化的维度，且这两个维度都需要进行扩展。 

> 对于两个独立变化的维度，使用桥接模式再适合不过了

#### 实现

<div align=center><img src='/pic/c&c++/dp-bridge.svg'/></div>
<center>桥接模式</center>

#### 例子
1. 创建桥接实现接口

```java
// DrawAPI.java

public interface DrawAPI {
   public void drawCircle(int radius, int x, int y);
}
```

2. 创建实现了 DrawAPI 接口的实体桥接实现类
```java
// RedCircle.java

public class RedCircle implements DrawAPI {
   @Override
   public void drawCircle(int radius, int x, int y) {
      System.out.println("Drawing Circle[ color: red, radius: "
         + radius +", x: " +x+", "+ y +"]");
   }
}
```

```java
// GreenCircle.java

public class GreenCircle implements DrawAPI {
   @Override
   public void drawCircle(int radius, int x, int y) {
      System.out.println("Drawing Circle[ color: green, radius: "
         + radius +", x: " +x+", "+ y +"]");
   }
}
```

3. 使用 DrawAPI 接口创建抽象类 Shape
```java
// Shape.java

public abstract class Shape {
   protected DrawAPI drawAPI;
   protected Shape(DrawAPI drawAPI){
      this.drawAPI = drawAPI;
   }
   public abstract void draw();
}
```

4. 创建实现了 Shape 抽象类的实体类
```java
// Circle.java

public class Circle extends Shape {
   private int x, y, radius;

   public Circle(int x, int y, int radius, DrawAPI drawAPI) {
      super(drawAPI);
      this.x = x;
      this.y = y;
      this.radius = radius;
   }

   public void draw() {
      drawAPI.drawCircle(radius,x,y);
   }
}
```

5. 使用 Shape 和 DrawAPI 类画出不同颜色的圆
```java
// BridgePatternDemo.java

public class BridgePatternDemo {
   public static void main(String[] args) {
      Shape redCircle = new Circle(100,100, 10, new RedCircle());
      Shape greenCircle = new Circle(100,100, 10, new GreenCircle());

      redCircle.draw();
      greenCircle.draw();
   }
}
```

### 过滤器模式

过滤器模式（Filter Pattern）或标准模式（Criteria Pattern）是一种设计模式，这种模式允许开发人员使用不同的标准来过滤一组对象，通过逻辑运算以解耦的方式把它们连接起来。这种类型的设计模式属于结构型模式，它结合多个标准来获得单一标准。

#### 实现

<div align=center><img src='/pic/c&c++/dp-filter.svg'/></div>
<center>过滤器模式</center>

#### 例子

1. 创建一个类，在该类上应用标准
```java
// Person.java

public class Person {

   private String name;
   private String gender;
   private String maritalStatus;

   public Person(String name,String gender,String maritalStatus){
      this.name = name;
      this.gender = gender;
      this.maritalStatus = maritalStatus;
   }

   public String getName() {
      return name;
   }
   public String getGender() {
      return gender;
   }
   public String getMaritalStatus() {
      return maritalStatus;
   }
}

```

2. 为标准（Criteria）创建一个接口
```java
// Criteria.java

import java.util.List;

public interface Criteria {
   public List<Person> meetCriteria(List<Person> persons);
}

```

3. 创建实现了 Criteria 接口的实体类
```java
// CriteriaMale.java

import java.util.ArrayList;
import java.util.List;

public class CriteriaMale implements Criteria {

   @Override
   public List<Person> meetCriteria(List<Person> persons) {
      List<Person> malePersons = new ArrayList<Person>();
      for (Person person : persons) {
         if(person.getGender().equalsIgnoreCase("MALE")){
            malePersons.add(person);
         }
      }
      return malePersons;
   }
}

```

```java
// CriteriaFemale.java

import java.util.ArrayList;
import java.util.List;

public class CriteriaFemale implements Criteria {

   @Override
   public List<Person> meetCriteria(List<Person> persons) {
      List<Person> femalePersons = new ArrayList<Person>();
      for (Person person : persons) {
         if(person.getGender().equalsIgnoreCase("FEMALE")){
            femalePersons.add(person);
         }
      }
      return femalePersons;
   }
}

```

```java
// CriteriaSingle.java

import java.util.ArrayList;
import java.util.List;

public class CriteriaSingle implements Criteria {

   @Override
   public List<Person> meetCriteria(List<Person> persons) {
      List<Person> singlePersons = new ArrayList<Person>();
      for (Person person : persons) {
         if(person.getMaritalStatus().equalsIgnoreCase("SINGLE")){
            singlePersons.add(person);
         }
      }
      return singlePersons;
   }
}
```

```java
// AndCriteria.java

import java.util.List;

public class AndCriteria implements Criteria {

   private Criteria criteria;
   private Criteria otherCriteria;

   public AndCriteria(Criteria criteria, Criteria otherCriteria) {
      this.criteria = criteria;
      this.otherCriteria = otherCriteria;
   }

   @Override
   public List<Person> meetCriteria(List<Person> persons) {
      List<Person> firstCriteriaPersons = criteria.meetCriteria(persons);
      return otherCriteria.meetCriteria(firstCriteriaPersons);
   }
}

```

```java
// OrCriteria.java

import java.util.List;

public class OrCriteria implements Criteria {

   private Criteria criteria;
   private Criteria otherCriteria;

   public OrCriteria(Criteria criteria, Criteria otherCriteria) {
      this.criteria = criteria;
      this.otherCriteria = otherCriteria;
   }

   @Override
   public List<Person> meetCriteria(List<Person> persons) {
      List<Person> firstCriteriaItems = criteria.meetCriteria(persons);
      List<Person> otherCriteriaItems = otherCriteria.meetCriteria(persons);

      for (Person person : otherCriteriaItems) {
         if(!firstCriteriaItems.contains(person)){
           firstCriteriaItems.add(person);
         }
      }
      return firstCriteriaItems;
   }
}

```

4. 使用不同的标准（Criteria）和它们的结合来过滤 Person 对象的列表
```java
// CriteriaPatternDemo.java

import java.util.ArrayList;
import java.util.List;

public class CriteriaPatternDemo {
   public static void main(String[] args) {
      List<Person> persons = new ArrayList<Person>();

      persons.add(new Person("Robert","Male", "Single"));
      persons.add(new Person("John","Male", "Married"));
      persons.add(new Person("Laura","Female", "Married"));
      persons.add(new Person("Diana","Female", "Single"));
      persons.add(new Person("Mike","Male", "Single"));
      persons.add(new Person("Bobby","Male", "Single"));

      Criteria male = new CriteriaMale();
      Criteria female = new CriteriaFemale();
      Criteria single = new CriteriaSingle();
      Criteria singleMale = new AndCriteria(single, male);
      Criteria singleOrFemale = new OrCriteria(single, female);

      System.out.println("Males: ");
      printPersons(male.meetCriteria(persons));

      System.out.println("\nFemales: ");
      printPersons(female.meetCriteria(persons));

      System.out.println("\nSingle Males: ");
      printPersons(singleMale.meetCriteria(persons));

      System.out.println("\nSingle Or Females: ");
      printPersons(singleOrFemale.meetCriteria(persons));
   }

   public static void printPersons(List<Person> persons){
      for (Person person : persons) {
         System.out.println("Person : [ Name : " + person.getName()
            +", Gender : " + person.getGender()
            +", Marital Status : " + person.getMaritalStatus()
            +" ]");
      }
   }
}

```

### 组合模式

组合模式（Composite Pattern），又叫部分整体模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。这种类型的设计模式属于结构型模式，它创建了对象组的树形结构。

#### 意图
将对象组合成树形结构以表示"部分-整体"的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性

#### 主要解决
它在我们树型结构的问题中，模糊了简单元素和复杂元素的概念，客户程序可以像处理简单元素一样来处理复杂元素，从而使得客户程序与复杂元素的内部结构解耦。

#### 如何使用
1. 您想表示对象的部分-整体层次结构（树形结构）
2. 您希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象

#### 如何解决
树枝和叶子实现统一接口，树枝内部组合该接口

#### 关键代码
树枝内部组合该接口，并且含有内部属性 List，里面放 Component

#### 应用实例
1. 算术表达式包括操作数、操作符和另一个操作数，其中，另一个操作数也可以是操作数、操作符和另一个操作数
2. 在 JAVA AWT 和 SWING 中，对于 Button 和 Checkbox 是树叶，Container 是树枝

#### 优点
1. 高层模块调用简单
2. 节点自由增加

#### 缺点
在使用组合模式时，其叶子和树枝的声明都是实现类，而不是接口，违反了依赖倒置原则

#### 使用场景
部分、整体场景，如树形菜单，文件、文件夹的管理

> 注意事项：定义时为具体类

#### 实现

<div align=center><img src='/pic/c&c++/dp-zuhe.svg'/></div>
<center>组合模式</center>

#### 例子

1. 创建 Employee 类，该类带有 Employee 对象的列表
```java
// Employee.java

import java.util.ArrayList;
import java.util.List;

public class Employee {
   private String name;
   private String dept;
   private int salary;
   private List<Employee> subordinates;

   //构造函数
   public Employee(String name,String dept, int sal) {
      this.name = name;
      this.dept = dept;
      this.salary = sal;
      subordinates = new ArrayList<Employee>();
   }

   public void add(Employee e) {
      subordinates.add(e);
   }

   public void remove(Employee e) {
      subordinates.remove(e);
   }

   public List<Employee> getSubordinates(){
     return subordinates;
   }

   public String toString(){
      return ("Employee :[ Name : "+ name
      +", dept : "+ dept + ", salary :"
      + salary+" ]");
   }
}

```

2. 使用 Employee 类来创建和打印员工的层次结构
```java
// CompositePatternDemo.java

public class CompositePatternDemo {
   public static void main(String[] args) {
      Employee CEO = new Employee("John","CEO", 30000);

      Employee headSales = new Employee("Robert","Head Sales", 20000);

      Employee headMarketing = new Employee("Michel","Head Marketing", 20000);

      Employee clerk1 = new Employee("Laura","Marketing", 10000);
      Employee clerk2 = new Employee("Bob","Marketing", 10000);

      Employee salesExecutive1 = new Employee("Richard","Sales", 10000);
      Employee salesExecutive2 = new Employee("Rob","Sales", 10000);

      CEO.add(headSales);
      CEO.add(headMarketing);

      headSales.add(salesExecutive1);
      headSales.add(salesExecutive2);

      headMarketing.add(clerk1);
      headMarketing.add(clerk2);

      //打印该组织的所有员工
      System.out.println(CEO);
      for (Employee headEmployee : CEO.getSubordinates()) {
         System.out.println(headEmployee);
         for (Employee employee : headEmployee.getSubordinates()) {
            System.out.println(employee);
         }
      }
   }
}
```

### 装饰器模式

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。
<br/>
这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。

#### 意图
动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活

#### 主要解决
一般的，我们为了扩展一个类经常使用继承方式实现，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀

#### 何时使用
在不想增加很多子类的情况下扩展类

#### 如果解决
将具体功能职责划分，同时继承装饰者模式

#### 关键代码
1. Component 类充当抽象角色，不应该具体实现
2. 修饰类引用和继承 Component 类，具体扩展类重写父类方法

#### 应用实例
不论一幅画有没有画框都可以挂在墙上，但是通常都是有画框的，并且实际上是画框被挂在墙上。在挂在墙上之前，画可以被蒙上玻璃，装到框子里；这时画、玻璃和画框形成了一个物体

#### 优点
装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能

#### 缺点
多层装饰比较复杂

#### 使用场景
1. 扩展一个类的功能
2. 动态增加功能，动态撤销

> 注意事项：可替代继承

#### 实现

<div align=center><img src='/pic/c&c++/dp-zsq.svg'/></div>
<center>装饰器模式</center>

#### 例子
1. 创建一个接口
```java
// Shape.java

public interface Shape {
   void draw();
}
```

2. 创建实现接口的实体类
```java
// Rectangle.java

public class Rectangle implements Shape {

   @Override
   public void draw() {
      System.out.println("Shape: Rectangle");
   }
}
```

```java
// Circle.java

public class Circle implements Shape {

   @Override
   public void draw() {
      System.out.println("Shape: Circle");
   }
}
```

3. 创建实现了 Shape 接口的抽象装饰类
```java
// ShapeDecorator.java

public abstract class ShapeDecorator implements Shape {
   protected Shape decoratedShape;

   public ShapeDecorator(Shape decoratedShape){
      this.decoratedShape = decoratedShape;
   }

   public void draw(){
      decoratedShape.draw();
   }
}
```

4. 创建扩展了 ShapeDecorator 类的实体装饰类
```java
// RedShapeDecorator.java

public class RedShapeDecorator extends ShapeDecorator {

   public RedShapeDecorator(Shape decoratedShape) {
      super(decoratedShape);
   }

   @Override
   public void draw() {
      decoratedShape.draw();
      setRedBorder(decoratedShape);
   }

   private void setRedBorder(Shape decoratedShape){
      System.out.println("Border Color: Red");
   }
}
```

5. 使用 RedShapeDecorator 来装饰 Shape 对象
```java
// DecoratorPatternDemo.java

public class DecoratorPatternDemo {
   public static void main(String[] args) {

      Shape circle = new Circle();
      ShapeDecorator redCircle = new RedShapeDecorator(new Circle());
      ShapeDecorator redRectangle = new RedShapeDecorator(new Rectangle());
      //Shape redCircle = new RedShapeDecorator(new Circle());
      //Shape redRectangle = new RedShapeDecorator(new Rectangle());
      System.out.println("Circle with normal border");
      circle.draw();

      System.out.println("\nCircle of red border");
      redCircle.draw();

      System.out.println("\nRectangle of red border");
      redRectangle.draw();
   }
}
```

### 外观模式
外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。这种类型的设计模式属于结构型模式，它向现有的系统添加一个接口，来隐藏系统的复杂性。
<br/>
这种模式涉及到一个单一的类，该类提供了客户端请求的简化方法和对现有系统类方法的委托调用。

#### 意图
为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用

#### 主要解决
降低访问复杂系统的内部子系统时的复杂度，简化客户端之间的接口

#### 何时使用
1. 客户端不需要知道系统内部的复杂联系，整个系统只需提供一个"接待员"即可
2. 定义系统的入口

#### 如何解决
客户端不与系统耦合，外观类与系统耦合

#### 关键代码
在客户端和复杂系统之间再加一层，这一层将调用顺序、依赖关系等处理好

#### 应用实例
去医院看病，可能要去挂号、门诊、划价、取药，让患者或患者家属觉得很复杂，如果有提供接待人员，只让接待人员来处理，就很方便

#### 优点
1. 减少系统相互依赖
2. 提高灵活性
3. 提高了安全性。 

#### 缺点
不符合开闭原则，如果要改东西很麻烦，继承重写都不合适

#### 使用场景
1. 为复杂的模块或子系统提供外界访问的模块
2. 子系统相对独立 
3. 预防低水平人员带来的风险

> 注意事项：在层次化结构中，可以使用外观模式定义系统中每一层的入口

#### 实现

<div align=center><img src='/pic/c&c++/dp-wg.svg'/></div>
<center>外观模式</center>

#### 例子

1. 创建一个接口
```java
// Shape.java

public interface Shape {
   void draw();
}

```

2. 创建实现接口的实体类
```java
// Rectangle.java

public class Rectangle implements Shape {

   @Override
   public void draw() {
      System.out.println("Rectangle::draw()");
   }
}
```

```java
// Square.java

public class Square implements Shape {

   @Override
   public void draw() {
      System.out.println("Square::draw()");
   }
}

```

```java
// Circle.java

public class Circle implements Shape {

   @Override
   public void draw() {
      System.out.println("Circle::draw()");
   }
}
```

3. 创建一个外观类

```java
// ShapeMaker.java

public class ShapeMaker {
   private Shape circle;
   private Shape rectangle;
   private Shape square;

   public ShapeMaker() {
      circle = new Circle();
      rectangle = new Rectangle();
      square = new Square();
   }

   public void drawCircle(){
      circle.draw();
   }
   public void drawRectangle(){
      rectangle.draw();
   }
   public void drawSquare(){
      square.draw();
   }
}

```

4. 使用该外观类画出各种类型的形状
```java
// FacadePatternDemo.java

public class FacadePatternDemo {
   public static void main(String[] args) {
      ShapeMaker shapeMaker = new ShapeMaker();

      shapeMaker.drawCircle();
      shapeMaker.drawRectangle();
      shapeMaker.drawSquare();
   }
}
```

### 享元模式
享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能。这种类型的设计模式属于结构型模式，它提供了减少对象数量从而改善应用所需的对象结构的方式。
<br/>
享元模式尝试重用现有的同类对象，如果未找到匹配的对象，则创建新对象。我们将通过创建 5 个对象来画出 20 个分布于不同位置的圆来演示这种模式。由于只有 5 种可用的颜色，所以 color 属性被用来检查现有的 Circle 对象。

#### 意图
运用共享技术有效地支持大量细粒度的对象

#### 主要解决
在有大量对象时，有可能会造成内存溢出，我们把其中共同的部分抽象出来，如果有相同的业务请求，直接返回在内存中已有的对象，避免重新创建

#### 何时使用
1. 系统中有大量对象
2. 这些对象消耗大量内存
3. 这些对象的状态大部分可以外部化
4. 这些对象可以按照内蕴状态分为很多组，当把外蕴对象从对象中剔除出来时，每一组对象都可以用一个对象来代替
5. 系统不依赖于这些对象身份，这些对象是不可分辨的。 

#### 如何解决
用唯一标识码判断，如果在内存中有，则返回这个唯一标识码所标识的对象

#### 关键代码
用 HashMap 存储这些对象

#### 优点
大大减少对象的创建，降低系统的内存，使效率提高

#### 缺点
提高了系统的复杂度，需要分离出外部状态和内部状态，而且外部状态具有固有化的性质，不应该随着内部状态的变化而变化，否则会造成系统的混乱

#### 使用场景
1. 系统有大量相似对象
2. 需要缓冲池的场景

> 注意事项：
>   1. 注意划分外部状态和内部状态，否则可能会引起线程安全问题
>   2. 这些类必须有一个工厂对象加以控制

#### 实现

<div align=center><img src='/pic/c&c++/dp-xy.svg'/></div>
<center>享元模式</center>

#### 例子

1. 创建一个接口
```java
// Shape.java

public interface Shape {
   void draw();
}

```

2. 创建实现接口的实体类
```java
// Circle.java

public class Circle implements Shape {
   private String color;
   private int x;
   private int y;
   private int radius;

   public Circle(String color){
      this.color = color;
   }

   public void setX(int x) {
      this.x = x;
   }

   public void setY(int y) {
      this.y = y;
   }

   public void setRadius(int radius) {
      this.radius = radius;
   }

   @Override
   public void draw() {
      System.out.println("Circle: Draw() [Color : " + color
         +", x : " + x +", y :" + y +", radius :" + radius);
   }
}

```

3. 创建一个工厂，生成基于给定信息的实体类的对象

```java
// ShapeFactory.java

import java.util.HashMap;

public class ShapeFactory {
   private static final HashMap<String, Shape> circleMap = new HashMap<>();

   public static Shape getCircle(String color) {
      Circle circle = (Circle)circleMap.get(color);

      if(circle == null) {
         circle = new Circle(color);
         circleMap.put(color, circle);
         System.out.println("Creating circle of color : " + color);
      }
      return circle;
   }
}

```

4. 使用该工厂，通过传递颜色信息来获取实体类的对象
```java
// FlyweightPatternDemo.java

public class FlyweightPatternDemo {
   private static final String colors[] =
      { "Red", "Green", "Blue", "White", "Black" };
   public static void main(String[] args) {

      for(int i=0; i < 20; ++i) {
         Circle circle =
            (Circle)ShapeFactory.getCircle(getRandomColor());
         circle.setX(getRandomX());
         circle.setY(getRandomY());
         circle.setRadius(100);
         circle.draw();
      }
   }
   private static String getRandomColor() {
      return colors[(int)(Math.random()*colors.length)];
   }
   private static int getRandomX() {
      return (int)(Math.random()*100 );
   }
   private static int getRandomY() {
      return (int)(Math.random()*100);
   }
}

```

### 代理模式

在代理模式（Proxy Pattern）中，一个类代表另一个类的功能。这种类型的设计模式属于结构型模式

#### 意图
为其他对象提供一种代理以控制对这个对象的访问

#### 主要解决
在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层

#### 何时使用
想在访问一个类时做一些控制

#### 如何解决
增加中间层

#### 应用实例
买火车票不一定在火车站买，也可以去代售点

#### 优点
1. 职责清晰
2. 高扩展性
3. 智能化

#### 缺点
1. 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢
2. 实现代理模式需要额外的工作，有些代理模式的实现非常复杂。 

#### 使用场景
按职责来划分，通常有以下使用场景： 
1. 远程代理
2. 虚拟代理
3. Copy-on-Write 代理
4. 保护（Protect or Access）代理
5. Cache代理
6. 防火墙（Firewall）代理
7. 同步化（Synchronization）代理
8. 智能引用（Smart Reference）代理

> 注意事项： 
>   1. 和适配器模式的区别：适配器模式主要改变所考虑对象的接口，而代理模式不能改变所代理类的接口
>   2. 和装饰器模式的区别：装饰器模式为了增强功能，而代理模式是为了加以控制

#### 实现

<div align=center><img src='/pic/c&c++/dp-proxy.svg'/></div>
<center>代理模式</center>

#### 例子
1. 创建一个接口
```java
// Image.java

public interface Image {
   void display();
}

```

2. 创建实现接口的实体类
```java
// RealImage.java

public class RealImage implements Image {

   private String fileName;

   public RealImage(String fileName){
      this.fileName = fileName;
      loadFromDisk(fileName);
   }

   @Override
   public void display() {
      System.out.println("Displaying " + fileName);
   }

   private void loadFromDisk(String fileName){
      System.out.println("Loading " + fileName);
   }
}

```

```java
// ProxyImage.java


public class ProxyImage implements Image{

   private RealImage realImage;
   private String fileName;

   public ProxyImage(String fileName){
      this.fileName = fileName;
   }

   @Override
   public void display() {
      if(realImage == null){
         realImage = new RealImage(fileName);
      }
      realImage.display();
   }
}

```

3. 当被请求时，使用 ProxyImage 来获取 RealImage 类的对象
```java
// ProxyPatternDemo.java

public class ProxyPatternDemo {

   public static void main(String[] args) {
      Image image = new ProxyImage("test_10mb.jpg");

      // 图像将从磁盘加载
      image.display();
      System.out.println("");
      // 图像不需要从磁盘加载
      image.display();
   }
}

```

### 责任链模式

责任链模式（Chain of Responsibility Pattern）为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。
<br/>
在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。

#### 意图
避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止

#### 主要解决
职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，所以职责链将请求的发送者和请求的处理者解耦了

#### 何时使用
在处理消息的时候以过滤很多道

#### 如何解决
拦截的类都实现统一接口

#### 关键代码
Handler 里面聚合它自己，在 HandlerRequest 里判断是否合适，如果没达到条件则向下传递，向谁传递之前 set 进去

#### 优点
1. 降低耦合度。它将请求的发送者和接收者解耦
2. 简化了对象。使得对象不需要知道链的结构
3. 增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任
4. 增加新的请求处理类很方便

#### 缺点
1. 不能保证请求一定被接收
2. 系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用
3. 可能不容易观察运行时的特征，有碍于除错

#### 使用场景
 1. 有多个对象可以处理同一个请求，具体哪个对象处理该请求由运行时刻自动确定
 2. 在不明确指定接收者的情况下，向多个对象中的一个提交一个请求
 3. 可动态指定一组对象处理请求。 

#### 实现

<div align=center><img src='/pic/c&c++/dp-zr.svg'/></div>
<center>责任链模式</center>

#### 例子

1. 创建抽象的记录器类
```java
// AbstractLogger.java

public abstract class AbstractLogger {
   public static int INFO = 1;
   public static int DEBUG = 2;
   public static int ERROR = 3;

   protected int level;

   //责任链中的下一个元素
   protected AbstractLogger nextLogger;

   public void setNextLogger(AbstractLogger nextLogger){
      this.nextLogger = nextLogger;
   }

   public void logMessage(int level, String message){
      if(this.level <= level){
         write(message);
      }
      if(nextLogger !=null){
         nextLogger.logMessage(level, message);
      }
   }

   abstract protected void write(String message);

}
```

2. 创建扩展了该记录器类的实体类
```java
// ConsoleLogger.java

public class ConsoleLogger extends AbstractLogger {

   public ConsoleLogger(int level){
      this.level = level;
   }

   @Override
   protected void write(String message) {
      System.out.println("Standard Console::Logger: " + message);
   }
}

```

```java
// ErrorLogger.java

public class ErrorLogger extends AbstractLogger {

   public ErrorLogger(int level){
      this.level = level;
   }

   @Override
   protected void write(String message) {
      System.out.println("Error Console::Logger: " + message);
   }
}

```

```java
// FileLogger.java

public class FileLogger extends AbstractLogger {

   public FileLogger(int level){
      this.level = level;
   }

   @Override
   protected void write(String message) {
      System.out.println("File::Logger: " + message);
   }
}

```

3. 创建不同类型的记录器。赋予它们不同的错误级别，并在每个记录器中设置下一个记录器。每个记录器中的下一个记录器代表的是链的一部分
```java
// ChainPatternDemo.java

public class ChainPatternDemo {

   private static AbstractLogger getChainOfLoggers(){

      AbstractLogger errorLogger = new ErrorLogger(AbstractLogger.ERROR);
      AbstractLogger fileLogger = new FileLogger(AbstractLogger.DEBUG);
      AbstractLogger consoleLogger = new ConsoleLogger(AbstractLogger.INFO);

      errorLogger.setNextLogger(fileLogger);
      fileLogger.setNextLogger(consoleLogger);

      return errorLogger;
   }

   public static void main(String[] args) {
      AbstractLogger loggerChain = getChainOfLoggers();

      loggerChain.logMessage(AbstractLogger.INFO, "This is an information.");

      loggerChain.logMessage(AbstractLogger.DEBUG,
         "This is a debug level information.");

      loggerChain.logMessage(AbstractLogger.ERROR,
         "This is an error information.");
   }
}

```

### 命令模式

命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。

#### 意图
将一个请求封装成一个对象，从而使您可以用不同的请求对客户进行参数化

#### 主要解决
在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系，但某些场合，比如需要对行为进行记录、撤销或重做、事务等处理时，这种无法抵御变化的紧耦合的设计就不太合适

#### 何时使用
在某些场合，比如要对行为进行"记录、撤销/重做、事务"等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将"行为请求者"与"行为实现者"解耦？将一组行为抽象为对象，可以实现二者之间的松耦合

#### 如何解决
通过调用者调用接受者执行命令，顺序：调用者→命令→接受者

#### 关键代码
定义三个角色：
1. received 真正的命令执行对象 
2. Command 
3. invoker 使用命令对象的入口

#### 应用实例
struts 1 中的 action 核心控制器 ActionServlet 只有一个，相当于 Invoker，而模型层的类会随着不同的应用有不同的模型类，相当于具体的 Command

#### 优点
1. 降低了系统耦合度
2. 新的命令可以很容易添加到系统中去

#### 缺点
使用命令模式可能会导致某些系统有过多的具体命令类

#### 使用场景
认为是命令的地方都可以使用命令模式，比如： 
1. GUI 中每一个按钮都是一条命令
2. 模拟 CMD

> 注意事项：系统需要支持命令的撤销(Undo)操作和恢复(Redo)操作，也可以考虑使用命令模式，见命令模式的扩展

#### 示意图
<div align=center><img src='/pic/c&c++/dp-ml-1.jpg'/></div>
<center>命令模式示意图</center>

#### 实现
<div align=center><img src='/pic/c&c++/dp-ml-2.jpg'/></div>
<center>命令模式</center>

#### 例子
1. 创建一个命令接口
```java
// Order.java

public interface Order {
   void execute();
}

```

2. 创建一个请求类
```java
// Stock.java

public class Stock {

   private String name = "ABC";
   private int quantity = 10;

   public void buy(){
      System.out.println("Stock [ Name: "+name+",
         Quantity: " + quantity +" ] bought");
   }
   public void sell(){
      System.out.println("Stock [ Name: "+name+",
         Quantity: " + quantity +" ] sold");
   }
}

```

3. 创建实现了 Order 接口的实体类
```java
// BuyStock.java

public class BuyStock implements Order {
   private Stock abcStock;

   public BuyStock(Stock abcStock){
      this.abcStock = abcStock;
   }

   public void execute() {
      abcStock.buy();
   }
}

```

```java
// SellStock.java

public class SellStock implements Order {
   private Stock abcStock;

   public SellStock(Stock abcStock){
      this.abcStock = abcStock;
   }

   public void execute() {
      abcStock.sell();
   }
}

```

4. 创建命令调用类
```java
// Broker.java

import java.util.ArrayList;
import java.util.List;

public class Broker {
   private List<Order> orderList = new ArrayList<Order>();

   public void takeOrder(Order order){
      orderList.add(order);
   }

   public void placeOrders(){
      for (Order order : orderList) {
         order.execute();
      }
      orderList.clear();
   }
}

```

5. 使用 Broker 类来接受并执行命令
```java
// CommandPatternDemo.java

public class CommandPatternDemo {
   public static void main(String[] args) {
      Stock abcStock = new Stock();

      BuyStock buyStockOrder = new BuyStock(abcStock);
      SellStock sellStockOrder = new SellStock(abcStock);

      Broker broker = new Broker();
      broker.takeOrder(buyStockOrder);
      broker.takeOrder(sellStockOrder);

      broker.placeOrders();
   }
}
```

### 解释器模式
解释器模式（Interpreter Pattern）提供了评估语言的语法或表达式的方式，它属于行为型模式。这种模式实现了一个表达式接口，该接口解释一个特定的上下文。这种模式被用在 SQL 解析、符号处理引擎等。

#### 意图
给定一个语言，定义它的文法表示，并定义一个解释器，这个解释器使用该标识来解释语言中的句子

#### 主要解决
对于一些固定文法构建一个解释句子的解释器

#### 何时使用
如果一种特定类型的问题发生的频率足够高，那么可能就值得将该问题的各个实例表述为一个简单语言中的句子。这样就可以构建一个解释器，该解释器通过解释这些句子来解决该问题

#### 如何解决
构建语法树，定义终结符与非终结符

#### 关键代码
构建环境类，包含解释器之外的一些全局信息，一般是 HashMap

#### 应用实例
编译器、运算表达式计算

#### 优点
1. 可扩展性比较好，灵活
2. 增加了新的解释表达式的方式
3. 易于实现简单文法

#### 缺点
1. 可利用场景比较少
2. 对于复杂的文法比较难维护
3. 解释器模式会引起类膨胀
4. 解释器模式采用递归调用方法

#### 使用场景
1. 可以将一个需要解释执行的语言中的句子表示为一个抽象语法树
2. 一些重复出现的问题可以用一种简单的语言来进行表达
3. 一个简单语法需要解释的场景。

> 注意事项：可利用场景比较少，JAVA 中如果碰到可以用 expression4J 代替

#### 实现
<div align=center><img src='/pic/c&c++/dp-jsq.jpg'/></div>
<center>解释器模式</center>

#### 例子
1. 创建一个表达式接口
```java
// Expression.java

public interface Expression {
   public boolean interpret(String context);
}

```

2. 创建实现了上述接口的实体类
```java
// TerminalExpression.java

public class TerminalExpression implements Expression {

   private String data;

   public TerminalExpression(String data){
      this.data = data;
   }

   @Override
   public boolean interpret(String context) {
      if(context.contains(data)){
         return true;
      }
      return false;
   }
}

```

```java
// OrExpression.java

public class OrExpression implements Expression {

   private Expression expr1 = null;
   private Expression expr2 = null;

   public OrExpression(Expression expr1, Expression expr2) {
      this.expr1 = expr1;
      this.expr2 = expr2;
   }

   @Override
   public boolean interpret(String context) {
      return expr1.interpret(context) || expr2.interpret(context);
   }
}

```

```java
// AndExpression.java

public class AndExpression implements Expression {

   private Expression expr1 = null;
   private Expression expr2 = null;

   public AndExpression(Expression expr1, Expression expr2) {
      this.expr1 = expr1;
      this.expr2 = expr2;
   }

   @Override
   public boolean interpret(String context) {
      return expr1.interpret(context) && expr2.interpret(context);
   }
}

```

3. InterpreterPatternDemo 使用 Expression 类来创建规则，并解析它们
```java
// InterpreterPatternDemo.java

public class InterpreterPatternDemo {

   //规则：Robert 和 John 是男性
   public static Expression getMaleExpression(){
      Expression robert = new TerminalExpression("Robert");
      Expression john = new TerminalExpression("John");
      return new OrExpression(robert, john);
   }

   //规则：Julie 是一个已婚的女性
   public static Expression getMarriedWomanExpression(){
      Expression julie = new TerminalExpression("Julie");
      Expression married = new TerminalExpression("Married");
      return new AndExpression(julie, married);
   }

   public static void main(String[] args) {
      Expression isMale = getMaleExpression();
      Expression isMarriedWoman = getMarriedWomanExpression();

      System.out.println("John is male? " + isMale.interpret("John"));
      System.out.println("Julie is a married women? "
      + isMarriedWoman.interpret("Married Julie"));
   }
}

```
### 迭代器模式

这种模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示
<br/>
迭代器模式属于行为型模式

#### 意图
提供一种方法顺序访问一个聚合对象中各个元素, 而又无须暴露该对象的内部表示

#### 主要解决
不同的方式来遍历整个整合对象

#### 何时使用
遍历一个聚合对象

#### 如何解决
把在元素之间游走的责任交给迭代器，而不是聚合对象

#### 关键代码
定义接口：hasNext, next

#### 优点
1. 它支持以不同的方式遍历一个聚合对象
2. 迭代器简化了聚合类
3. 在同一个聚合上可以有多个遍历
4. 在迭代器模式中，增加新的聚合类和迭代器类都很方便，无须修改原有代码。 

#### 缺点
由于迭代器模式将存储数据和遍历数据的职责分离，增加新的聚合类需要对应增加新的迭代器类，类的个数成对增加，这在一定程度上增加了系统的复杂性

#### 使用场景
1. 访问一个聚合对象的内容而无须暴露它的内部表示
2. 需要为聚合对象提供多种遍历方式
3. 为遍历不同的聚合结构提供一个统一的接口

> 注意事项：迭代器模式就是分离了集合对象的遍历行为，抽象出一个迭代器类来负责，这样既可以做到不暴露集合的内部结构，又可让外部代码透明地访问集合内部的数据

#### 实现
<div align=center><img src='/pic/c&c++/dp-i.png'/></div>
<center>迭代器模式</center>

#### 例子 

1. 创建接口
```java
// Iterator.java

public interface Iterator {
   public boolean hasNext();
   public Object next();
}

```

```java
// Container.java

public interface Container {
   public Iterator getIterator();
}

```

2. 创建实现了 Container 接口的实体类。该类有实现了 Iterator 接口的内部类 NameIterator

```java
// NameRepository.java

public class NameRepository implements Container {
   public String[] names = {"Robert" , "John" ,"Julie" , "Lora"};

   @Override
   public Iterator getIterator() {
      return new NameIterator();
   }

   private class NameIterator implements Iterator {

      int index;

      @Override
      public boolean hasNext() {
         if(index < names.length){
            return true;
         }
         return false;
      }

      @Override
      public Object next() {
         if(this.hasNext()){
            return names[index++];
         }
         return null;
      }
   }
}

```

3. 使用 NameRepository 来获取迭代器，并打印名字
```java
// IteratorPatternDemo.java

public class IteratorPatternDemo {

   public static void main(String[] args) {
      NameRepository namesRepository = new NameRepository();

      for(Iterator iter = namesRepository.getIterator(); iter.hasNext();){
         String name = (String)iter.next();
         System.out.println("Name : " + name);
      }
   }
}

```
### 中介者模式
中介者模式（Mediator Pattern）是用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护。中介者模式属于行为型模式。

#### 意图
用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互

#### 主要解决
对象与对象之间存在大量的关联关系，这样势必会导致系统的结构变得很复杂，同时若一个对象发生改变，我们也需要跟踪与之相关联的对象，同时做出相应的处理

#### 何时使用
多个类相互耦合，形成了网状结构

#### 如何解决
将上述网状结构分离为星型结构

#### 应用实例
1. 中国加入 WTO 之前是各个国家相互贸易，结构复杂，现在是各个国家通过 WTO 来互相贸易
2. 机场调度系统
3. MVC 框架，其中C（控制器）就是 M（模型）和 V（视图）的中介者

#### 优点
1. 降低了类的复杂度，将一对多转化成了一对一
2. 各个类之间的解耦
3. 符合迪米特原则

#### 缺点
中介者会庞大，变得复杂难以维护

#### 使用场景
1. 系统中对象之间存在比较复杂的引用关系，导致它们之间的依赖关系结构混乱而且难以复用该对象
2. 想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类

#### 实现
<div align=center><img src='/pic/c&c++/dp-zj.jpg'/></div>
<center>中介者模式</center>

#### 例子

1. 创建中介类
```java
// ChatRoom.java

import java.util.Date;

public class ChatRoom {
   public static void showMessage(User user, String message){
      System.out.println(new Date().toString()
         + " [" + user.getName() +"] : " + message);
   }
}

```

1. 创建 user 类
```java
// User.java

public class User {
   private String name;

   public String getName() {
      return name;
   }

   public void setName(String name) {
      this.name = name;
   }

   public User(String name){
      this.name  = name;
   }

   public void sendMessage(String message){
      ChatRoom.showMessage(this,message);
   }
}

```

3. 使用 User 对象来显示他们之间的通信
```java
// MediatorPatternDemo.java

public class MediatorPatternDemo {
   public static void main(String[] args) {
      User robert = new User("Robert");
      User john = new User("John");

      robert.sendMessage("Hi! John!");
      john.sendMessage("Hello! Robert!");
   }
}

```

### 备忘录模式
备忘录模式（Memento Pattern）保存一个对象的某个状态，以便在适当的时候恢复对象。备忘录模式属于行为型模式。

#### 意图
在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态

#### 主要解决
所谓备忘录模式就是在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态

#### 何时使用
很多时候我们总是需要记录一个对象的内部状态，这样做的目的就是为了允许用户取消不确定或者错误的操作，能够恢复到他原先的状态，使得他有"后悔药"可吃

#### 如何解决
通过一个备忘录类专门存储对象状态

#### 关键代码
客户不与备忘录类耦合，与备忘录管理类耦合

#### 应用实例
1. 打游戏时的存档
2. Windows 里的 `ctrl + z`
3. IE 中的后退
4. 数据库的事务管理

#### 优点
1. 给用户提供了一种可以恢复状态的机制，可以使用户能够比较方便地回到某个历史的状态
2. 实现了信息的封装，使得用户不需要关心状态的保存细节。

#### 缺点
消耗资源。如果类的成员变量过多，势必会占用比较大的资源，而且每一次保存都会消耗一定的内存

#### 使用场景
1. 需要保存/恢复数据的相关状态场景
2. 提供一个可回滚的操作

#### 注意事项
1. 为了符合迪米特原则，还要增加一个管理备忘录的类
2. 为了节约内存，可使用原型模式+备忘录模式

#### 实现
<div align=center><img src='/pic/c&c++/dp-bw.svg'/></div>
<center>备忘录模式</center>

#### 例子
1. 创建 Memento 类
```java
// Memento.java

public class Memento {
   private String state;

   public Memento(String state){
      this.state = state;
   }

   public String getState(){
      return state;
   }
}

```

2. 创建 Originator 类
```java
// Originator.java

public class Originator {
   private String state;

   public void setState(String state){
      this.state = state;
   }

   public String getState(){
      return state;
   }

   public Memento saveStateToMemento(){
      return new Memento(state);
   }

   public void getStateFromMemento(Memento Memento){
      state = Memento.getState();
   }
}

```

3. 创建 CareTaker 类
```java
// CareTaker.java

import java.util.ArrayList;
import java.util.List;

public class CareTaker {
   private List<Memento> mementoList = new ArrayList<Memento>();

   public void add(Memento state){
      mementoList.add(state);
   }

   public Memento get(int index){
      return mementoList.get(index);
   }
}

```

4. 使用 CareTaker 和 Originator 对象
```java
// MementoPatternDemo.java

public class MementoPatternDemo {
   public static void main(String[] args) {
      Originator originator = new Originator();
      CareTaker careTaker = new CareTaker();
      originator.setState("State #1");
      originator.setState("State #2");
      careTaker.add(originator.saveStateToMemento());
      originator.setState("State #3");
      careTaker.add(originator.saveStateToMemento());
      originator.setState("State #4");

      System.out.println("Current State: " + originator.getState());
      originator.getStateFromMemento(careTaker.get(0));
      System.out.println("First saved State: " + originator.getState());
      originator.getStateFromMemento(careTaker.get(1));
      System.out.println("Second saved State: " + originator.getState());
   }
}

```

### 观察者模式
当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知依赖它的对象。观察者模式属于行为型模式

#### 意图
定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新

#### 主要解决
一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作

#### 何时使用
一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知

#### 如何解决
使用面向对象技术，可以将这种依赖关系弱化

#### 关键代码
在抽象类里有一个 ArrayList 存放观察者们

#### 应用实例
1. 拍卖的时候，拍卖师观察最高标价，然后通知给其他竞价者竞价

#### 优点
1. 观察者和被观察者是抽象耦合的
2. 建立一套触发机制

#### 缺点
1. 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间
2. 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃
3. 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化

#### 使用场景
1. 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用。
2. 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度。
3. 一个对象必须通知其他对象，而并不知道这些对象是谁。
4. 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制。

> 注意事项:
>   1. JAVA 中已经有了对观察者模式的支持类
>   2. 避免循环引用
>   3. 如果顺序执行，某一观察者错误会导致系统卡壳，一般采用异步方式

#### 实现
<div align=center><img src='/pic/c&c++/dp-gcz.svg'/></div>
<center>观察者模式</center>

#### 例子
1. 创建 Subject 类
```java
// Subject.java

import java.util.ArrayList;
import java.util.List;

public class Subject {

   private List<Observer> observers
      = new ArrayList<Observer>();
   private int state;

   public int getState() {
      return state;
   }

   public void setState(int state) {
      this.state = state;
      notifyAllObservers();
   }

   public void attach(Observer observer){
      observers.add(observer);
   }

   public void notifyAllObservers(){
      for (Observer observer : observers) {
         observer.update();
      }
   }
}

```

2. 创建 Observer 类
```java
// Observer.java

public abstract class Observer {
   protected Subject subject;
   public abstract void update();
}

```

3. 创建实体观察者类
```java
// BinaryObserver.java

public class BinaryObserver extends Observer{

   public BinaryObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }

   @Override
   public void update() {
      System.out.println( "Binary String: "
      + Integer.toBinaryString( subject.getState() ) );
   }
}

```

```java
// OctalObserver.java

public class OctalObserver extends Observer{

   public OctalObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }

   @Override
   public void update() {
     System.out.println( "Octal String: "
     + Integer.toOctalString( subject.getState() ) );
   }
}

```

```java
// HexaObserver.java

public class HexaObserver extends Observer{

   public HexaObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }

   @Override
   public void update() {
      System.out.println( "Hex String: "
      + Integer.toHexString( subject.getState() ).toUpperCase() );
   }
}

```

4. 使用 Subject 和实体观察者对象
```java
// ObserverPatternDemo.java

public class ObserverPatternDemo {
   public static void main(String[] args) {
      Subject subject = new Subject();

      new HexaObserver(subject);
      new OctalObserver(subject);
      new BinaryObserver(subject);

      System.out.println("First state change: 15");
      subject.setState(15);
      System.out.println("Second state change: 10");
      subject.setState(10);
   }
}

```

### 状态模式
在状态模式（State Pattern）中，类的行为是基于它的状态改变的。这种类型的设计模式属于行为型模式
在状态模式中，我们创建表示各种状态的对象和一个行为随着状态对象改变而改变的 context 对象

#### 意图
允许对象在内部状态发生改变时改变它的行为，对象看起来好像修改了它的类

#### 主要解决
对象的行为依赖于它的状态（属性），并且可以根据它的状态改变而改变它的相关行为

#### 何时使用
代码中包含大量与对象状态有关的条件语句

#### 如何解决
将各种具体的状态类抽象出来

#### 关键代码
通常命令模式的接口中只有一个方法。而状态模式的接口中有一个或者多个方法。而且，状态模式的实现类的方法，一般返回值，或者是改变实例变量的值。也就是说，状态模式一般和对象的状态有关。实现类的方法有不同的功能，覆盖接口中的方法。状态模式和命令模式一样，也可以用于消除 if...else 等条件选择语句。

#### 应用实例
1. 打篮球的时候运动员可以有正常状态、不正常状态和超常状态
2. 曾侯乙编钟中，'钟是抽象接口','钟A'等是具体状态，'曾侯乙编钟'是具体环境（Context）

#### 优点
1. 封装了转换规则
2. 枚举可能的状态，在枚举状态之前需要确定状态种类
3. 将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为
4. 允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块
5. 可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数

#### 缺点
1. 状态模式的使用必然会增加系统类和对象的个数
2. 状态模式的结构与实现都较为复杂，如果使用不当将导致程序结构和代码的混乱
3. 状态模式对"开闭原则"的支持并不太好，对于可以切换状态的状态模式，增加新的状态类需要修改那些负责状态转换的源代码，否则无法切换到新增状态，而且修改某个状态类的行为也需修改对应类的源代码。

#### 使用场景
1. 行为随状态改变而改变的场景
2. 条件、分支语句的代替者

> 注意事项：在行为受状态约束的时候使用状态模式，而且状态不超过 5 个

#### 实现
<div align=center><img src='/pic/c&c++/dp-zt.png'/></div>
<center>状态模式</center>

#### 例子

1. 创建一个接口 
```java
// State.java

public interface State {
   public void doAction(Context context);
}

```

2. 创建实现接口的实体类
```java
// StartState.java

public class StartState implements State {

   public void doAction(Context context) {
      System.out.println("Player is in start state");
      context.setState(this);
   }

   public String toString(){
      return "Start State";
   }
}

```

```java
// StopState.java

public class StopState implements State {

   public void doAction(Context context) {
      System.out.println("Player is in stop state");
      context.setState(this);
   }

   public String toString(){
      return "Stop State";
   }
}

```

3. 创建 Context 类
```java
// Context.java

public class Context {
   private State state;

   public Context(){
      state = null;
   }

   public void setState(State state){
      this.state = state;
   }

   public State getState(){
      return state;
   }
}

```

4. 使用 Context 来查看当状态 State 改变时的行为变化
```java
// StatePatternDemo.java

public class StatePatternDemo {
   public static void main(String[] args) {
      Context context = new Context();

      StartState startState = new StartState();
      startState.doAction(context);

      System.out.println(context.getState().toString());

      StopState stopState = new StopState();
      stopState.doAction(context);

      System.out.println(context.getState().toString());
   }
}

```

### 空对象模式

在空对象模式（Null Object Pattern）中，一个空对象取代 NULL 对象实例的检查。Null 对象不是检查空值，而是反应一个不做任何动作的关系。这样的 Null 对象也可以在数据不可用的时候提供默认的行为。
<br/>
在空对象模式中，我们创建一个指定各种要执行的操作的抽象类和扩展该类的实体类，还创建一个未对该类做任何实现的空对象类，该空对象类将无缝地使用在需要检查空值的地方。

#### 实现

<div align=center><img src='/pic/c&c++/dp-null.jpg'/></div>
<center>空对象模式</center>

#### 例子
1. 创建一个抽象类
```java
// AbstractCustomer.java

public abstract class AbstractCustomer {
   protected String name;
   public abstract boolean isNil();
   public abstract String getName();
}

```

2. 创建扩展了上述类的实体类
```java
// RealCustomer.java

public class RealCustomer extends AbstractCustomer {

   public RealCustomer(String name) {
      this.name = name;
   }

   @Override
   public String getName() {
      return name;
   }

   @Override
   public boolean isNil() {
      return false;
   }
}

```

```java
// NullCustomer.java

public class NullCustomer extends AbstractCustomer {

   @Override
   public String getName() {
      return "Not Available in Customer Database";
   }

   @Override
   public boolean isNil() {
      return true;
   }
}

```

3. 创建 CustomerFactory 类
```java
// CustomerFactory.java

public class CustomerFactory {

   public static final String[] names = {"Rob", "Joe", "Julie"};

   public static AbstractCustomer getCustomer(String name){
      for (int i = 0; i < names.length; i++) {
         if (names[i].equalsIgnoreCase(name)){
            return new RealCustomer(name);
         }
      }
      return new NullCustomer();
   }
}

```

4. 使用 CustomerFactory，基于客户传递的名字，来获取 RealCustomer 或 NullCustomer 对象。
```java
// NullPatternDemo.java

public class NullPatternDemo {
   public static void main(String[] args) {

      AbstractCustomer customer1 = CustomerFactory.getCustomer("Rob");
      AbstractCustomer customer2 = CustomerFactory.getCustomer("Bob");
      AbstractCustomer customer3 = CustomerFactory.getCustomer("Julie");
      AbstractCustomer customer4 = CustomerFactory.getCustomer("Laura");

      System.out.println("Customers");
      System.out.println(customer1.getName());
      System.out.println(customer2.getName());
      System.out.println(customer3.getName());
      System.out.println(customer4.getName());
   }
}

```

### 策略模式
在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。
<br/>
在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。

#### 意图
定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换

#### 主要解决
在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护

#### 何时使用
一个系统有许多许多类，而区分它们的只是他们直接的行为

#### 如何解决
将这些算法封装成一个一个的类，任意地替换

#### 关键代码
实现同一个接口

#### 应用实例
旅行的出游方式，选择骑自行车、坐汽车，每一种旅行方式都是一个策略

#### 优点
1. 算法可以自由切换
2. 避免使用多重条件判断
3. 扩展性良好

#### 缺点
1. 策略类会增多
2. 所有策略类都需要对外暴露

#### 使用场景
1. 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为
2. 一个系统需要动态地在几种算法中选择一种
3. 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现

> 注意事项：如果一个系统的策略多于四个，就需要考虑使用混合模式，解决策略类膨胀的问题

#### 实现

<div align=center><img src='/pic/c&c++/dp-cl.jpg'/></div>
<center>策略模式</center>

#### 例子

1. 创建一个接口
```java
// Strategy.java

public interface Strategy {
   public int doOperation(int num1, int num2);
}

```

2. 创建实现接口的实体类
```java
// OperationAdd.java

public class OperationAdd implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 + num2;
   }
}

```

```java
// OperationSubtract.java

public class OperationSubtract implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 - num2;
   }
}

```

```java
// OperationMultiply.java

public class OperationMultiply implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 * num2;
   }
}

```

3. 创建 Context 类
```java
// Context.java

public class Context {
   private Strategy strategy;

   public Context(Strategy strategy){
      this.strategy = strategy;
   }

   public int executeStrategy(int num1, int num2){
      return strategy.doOperation(num1, num2);
   }
}

```

4. 使用 Context 来查看当它改变策略 Strategy 时的行为变化
```java
// StrategyPatternDemo.java

public class StrategyPatternDemo {
   public static void main(String[] args) {
      Context context = new Context(new OperationAdd());
      System.out.println("10 + 5 = " + context.executeStrategy(10, 5));

      context = new Context(new OperationSubtract());
      System.out.println("10 - 5 = " + context.executeStrategy(10, 5));

      context = new Context(new OperationMultiply());
      System.out.println("10 * 5 = " + context.executeStrategy(10, 5));
   }
}

```

### 模板模式
在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。这种类型的设计模式属于行为型模式。

#### 意图
定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤

#### 主要解决
一些方法通用，却在每一个子类都重新写了这一方法

#### 何时使用
有一些通用的方法

#### 如何解决
将这些通用算法抽象出来

#### 关键代码
关键步骤在抽象类实现，其他步骤在子类实现

#### 应用实例
在造房子的时候，地基、走线、水管都一样，只有在建筑的后期才有加壁橱加栅栏等差异

#### 优点
1. 封装不变部分，扩展可变部分
2. 提取公共代码，便于维护
3. 行为由父类控制，子类实现

#### 缺点
每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大

#### 使用场景
1. 有多个子类共有的方法，且逻辑相同
2. 重要的、复杂的方法，可以考虑作为模板方法。

> 注意事项：为防止恶意操作，一般模板方法都加上 final 关键词

#### 实现

<div align=center><img src='/pic/c&c++/dp-mb.jpg'/></div>
<center>模板模式</center>

### 访问者模式
在访问者模式（Visitor Pattern）中，我们使用了一个访问者类，它改变了元素类的执行算法。通过这种方式，元素的执行算法可以随着访问者改变而改变。这种类型的设计模式属于行为型模式。根据模式，元素对象已接受访问者对象，这样访问者对象就可以处理元素对象上的操作。

#### 意图
主要将数据结构与数据操作分离

#### 主要解决
稳定的数据结构和易变的操作耦合问题

#### 何时使用
需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，使用访问者模式将这些封装到类中

#### 如何解决
在被访问的类里面加一个对外提供接待访问者的接口

#### 关键代码
在数据基础类里面有一个方法接受访问者，将自身引用传入访问者

#### 应用实例
您在朋友家做客，您是访问者，朋友接受您的访问，您通过朋友的描述，然后对朋友的描述做出一个判断，这就是访问者模式

#### 优点
1. 符合单一职责原则
2. 优秀的扩展性
3. 灵活性

#### 缺点
1. 具体元素对访问者公布细节，违反了迪米特原则
2. 具体元素变更比较困难
3. 违反了依赖倒置原则，依赖了具体类，没有依赖抽象

#### 使用场景
1. 对象结构中对象对应的类很少改变，但经常需要在此对象结构上定义新的操作
2. 需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，也不希望在增加新操作时修改这些类。

> 注意事项：访问者可以对功能进行统一，可以做报表、UI、拦截器与过滤器。

#### 实现

<div align=center><img src='/pic/c&c++/dp-fwz.jpg'/></div>
<center>访问者模式</center>

#### 例子
1. 定义一个表示元素的接口
```java
// ComputerPart.java

public interface ComputerPart {
   public void accept(ComputerPartVisitor computerPartVisitor);
}

```

2. 创建扩展了上述类的实体类
```java
// Keyboard.java

public class Keyboard  implements ComputerPart {

   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}

```

```java
// Monitor.java

public class Monitor  implements ComputerPart {

   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}

```

```java
// Mouse.java

public class Mouse  implements ComputerPart {

   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      computerPartVisitor.visit(this);
   }
}

```

```java
// Computer.java

public class Computer implements ComputerPart {

   ComputerPart[] parts;

   public Computer(){
      parts = new ComputerPart[] {new Mouse(), new Keyboard(), new Monitor()};
   }


   @Override
   public void accept(ComputerPartVisitor computerPartVisitor) {
      for (int i = 0; i < parts.length; i++) {
         parts[i].accept(computerPartVisitor);
      }
      computerPartVisitor.visit(this);
   }
}

```

3. 定义一个表示访问者的接口
```java
// ComputerPartVisitor.java

public interface ComputerPartVisitor {
   public void visit(Computer computer);
   public void visit(Mouse mouse);
   public void visit(Keyboard keyboard);
   public void visit(Monitor monitor);
}

```

4. 创建实现了上述类的实体访问者
```java
// ComputerPartDisplayVisitor.java

public class ComputerPartDisplayVisitor implements ComputerPartVisitor {

   @Override
   public void visit(Computer computer) {
      System.out.println("Displaying Computer.");
   }

   @Override
   public void visit(Mouse mouse) {
      System.out.println("Displaying Mouse.");
   }

   @Override
   public void visit(Keyboard keyboard) {
      System.out.println("Displaying Keyboard.");
   }

   @Override
   public void visit(Monitor monitor) {
      System.out.println("Displaying Monitor.");
   }
}

```

5. 使用 ComputerPartDisplayVisitor 来显示 Computer 的组成部分
```java
// VisitorPatternDemo.java

public class VisitorPatternDemo {
   public static void main(String[] args) {

      ComputerPart computer = new Computer();
      computer.accept(new ComputerPartDisplayVisitor());
   }
}

```

### MVC模式
MVC 模式代表 Model-View-Controller（模型-视图-控制器） 模式。这种模式用于应用程序的分层开发。

- Model(模型): 模型代表一个存取数据的对象或 JAVA POJO。它也可以带有逻辑，在数据变化时更新控制器。
- View(视图): 视图代表模型包含的数据的可视化。
- Controller(控制器): 控制器作用于模型和视图上。它控制数据流向模型对象，并在数据变化时更新视图。它使视图与模型分离开。

<div align=center><img src='/pic/c&c++/dp-mvc1.png'/></div>
<center>MVC</center>

#### 实现
<div align=center><img src='/pic/c&c++/dp-mvc2.svg'/></div>
<center>MVC</center>

#### 例子

1. 创建模型
```java
// Student.java

public class Student {
   private String rollNo;
   private String name;
   public String getRollNo() {
      return rollNo;
   }
   public void setRollNo(String rollNo) {
      this.rollNo = rollNo;
   }
   public String getName() {
      return name;
   }
   public void setName(String name) {
      this.name = name;
   }
}

```

2. 创建视图
```java
// StudentView.java

public class StudentView {
   public void printStudentDetails(String studentName, String studentRollNo){
      System.out.println("Student: ");
      System.out.println("Name: " + studentName);
      System.out.println("Roll No: " + studentRollNo);
   }
}

```

3. 创建控制器
```java
// StudentController.java

public class StudentController {
   private Student model;
   private StudentView view;

   public StudentController(Student model, StudentView view){
      this.model = model;
      this.view = view;
   }

   public void setStudentName(String name){
      model.setName(name);
   }

   public String getStudentName(){
      return model.getName();
   }

   public void setStudentRollNo(String rollNo){
      model.setRollNo(rollNo);
   }

   public String getStudentRollNo(){
      return model.getRollNo();
   }

   public void updateView(){
      view.printStudentDetails(model.getName(), model.getRollNo());
   }
}

```

4. 使用 StudentController 方法来演示 MVC 设计模式的用法
```java
// MVCPatternDemo.java

public class MVCPatternDemo {
   public static void main(String[] args) {

      //从数据库获取学生记录
      Student model  = retrieveStudentFromDatabase();

      //创建一个视图：把学生详细信息输出到控制台
      StudentView view = new StudentView();

      StudentController controller = new StudentController(model, view);

      controller.updateView();

      //更新模型数据
      controller.setStudentName("John");

      controller.updateView();
   }

   private static Student retrieveStudentFromDatabase(){
      Student student = new Student();
      student.setName("Robert");
      student.setRollNo("10");
      return student;
   }
}

```

### 业务代表模式

业务代表模式（Business Delegate Pattern）用于对表示层和业务层解耦。它基本上是用来减少通信或对表示层代码中的业务层代码的远程查询功能。在业务层中我们有以下实体

- 客户端(Client): 表示层代码可以是 JSP、servlet 或 UI java 代码。
- 业务代表(Business Delegate): 一个为客户端实体提供的入口类，它提供了对业务服务方法的访问。
- 查询服务(LookUp Service): 查找服务对象负责获取相关的业务实现，并提供业务对象对业务代表对象的访问。
- 业务服务(Business Service): 业务服务接口。实现了该业务服务的实体类，提供了实际的业务实现逻辑。

#### 实现
<div align=center><img src='/pic/c&c++/dp-ywdb.svg'/></div>
<center>业务代表模式</center>

#### 例子
1. 创建 BusinessService 接口
```java
// BusinessService.java

public interface BusinessService {
   public void doProcessing();
}

```

2. 创建实体服务类
```java
// EJBService.java

public class EJBService implements BusinessService {

   @Override
   public void doProcessing() {
      System.out.println("Processing task by invoking EJB Service");
   }
}

```

```java
// JMSService.java

public class JMSService implements BusinessService {

   @Override
   public void doProcessing() {
      System.out.println("Processing task by invoking JMS Service");
   }
}

```

3. 创建业务查询服务
```java
// BusinessLookUp.java

public class BusinessLookUp {
   public BusinessService getBusinessService(String serviceType){
      if(serviceType.equalsIgnoreCase("EJB")){
         return new EJBService();
      }else {
         return new JMSService();
      }
   }
}

```

4. 创建业务代表
```java
// BusinessDelegate.java

public class BusinessDelegate {
   private BusinessLookUp lookupService = new BusinessLookUp();
   private BusinessService businessService;
   private String serviceType;

   public void setServiceType(String serviceType){
      this.serviceType = serviceType;
   }

   public void doTask(){
      businessService = lookupService.getBusinessService(serviceType);
      businessService.doProcessing();
   }
}

```

5. 创建客户端
```java
// Client.java

public class Client {

   BusinessDelegate businessService;

   public Client(BusinessDelegate businessService){
      this.businessService  = businessService;
   }

   public void doTask(){
      businessService.doTask();
   }
}

```

6. 使用 BusinessDelegate 和 Client 类来演示业务代表模式
```java
// BusinessDelegatePatternDemo.java

public class BusinessDelegatePatternDemo {

   public static void main(String[] args) {

      BusinessDelegate businessDelegate = new BusinessDelegate();
      businessDelegate.setServiceType("EJB");

      Client client = new Client(businessDelegate);
      client.doTask();

      businessDelegate.setServiceType("JMS");
      client.doTask();
   }
}

```

### 组合实体模式
组合实体模式（Composite Entity Pattern）用在 EJB 持久化机制中。一个组合实体是一个 EJB 实体 bean，代表了对象的图解。当更新一个组合实体时，内部依赖对象 beans 会自动更新，因为它们是由 EJB 实体 bean 管理的。以下是组合实体 bean 的参与者。

- 组合实体（Composite Entity） - 它是主要的实体 bean。它可以是粗粒的，或者可以包含一个粗粒度对象，用于持续生命周期。
- 粗粒度对象（Coarse-Grained Object） - 该对象包含依赖对象。它有自己的生命周期，也能管理依赖对象的生命周期。
- 依赖对象（Dependent Object） - 依赖对象是一个持续生命周期依赖于粗粒度对象的对象。
- 策略（Strategies） - 策略表示如何实现组合实体。

#### 实现

<div align=center><img src='/pic/c&c++/dp-zhst.jpg'/></div>
<center>组合实体模式</center>

#### 例子
1. 创建依赖对象
```java
// DependentObject1.java

public class DependentObject1 {

   private String data;

   public void setData(String data){
      this.data = data;
   }

   public String getData(){
      return data;
   }
}

```

```java
// DependentObject2.java

public class DependentObject2 {

   private String data;

   public void setData(String data){
      this.data = data;
   }

   public String getData(){
      return data;
   }
}

```

2. 创建粗粒度对象
```JAVA
// CoarseGrainedObject.java

public class CoarseGrainedObject {
   DependentObject1 do1 = new DependentObject1();
   DependentObject2 do2 = new DependentObject2();

   public void setData(String data1, String data2){
      do1.setData(data1);
      do2.setData(data2);
   }

   public String[] getData(){
      return new String[] {do1.getData(),do2.getData()};
   }
}

```

3. 创建组合实体
```java
// CompositeEntity.java

public class CompositeEntity {
   private CoarseGrainedObject cgo = new CoarseGrainedObject();

   public void setData(String data1, String data2){
      cgo.setData(data1, data2);
   }

   public String[] getData(){
      return cgo.getData();
   }
}

```

4. 创建使用组合实体的客户端类
```java
// Client.java

public class Client {
   private CompositeEntity compositeEntity = new CompositeEntity();

   public void printData(){
      for (int i = 0; i < compositeEntity.getData().length; i++) {
         System.out.println("Data: " + compositeEntity.getData()[i]);
      }
   }

   public void setData(String data1, String data2){
      compositeEntity.setData(data1, data2);
   }
}

```

5. 使用 Client 来演示组合实体设计模式的用法
```java
// CompositeEntityPatternDemo.java

public class CompositeEntityPatternDemo {
   public static void main(String[] args) {
       Client client = new Client();
       client.setData("Test", "Data");
       client.printData();
       client.setData("Second Test", "Data1");
       client.printData();
   }
}

```

### 数据访问对象模式
数据访问对象模式（Data Access Object Pattern）或 DAO 模式用于把低级的数据访问 API 或操作从高级的业务服务中分离出来。以下是数据访问对象模式的参与者。

- 数据访问对象接口（Data Access Object Interface） - 该接口定义了在一个模型对象上要执行的标准操作。
- 数据访问对象实体类（Data Access Object concrete class） - 该类实现了上述的接口。该类负责从数据源获取数据，数据源可以是数据库，也可以是 xml，或者是其他的存储机制。
- 模型对象/数值对象（Model Object/Value Object） - 该对象是简单的 POJO，包含了 get/set 方法来存储通过使用 DAO 类检索到的数据。

#### 实现
<div align=center><img src='/pic/c&c++/dp-sjfwdx.jpg'/></div>
<center>数据访问对象模式</center>

#### 例子
1. 创建数值对象
```java
// Student.java

public class Student {
   private String name;
   private int rollNo;

   Student(String name, int rollNo){
      this.name = name;
      this.rollNo = rollNo;
   }

   public String getName() {
      return name;
   }

   public void setName(String name) {
      this.name = name;
   }

   public int getRollNo() {
      return rollNo;
   }

   public void setRollNo(int rollNo) {
      this.rollNo = rollNo;
   }
}

```

2. 创建数据访问对象接口
```java
// StudentDao.java

import java.util.List;

public interface StudentDao {
   public List<Student> getAllStudents();
   public Student getStudent(int rollNo);
   public void updateStudent(Student student);
   public void deleteStudent(Student student);
}

```

3. 创建实现了上述接口的实体类
```java
// StudentDaoImpl.java

import java.util.ArrayList;
import java.util.List;

public class StudentDaoImpl implements StudentDao {

   //列表是当作一个数据库
   List<Student> students;

   public StudentDaoImpl(){
      students = new ArrayList<Student>();
      Student student1 = new Student("Robert",0);
      Student student2 = new Student("John",1);
      students.add(student1);
      students.add(student2);
   }
   @Override
   public void deleteStudent(Student student) {
      students.remove(student.getRollNo());
      System.out.println("Student: Roll No " + student.getRollNo()
         +", deleted from database");
   }

   //从数据库中检索学生名单
   @Override
   public List<Student> getAllStudents() {
      return students;
   }

   @Override
   public Student getStudent(int rollNo) {
      return students.get(rollNo);
   }

   @Override
   public void updateStudent(Student student) {
      students.get(student.getRollNo()).setName(student.getName());
      System.out.println("Student: Roll No " + student.getRollNo()
         +", updated in the database");
   }
}

```

4. 使用 StudentDao 来演示数据访问对象模式的用法
```java
// DaoPatternDemo.java

public class DaoPatternDemo {
   public static void main(String[] args) {
      StudentDao studentDao = new StudentDaoImpl();

      //输出所有的学生
      for (Student student : studentDao.getAllStudents()) {
         System.out.println("Student: [RollNo : "
            +student.getRollNo()+", Name : "+student.getName()+" ]");
      }


      //更新学生
      Student student =studentDao.getAllStudents().get(0);
      student.setName("Michael");
      studentDao.updateStudent(student);

      //获取学生
      studentDao.getStudent(0);
      System.out.println("Student: [RollNo : "
         +student.getRollNo()+", Name : "+student.getName()+" ]");
   }
}

```

### 前端控制器模式

前端控制器模式（Front Controller Pattern）是用来提供一个集中的请求处理机制，所有的请求都将由一个单一的处理程序处理。该处理程序可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序。以下是这种设计模式的实体。

- 前端控制器（Front Controller） - 处理应用程序所有类型请求的单个处理程序，应用程序可以是基于 web 的应用程序，也可以是基于桌面的应用程序。
- 调度器（Dispatcher） - 前端控制器可能使用一个调度器对象来调度请求到相应的具体处理程序。
- 视图（View） - 视图是为请求而创建的对象。

#### 实现
<div align=center><img src='/pic/c&c++/dp-qdkz.jpg'/></div>
<center>前端控制器模式</center>

#### 例子
1. 创建视图
```java
// HomeView.java

public class HomeView {
   public void show(){
      System.out.println("Displaying Home Page");
   }
}

```

```java
// StudentView.java

public class StudentView {
   public void show(){
      System.out.println("Displaying Student Page");
   }
}

```

2. 创建调度器 Dispatcher

```java
// Dispatcher.java

public class Dispatcher {
   private StudentView studentView;
   private HomeView homeView;
   public Dispatcher(){
      studentView = new StudentView();
      homeView = new HomeView();
   }

   public void dispatch(String request){
      if(request.equalsIgnoreCase("STUDENT")){
         studentView.show();
      }else{
         homeView.show();
      }
   }
}

```

3. 创建前端控制器 FrontController
```java
// FrontController.java

public class FrontController {

   private Dispatcher dispatcher;

   public FrontController(){
      dispatcher = new Dispatcher();
   }

   private boolean isAuthenticUser(){
      System.out.println("User is authenticated successfully.");
      return true;
   }

   private void trackRequest(String request){
      System.out.println("Page requested: " + request);
   }

   public void dispatchRequest(String request){
      //记录每一个请求
      trackRequest(request);
      //对用户进行身份验证
      if(isAuthenticUser()){
         dispatcher.dispatch(request);
      }
   }
}

```

4. 使用 FrontController 来演示前端控制器设计模式
```java
// FrontControllerPatternDemo.java

public class FrontControllerPatternDemo {
   public static void main(String[] args) {
      FrontController frontController = new FrontController();
      frontController.dispatchRequest("HOME");
      frontController.dispatchRequest("STUDENT");
   }
}

```

### 拦截过滤器模式
拦截过滤器模式（Intercepting Filter Pattern）用于对应用程序的请求或响应做一些预处理/后处理。定义过滤器，并在把请求传给实际目标应用程序之前应用在请求上。过滤器可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序。以下是这种设计模式的实体。

- 过滤器（Filter） - 过滤器在请求处理程序执行请求之前或之后，执行某些任务。
- 过滤器链（Filter Chain） - 过滤器链带有多个过滤器，并在 Target 上按照定义的顺序执行这些过滤器。
- Target - Target 对象是请求处理程序。
- 过滤管理器（Filter Manager） - 过滤管理器管理过滤器和过滤器链。
- 客户端（Client） - Client 是向 Target 对象发送请求的对象

#### 实现

<div align=center><img src='/pic/c&c++/dp-ljglq.jpg'/></div>
<center>拦截过滤器模式</center>

#### 例子
1. 创建过滤器接口 Filter
```java
// Filter.java

public interface Filter {
   public void execute(String request);
}

```

2. 创建实体过滤器
```java
// AuthenticationFilter.java

public class AuthenticationFilter implements Filter {
   public void execute(String request){
      System.out.println("Authenticating request: " + request);
   }
}

```

```java
// DebugFilter.java

public class DebugFilter implements Filter {
   public void execute(String request){
      System.out.println("request log: " + request);
   }
}

```

3. 创建 Target
```java
// Target.java

public class Target {
   public void execute(String request){
      System.out.println("Executing request: " + request);
   }
}

```

4. 创建过滤器链
```java
// FilterChain.java

import java.util.ArrayList;
import java.util.List;

public class FilterChain {
   private List<Filter> filters = new ArrayList<Filter>();
   private Target target;

   public void addFilter(Filter filter){
      filters.add(filter);
   }

   public void execute(String request){
      for (Filter filter : filters) {
         filter.execute(request);
      }
      target.execute(request);
   }

   public void setTarget(Target target){
      this.target = target;
   }
}

```

5. 创建过滤管理器
```java
// FilterManager.java

public class FilterManager {
   FilterChain filterChain;

   public FilterManager(Target target){
      filterChain = new FilterChain();
      filterChain.setTarget(target);
   }
   public void setFilter(Filter filter){
      filterChain.addFilter(filter);
   }

   public void filterRequest(String request){
      filterChain.execute(request);
   }
}

```

6. 创建客户端 Client
```java
// Client.java

public class Client {
   FilterManager filterManager;

   public void setFilterManager(FilterManager filterManager){
      this.filterManager = filterManager;
   }

   public void sendRequest(String request){
      filterManager.filterRequest(request);
   }
}

```

7. 使用 Client 来演示拦截过滤器设计模式
```java
// InterceptingFilterDemo.java

public class InterceptingFilterDemo {
   public static void main(String[] args) {
      FilterManager filterManager = new FilterManager(new Target());
      filterManager.setFilter(new AuthenticationFilter());
      filterManager.setFilter(new DebugFilter());

      Client client = new Client();
      client.setFilterManager(filterManager);
      client.sendRequest("HOME");
   }
}

```

### 服务定位器模式
服务定位器模式（Service Locator Pattern）用在我们想使用 JNDI 查询定位各种服务的时候。考虑到为某个服务查找 JNDI 的代价很高，服务定位器模式充分利用了缓存技术。在首次请求某个服务时，服务定位器在 JNDI 中查找服务，并缓存该服务对象。当再次请求相同的服务时，服务定位器会在它的缓存中查找，这样可以在很大程度上提高应用程序的性能。以下是这种设计模式的实体。

- 服务（Service） - 实际处理请求的服务。对这种服务的引用可以在 JNDI 服务器中查找到。
- Context / 初始的 Context - JNDI Context 带有对要查找的服务的引用。
- 服务定位器（Service Locator） - 服务定位器是通过 JNDI 查找和缓存服务来获取服务的单点接触。
- 缓存（Cache） - 缓存存储服务的引用，以便复用它们。
- 客户端（Client） - Client 是通过 ServiceLocator 调用服务的对象。

#### 实现
<div align=center><img src='/pic/c&c++/dp-fwdw.svg'/></div>
<center>服务定位模式</center>

#### 例子
1. 创建服务接口 Service
```java
// Service.java

public interface Service {
   public String getName();
   public void execute();
}

```

2. 创建实体服务
```java
// Service1.java

public class Service1 implements Service {
   public void execute(){
      System.out.println("Executing Service1");
   }

   @Override
   public String getName() {
      return "Service1";
   }
}

```

```java
// Service2.java

public class Service2 implements Service {
   public void execute(){
      System.out.println("Executing Service2");
   }

   @Override
   public String getName() {
      return "Service2";
   }
}

```

3. 为 JNDI 查询创建 InitialContext
```java
// InitialContext.java

public class InitialContext {
   public Object lookup(String jndiName){
      if(jndiName.equalsIgnoreCase("SERVICE1")){
         System.out.println("Looking up and creating a new Service1 object");
         return new Service1();
      }else if (jndiName.equalsIgnoreCase("SERVICE2")){
         System.out.println("Looking up and creating a new Service2 object");
         return new Service2();
      }
      return null;
   }
}

```

4. 创建缓存 Cache
```java
// Cache.java

import java.util.ArrayList;
import java.util.List;

public class Cache {

   private List<Service> services;

   public Cache(){
      services = new ArrayList<Service>();
   }

   public Service getService(String serviceName){
      for (Service service : services) {
         if(service.getName().equalsIgnoreCase(serviceName)){
            System.out.println("Returning cached  "+serviceName+" object");
            return service;
         }
      }
      return null;
   }

   public void addService(Service newService){
      boolean exists = false;
      for (Service service : services) {
         if(service.getName().equalsIgnoreCase(newService.getName())){
            exists = true;
         }
      }
      if(!exists){
         services.add(newService);
      }
   }
}

```

5. 创建服务定位器
```java
// ServiceLocator.java

public class ServiceLocator {
   private static Cache cache;

   static {
      cache = new Cache();
   }

   public static Service getService(String jndiName){

      Service service = cache.getService(jndiName);

      if(service != null){
         return service;
      }

      InitialContext context = new InitialContext();
      Service service1 = (Service)context.lookup(jndiName);
      cache.addService(service1);
      return service1;
   }
}

```

6. 使用 ServiceLocator 来演示服务定位器设计模式
```java
// ServiceLocatorPatternDemo.java

public class ServiceLocatorPatternDemo {
   public static void main(String[] args) {
      Service service = ServiceLocator.getService("Service1");
      service.execute();
      service = ServiceLocator.getService("Service2");
      service.execute();
      service = ServiceLocator.getService("Service1");
      service.execute();
      service = ServiceLocator.getService("Service2");
      service.execute();
   }
}

```

### 传输对象模式
传输对象模式（Transfer Object Pattern）用于从客户端向服务器一次性传递带有多个属性的数据。传输对象也被称为数值对象。传输对象是一个具有 getter/setter 方法的简单的 POJO 类，它是可序列化的，所以它可以通过网络传输。它没有任何的行为。服务器端的业务类通常从数据库读取数据，然后填充 POJO，并把它发送到客户端或按值传递它。对于客户端，传输对象是只读的。客户端可以创建自己的传输对象，并把它传递给服务器，以便一次性更新数据库中的数值。以下是这种设计模式的实体。

- 业务对象（Business Object） - 为传输对象填充数据的业务服务。
- 传输对象（Transfer Object） - 简单的 POJO，只有设置/获取属性的方法。
- 客户端（Client） - 客户端可以发送请求或者发送传输对象到业务对象。

#### 实现
<div align=center><img src='/pic/c&c++/dp-csdx.svg'/></div>
<center>传输对象模式</center>

#### 例子
1. 创建传输对象
```java
// StudentVO.java

public class StudentVO {
   private String name;
   private int rollNo;

   StudentVO(String name, int rollNo){
      this.name = name;
      this.rollNo = rollNo;
   }

   public String getName() {
      return name;
   }

   public void setName(String name) {
      this.name = name;
   }

   public int getRollNo() {
      return rollNo;
   }

   public void setRollNo(int rollNo) {
      this.rollNo = rollNo;
   }
}

```

2. 创建业务对象
```java
// StudentBO.java

import java.util.ArrayList;
import java.util.List;

public class StudentBO {

   //列表是当作一个数据库
   List<StudentVO> students;

   public StudentBO(){
      students = new ArrayList<StudentVO>();
      StudentVO student1 = new StudentVO("Robert",0);
      StudentVO student2 = new StudentVO("John",1);
      students.add(student1);
      students.add(student2);
   }
   public void deleteStudent(StudentVO student) {
      students.remove(student.getRollNo());
      System.out.println("Student: Roll No "
      + student.getRollNo() +", deleted from database");
   }

   //从数据库中检索学生名单
   public List<StudentVO> getAllStudents() {
      return students;
   }

   public StudentVO getStudent(int rollNo) {
      return students.get(rollNo);
   }

   public void updateStudent(StudentVO student) {
      students.get(student.getRollNo()).setName(student.getName());
      System.out.println("Student: Roll No "
      + student.getRollNo() +", updated in the database");
   }
}

```

3. 使用 StudentBO 来演示传输对象设计模式
```java
// TransferObjectPatternDemo.java

public class TransferObjectPatternDemo {
   public static void main(String[] args) {
      StudentBO studentBusinessObject = new StudentBO();

      //输出所有的学生
      for (StudentVO student : studentBusinessObject.getAllStudents()) {
         System.out.println("Student: [RollNo : "
         +student.getRollNo()+", Name : "+student.getName()+" ]");
      }

      //更新学生
      StudentVO student =studentBusinessObject.getAllStudents().get(0);
      student.setName("Michael");
      studentBusinessObject.updateStudent(student);

      //获取学生
      studentBusinessObject.getStudent(0);
      System.out.println("Student: [RollNo : "
      +student.getRollNo()+", Name : "+student.getName()+" ]");
   }
}

```





