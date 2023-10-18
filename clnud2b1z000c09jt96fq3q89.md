---
title: "设计模式 一"
datePublished: Tue Oct 17 2023 13:29:27 GMT+0000 (Coordinated Universal Time)
cuid: clnud2b1z000c09jt96fq3q89
slug: 6k66k6h5qih5bypios4ga
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/hGV2TfOh0ns/upload/c6c9eca1ac4e2d3d9b024a2a729703a3.png
tags: design-patterns

---

模式不是代码，而是一种通用问题的通用解法。

设计模式分为 3 大类型

1. 创建型模式：单例模式、抽象工厂模式、原型模式、建造者模式、工厂模式。
    
2. 结构型模式：适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式。
    
3. 行为型模式：模版方法模式、命令模式、访问者模式、迭代器模式、观察者模式、中介者模式、备忘录模式、 解释器模式(Interpreter模式)、状态模式、策略模式、职责链模式（责任链模式）。
    

# 单例模式

所谓类的单例设计模式，就是**采取一定的方法保证在整个的软件系统中，对某个类只能存在一个对象实例**， 并且该类只提供一个取得其对象实例的方法（静态方法）。

## 单例设计模式的八种方式

1）**饿汉式（静态常量）**

1. **饿汉式（静态代码块）**
    
2. 懒汉式（线程不安全）
    
3. 懒汉式（线程安全，同步方法）
    
4. 懒汉式（线程安全，同步代码块）
    
5. **双重检查**
    
6. **静态内部类**
    
7. **枚举**
    

**使用场景**

1. 单例模式保证了系统内存中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使 用单例模式可以提高系统性能
    
2. 当想实例化一个单例类的时候，必须要记住使用相应的获取对象的方法，而不是使用
    
3. 单例模式使用的场景：需要频繁的进行创建和销毁的对象、创建对象时耗时过多或耗费资源过多（即：**重量级对象**)，但又经常用到的对象、**工具类对象**、频繁访问数据库或文件的对象（比如**数据源**、**session工厂**等）
    

### 饿汉式（静态常量）

有下面几点特点

1）私有化构造器

2）类的内部创建对象

3）向外暴露一个静态的公共方法 getInstance()

```java
/**
 * @author leikooo @Description 单例模式
 */
public class SingletonTest01 {
  public static void main(String[] args) {
    Singleton instance1 = Singleton.getInstance();
    Singleton instance2 = Singleton.getInstance();
    System.out.println(instance1 == instance2);
    System.out.println(instance1.hashCode());
    System.out.println(instance2.hashCode());
  }
}

// 饿汉式（静态成员变量）
class Singleton {

  // 私有化构造器
  private Singleton() {}

  // 本类内部创建对象实例
  public static final Singleton INSTANCE = new Singleton();

  public static Singleton getInstance() {
    return INSTANCE;
  }
}
```

`优点`：这种写法比较简单，就是在类装载的时候就完成实例化。避免了线程同步问题。

`缺点`：在类装载的时候就完成实例化，没有达到 Lazy Loading 的效果。如果从始至终从未使用过这个实例，则 会造成内存的浪费。这种方式基于 classloder 机制避免了多线程的同步问题，不过，instance在类装载时就实例化，在单例模式中大 多数都是调用 getInstance方法，但是导致类装载的原因有很多种，因此不能确定有其他的方式（或者其他的静态方法)导致类装载，这时候初始化 instance 就没有达到 lazy loading 的效果

> 总结：可以用但是可能会造成资源浪费

### **饿汉式（静态代码块）**

```java
class Singleton {

  // 私有化构造器
  private Singleton() {}

  // 本类内部创建对象实例
  public static final Singleton INSTANCE;

  static {
    INSTANCE = new Singleton();
  }

  public static Singleton getInstance() {
    return INSTANCE;
  }
}
```

> 总结：可以用但是可能会造成资源浪费

### 懒汉式（线程不安全）

```java
public class SingletonTest03 {
  public static void main(String[] args) {
    Singleton instance1 = Singleton.getInstance();
    Singleton instance2 = Singleton.getInstance();
    System.out.println(instance1 == instance2);
    System.out.println(instance1.hashCode());
    System.out.println(instance2.hashCode());
  }
}


class Singleton {
  // 私有化构造器, 防止 new
  private Singleton() {}

  private static Singleton INSTANCE;

  // 用到的时候才去 new
  public static Singleton getInstance() {
    if (INSTANCE == null) {
      INSTANCE = new Singleton();
    }
    return INSTANCE;
  }
}
```

1. 起到了 `Lazy Loading` 的效果，但是只能在单线程下使用。
    
2. 如果在多线程下，一个线程进入了 if(singleton = null) 判断语句块，还未来得及往下执行，另一个线程也通过 了这个判断语句，这时便会产生多个实例。所以在多线程环境下不可使用这种方式 。这种经典的情况，下次需要敏感一些了
    

> 结论：**在实际开发中，不要使用这种方式**

### 懒汉式（线程安全，同步方法）

```java
class Singleton {
  // 私有化构造器, 防止 new
  private Singleton() {}

  private static Singleton INSTANCE;

  // 用到的时候才去 new
  public static synchronized Singleton getInstance() {
    if (INSTANCE == null) {
      INSTANCE = new Singleton();
    }
    return INSTANCE;
  }
}
```

1. 解决了线程安全问题
    
2. 效率**太低**了，每个线程在想获得类的实例时候，执行getInstance()方法都要进行同步。而其实这个方法只执行 一次实例化代码就够了，后面的想获得该类实例，直接reum就行了。方法进行同步效率太低
    

> 结论：在实际开发中，不推荐使用这种方式

### 懒汉式（线程安全，同步代码块）

```java
class Singleton {
  // 私有化构造器, 防止 new
  private Singleton() {}

  private static Singleton INSTANCE;

  // 当用户到的时候才去 new
  public static Singleton getInstance() {
    if (INSTANCE == null) {
      synchronized (Singleton.class) {
        INSTANCE = new Singleton();
      }
    }
    return INSTANCE;
  }
}
```

简直就灾难，这样写不仅解决不了线程安全，也解决不了单例

### **双重检查**

在Java中，`volatile` 是一个关键字，它用于声明变量。`volatile` 主要具有以下两个作用：

1. **可见性**：`volatile` 保证了一个线程对被声明为 `volatile` 的变量的修改对其他线程是可见的。这意味着当一个线程修改了一个 `volatile` 变量的值时，其他线程可以立即看到这个值的变化。这有助于**避免了线程之间的缓存不一致性问题**。
    
2. **禁止重排序**：`volatile` 还禁止了编译器和运行时系统对被声明为 `volatile` 的变量进行重排序。这可以确保变量的读取和写入操作按照代码的顺序执行，不会因为编译器或处理器的优化而改变执行顺序。
    

```java
class Singleton {
  // 私有化构造器, 防止 new
  private Singleton() {}

  // volatile 确保单例对象的初始化在多线程环境下是正确的
  // 可以理解为 轻量级 synchronized 锁🔒
  private static volatile Singleton INSTANCE;

  // 当用户到的时候才去 new
  public static Singleton getInstance() {
    // 5、C 线程这里就进入不进去
    if (INSTANCE == null) {
      // 1、假设现在有两个线程 A、B 同时在这里
      // 2、A 先进入下面的 synchronized, 此时 A、B 两个线程不可能同时进入这个线程之中
      synchronized (Singleton.class) {
        // 4、B 进入到这里之中, 由于 INSTANCE 加上了 volatile 注解, A 修改之后会立刻更新内存中的 INSTANCE 
        // 所以也进不去下面的 if 循环之中
        if (INSTANCE == null) {
          // 3、A 开始创建对象
          INSTANCE = new Singleton();
        }
      }
    }
    return INSTANCE;
  }
}
```

1）Double-Check概念是多线程开发中常使用到的，如代码中所示，我们进行了两次if(singleton=null)检查，这 样就可以保证线程安全了。

2）这样，实例化代码只用执行一次，后面再次访问时，判断if(singleton=null),直接return实例化对象，也避 免的反复进行方法同步

3）**线程安全：延迟加载：效率较高**

> 结论：在实际开发中，**推荐使用这种单例设计模式**

### **静态内部类**

```java
class Singleton {
  // 私有化构造器, 防止 new
  private Singleton() {}

  private static volatile Singleton instance;

  // 写一个静态内部类，该类中有一个静态属性 Singleton
  // 使用了 JVM 底层对应的线程安全
  private static class SingletonInstance {
    private static final Singleton INSTANCE = new Singleton();
  }

  // 提供一个静态的公有方法，直接返回SingletonInstance.INSTANCE
  public static synchronized Singleton getInstance() {
    return SingletonInstance.INSTANCE;
  }
}
```

1. 静态内部类方式在 Singleton 类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance 方法，才会装载 SingletonInstance 类，从而完成 Singleton 的实例化。
    
2. 类的静态属性只会在第一次加载类的时候初始化，所以在这里，**JVM 帮助我们保证了线程的安全性**，在类进行初始化时，别的线程是无法进入的。
    
3. 优点：避免了线程不安全，利用静态内部类特点实现延迟加载，效率高
    

> 结论：推荐使用

### **枚举**

```java
enum Singleton {
  INSTANCE;

  public void sayOk() {
    System.out.println("ok~~");
  }
}
```

1. 这借助 JDK1.5 中添加的枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建 新的对象。
    
2. 这种方式是 Effective Java 作者 Josh Bloch 提倡的方式
    

> 结论：推荐使用

# 抽象工厂模式

## 简单工厂

看一个披萨的项目：要便于披萨种类的扩展，要便于维护

1. 披萨的种类很多（比如GreekPizz、CheesePizz等）
    
2. 披萨的制作有prepare,bake,cut,box
    
3. 完成披萨店订购功能。
    

```java
public class OrderPizza {
  public OrderPizza() {
    Pizza pizza = null;
    do {
      String pizzaType = getType();
      if ("GeekPizza".equals(pizzaType)) {
        pizza = new GeekPizza();
      } else if ("CheesePizza".equals(pizzaType)) {
        pizza = new CheesePizza();
      } else {
        break;
      }
      pizza.setName(pizzaType);
      pizza.prepare();
      pizza.cut();
      pizza.bake();
      pizza.box();
    } while (true);
  }
  public String getType() {
    try{
      BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
      System.out.println("input pizza species: ");
      return bufferedReader.readLine();
    } catch (IOException e) {
      e.printStackTrace();
      return "";
    }
  }
}
```

这段代码是可以最先想到的，但是我们如果想要新增加一些一个新的类，我们就需要在OrderPizza 类修改很大，这违反了 `开闭原则` 所以不可取，而且类图很复杂😢

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697263978962/043e2ff4-2fdd-482e-a9b4-73d3f1858a5f.png align="center")

一个比较简单的解决方法就是引入 一个简单工厂

1. 简单工厂模式是属于创建型模式，是工厂模式的一种。简单工厂模式是由一个工厂对象决定创建出哪一种产品 类的实例。简单工厂模式是工厂模式家族中**最简单实用的模式**
    
2. 简单工厂模式：定义了一个创建对象的类，由这个类来封装实例化对象的行为（代码）
    
3. 在软件开发中，当我们会用到大量的创建某种、某类或者某批对象时，就会使用到工厂模式.
    

```java
/**
 * @author leikooo @Description 简单工厂
 */
public class OrderPizza {

  // 定义一个建档工厂对象， 聚合关系
  SimpleFactory simpleFactory;

  public OrderPizza(SimpleFactory simpleFactory) {
    this.setSimpleFactory(simpleFactory);
    do {
      String pizzaType = getType();
      Pizza pizza = simpleFactory.creatPizza(pizzaType);
      if (pizza != null) {
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
      } else {
        System.out.println("订购 pizza 失败");
        return;
      }
    } while (true);
  }


  public String getType() {
    try{
      BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
      System.out.println("input pizza species: ");
      return bufferedReader.readLine();
    } catch (IOException e) {
      e.printStackTrace();
      return "";
    }
  }

  public void setSimpleFactory(SimpleFactory simpleFactory) {
    this.simpleFactory = simpleFactory;
  }
}
```

工厂类

```java
/**
 * @author leikooo @Description
 */
public class SimpleFactory {
  // 通过 orderType 返回 Pizza 对象
  public Pizza creatPizza(String pizzaType) {
    System.out.println("使用 simple factory");
    Pizza pizza = null;
    if ("GeekPizza".equals(pizzaType)) {
      pizza = new GeekPizza();
      pizza.setName("GeekPizza");
    } else if ("CheesePizza".equals(pizzaType)) {
      pizza = new CheesePizza();
      pizza.setName("CheesePizza");
    } else if ("PepperPizza".equals(pizzaType)) {
      pizza = new PepperPizza();
      pizza.setName("PepperPizza");
    }
    return pizza;
  }
}
```

调用代码

```java
public class PizzaStore {
  public static void main(String[] args) {
    new OrderPizza(new SimpleFactory());
    System.out.println("退出程序~~~");
  }
}
```

使用了简单工厂之后的代码就变成了这样的 UML ，非常清爽

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697263164977/8f9c452c-296e-4505-9256-2e26108cb5e0.png align="center")

简单工厂也叫做**静态工厂模式**

```java
/**
 * @author leikooo @Description
 */
public class OrderPizza2 {

  // 定义一个建档工厂对象， 聚合关系
  SimpleFactory simpleFactory;

  public OrderPizza2() {
    do {
      String pizzaType = getType();
      Pizza pizza = SimpleFactory.creatPizza(pizzaType);
      if (pizza != null) {
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
      } else {
        System.out.println("订购 pizza 失败");
        return;
      }
    } while (true);
  }

  public String getType() {
    try {
      BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
      System.out.println("input pizza species: ");
      return bufferedReader.readLine();
    } catch (IOException e) {
      e.printStackTrace();
      return "";
    }
  }
}
```

工厂类只需要修改对应的创建方式变成静态即可

```java
/**
 * @author leikooo @Description 静态工厂类
 */
public class SimpleFactory {
  // 通过 orderType 返回 Pizza 对象
  public static Pizza creatPizza(String pizzaType) {
    System.out.println("使用 simple factory");
    Pizza pizza = null;
    if ("GeekPizza".equals(pizzaType)) {
      pizza = new GeekPizza();
      pizza.setName("GeekPizza");
    } else if ("CheesePizza".equals(pizzaType)) {
      pizza = new CheesePizza();
      pizza.setName("CheesePizza");
    } else if ("PepperPizza".equals(pizzaType)) {
      pizza = new PepperPizza();
      pizza.setName("PepperPizza");
    }
    return pizza;
  }
}
```

## 工厂方法模式

披萨项目新的需求：客户在点披萨时，可以点不同口味的披萨，比如北京的奶酪 pizza、北京的胡椒pizza或 者是伦敦的奶酪 pizza、伦敦的胡椒 pizza。

我们不能创建 4 个工厂去创建相应的对象，所以我们需要这个 「工厂方法模式」

1. 工厂方法模式设计方案：将披萨项目的实例化功能抽象成抽象方法，在不同的口味点餐子类中具体实现。
    
2. 工厂方法模式：**定义了一个创建对象的抽象方法，由子类决定要实例化的类**。工厂方法模式将对象的实例化推迟到子类。
    

*定义一个抽象方法，让其子类实现具体代码*

```java
/**
 * @author leikooo @Description
 */
public abstract class OrderPizza {
  /**
   * 定义一个抽象方法，让其子类实现具体代码
   *
   * @param pizzaType pizza 的类型
   * @return
   */
  public abstract Pizza creatPizza(String pizzaType);

  // 构造器
  public OrderPizza() {
    Pizza pizza = null;
    String pizzaType = "";
    do {
      pizzaType = getType();
      pizza = creatPizza(pizzaType);
      if (pizza == null) {
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
      } else {
        return;
      }
    } while (true);
  }

  // 从键盘上获取对应的类型
  public String getType() {
    try {
      BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
      System.out.println("input pizza species: ");
      return bufferedReader.readLine();
    } catch (IOException e) {
      e.printStackTrace();
      return "";
    }
  }
}
```

```java
/**
 * @author leikooo 
 */
public class BJOrderPizza extends OrderPizza{

  @Override
  public Pizza creatPizza(String pizzaType) {
    Pizza pizza = null;
    if ("geek".equals(pizzaType)) {
      pizza = new BJGeekPizza();
    } else {
      pizza = new BJCheesePizza();
    }
    return pizza;
  }
}
```

测试方法

```java
public class PizzaStore {
  public static void main(String[] args) {
    // 先调用父类构造器, 再创建子类构造器
    OrderPizza orderPizza = new BJOrderPizza();
    System.out.println("退出程序~~~");
  }
}
```

`OrderPizza orderPizza = new BJOrderPizza()` 执行过程先调用父类的构造器，之后调用子类的构造器 [https://www.itwanger.com/java/2019/11/06/java-duotai.html](https://www.itwanger.com/java/2019/11/06/java-duotai.html)

## 抽象工厂模式

1. 抽象工厂模式：定义了一个interface用于创建相关或有依赖关系的对象簇，而无需指明具体的类
    
2. 抽象工厂模式可以将简单工厂模式和工厂方法模式进行整合。
    
3. 从设计层面看，抽象工厂模式就是对简单工厂模式的改进（或者称为进一步的抽象）。
    
4. 将工厂抽象成两层，AbsFactory(抽象工厂)和具体实现的工厂子类。程序员可以根据创建对象类型使用对应 的工厂子类。这样将单个的简单工厂类变成了工厂簇，更利于代码的维护和扩展。
    

类图

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697462137183/c7209e58-a3b3-497a-9423-2e7dadee5382.png align="center")

```java
/**
 * @author leikooo @Description 抽象工厂模式的抽象层
 */
public interface AbsFactory {
  /** 创建 pizza */
  Pizza creatPizza(String orderType);
}
```

```java
/**
 * @author leikooo @Description
 */
public class BJFactory implements AbsFactory {
  @Override
  public Pizza creatPizza(String orderType) {
    System.out.println("~ use BJFactory ~");
    Pizza pizza = null;
    if ("chess".equals(orderType)) {
      pizza = new BJCheesePizza();
    } else if("geek".equals(orderType)){
      pizza = new BJGeekPizza();
    }
    return pizza;
  }
}
```

```java
/**
 * @author leikooo @Description
 */
public class OrderPizza {
  AbsFactory factory;

  public OrderPizza(AbsFactory factory) {
    setFactory(factory);
  }

  private void setFactory(AbsFactory factory) {
    Pizza pizza = null;
    String orderType = "";
    this.factory = factory;
    do {
      orderType = getType();
      pizza = factory.creatPizza(orderType);
      if (pizza != null) {
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
      } else {
        System.out.println("订购失败");
        break;
      }
    } while (true);
  }

  // 从键盘上获取对应的类型
  public String getType() {
    try {
      BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
      System.out.println("input pizza species: ");
      return bufferedReader.readLine();
    } catch (IOException e) {
      e.printStackTrace();
      return "";
    }
  }
}
```

调用方法

```java
public class PizzaStore {
  public static void main(String[] args) {
    // 先调用父类构造器, 再创建子类构造器
    new OrderPizza(new BJFactory());
    System.out.println("退出程序~~~");
  }
}
```

# 工厂模式小结

1. 工厂模式的意义 将实例化对象的代码提取出来，放到一个类中统一管理和维护，达到和主项目的依赖关系的解耦。从而提高项 目的扩展和维护性。
    
2. 三种工厂模式（简单工厂模式、工厂方法模式、抽象工厂模式）
    
3. 设计模式的`依赖抽象原则：`
    
4. 创建对象实例时，不要直接new类，而是把这个new类的动作放在一个工厂的方法中，并返回。有的书上说， 变量不要直接持有具体类的引用。
    
5. 不要让类继承具体类，而是继承抽象类或者是实现 interface (接口)
    
6. 不要覆盖基类中己经实现的方法。
    

# 原型模式

现在有一只羊tom, 姓名为：tom, 年龄为：1，颜色为：白色，请编写程序创建和 tom 羊属性完全相同的10 只羊。

传统方式：

```java
public class Client {
  public static void main(String[] args) {
    // 传统方法
    Sheep sheep = new Sheep("tom", 10, "red");
    Sheep sheep1 = new Sheep(sheep.getName(), sheep.getAge(), sheep.getColor());
    Sheep sheep2 = new Sheep(sheep.getName(), sheep.getAge(), sheep.getColor());
    Sheep sheep3 = new Sheep(sheep.getName(), sheep.getAge(), sheep.getColor());
    Sheep sheep4 = new Sheep(sheep.getName(), sheep.getAge(), sheep.getColor());
    Sheep sheep5 = new Sheep(sheep.getName(), sheep.getAge(), sheep.getColor());
    // ...

  }
}
```

**改进思路**：Java 中 Object 类是所有类的根类，Object 类提供了一个 clone (方法，该方法可以将一个 Java 对象复制 一份，但是需要实现 clone 的 Java 类必须要实现一个接口`Cloneable` ,该接口表示该类能够复制且具有复制的能力 =&gt; `原型模式`

1. 原型模式(Prototype模式)是指：用原型实例指定创建对象的种类，并且通过拷贝这些原型，创建新的对象
    
2. 原型模式是一种创建型设计模式，允许一个对象再创建另外一个可定制的对象，无需知道如何创建的细节
    
3. 工作原理是：通过将一个原型对象传给那个要发动创建的对象，这个要发动创建的对象通过请求原型对象拷贝它 们自己来实施创建，即对象.clone(）
    
4. 形象的理解：孙大圣拔出猴毛，变出其它孙大圣
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697523729086/89f7f833-e149-495a-ba8d-13ef307dc740.png align="center")

需要实现 Cloneable 接口

```java
/**
 * @author leikooo @Description 原型模式
 */
public class Sheep implements Cloneable {
  private String name;
  private int age;
  private String color;

  public Sheep(String name, int age, String color) {
    this.name = name;
    this.age = age;
    this.color = color;
  }

  @Override
  protected Object clone() {
    Sheep sheep = null;
    try {
      sheep = (Sheep) super.clone();
    } catch (CloneNotSupportedException e) {
      throw new RuntimeException(e);
    }
    return sheep;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }

  public String getColor() {
    return color;
  }

  public void setColor(String color) {
    this.color = color;
  }
}
```

调用代码

```java
/**
 * @author leikooo @Description 创建 10 个一样的🐏对象
 */
public class Client {
  public static void main(String[] args) {
    // 传统方法
    Sheep sheep = new Sheep("tom", 10, "red");
    System.out.println("原型模式实现克隆~");

    Sheep clone1 = (Sheep) sheep.clone();
    Sheep clone2 = (Sheep) sheep.clone();
    Sheep clone3 = (Sheep) sheep.clone();

    System.out.println("clone1 = " + clone1);
    System.out.println("clone2 = " + clone2);
    System.out.println("clone3 = " + clone3);
    System.out.println("sheep = " + sheep);
  }
}
```

## 深拷贝和浅拷贝

### 浅拷贝

1. 对于数据类型是**基本数据**类型的成员变量，浅拷贝会直接进行**值传递**，也就是将**该属性值复制一份给新的对象**。
    
2. 对于数据类型是引用数据类型的成员变量，比如说成员变量是某个数组、某个类的对象等，那么浅拷贝会进行 引用传递，也就是只是将该成员变量的引用值（内存地址）复制一份给新的对象。因为实际上两个对象的该成 员变量都指向**同一个实例**。**在这种情沉下，在一个对象中修改该成员变量会影响到另一个对象的该成员变量值**
    
3. 浅拷贝是使用**默认的 clone() 方法**来实现
    

### 深拷贝

1. 复制对象的所有基本数据类型的成员变量值
    
2. 为所有引用数据类型的成员变量申请存储空间，并复制每个引用数据类型成员变量所引用的对象，直到该对象 可达的所有对象。也就是说，对象进行深拷贝要对整个对象（包括对象的引用类型）进行拷贝
    
3. 深拷贝实现方式
    
4. 重写clone方法来实现深拷贝
    
5. 通过对象序列化实现深拷贝\*\*（推荐）\*\*
    

方式一

还需要对引用数据类型的参数进行单独拷贝，不推荐

```java
public class DeepProtoType implements Serializable, Cloneable {

  public String name; 
  public DeepCloneableTarget deepCloneableTarget; 

  public DeepProtoType() {
    super();
  }

  // 重写 clone 方法
  @Override
  protected Object clone() throws CloneNotSupportedException {

    Object deep = null;
    // 这里完成对基本数据类型的克隆
    deep = super.clone();
    // 对引用类型的属性进行单独处理
    DeepProtoType deepProtoType = (DeepProtoType) deep;
    deepProtoType.deepCloneableTarget = (DeepCloneableTarget) deepCloneableTarget.clone();
    return deepProtoType;
  }
}
```

方式二

```java
public class DeepProtoType implements Serializable {

  public String name; 
  public DeepCloneableTarget deepCloneableTarget; 

  public DeepProtoType() {
    super();
  }

  public Object deepClone() {

    ByteArrayOutputStream bos = null;
    ObjectOutputStream oos = null;
    ByteArrayInputStream bis = null;
    ObjectInputStream ois = null;

    try {

      bos = new ByteArrayOutputStream();
      oos = new ObjectOutputStream(bos);
      oos.writeObject(this);

      bis = new ByteArrayInputStream(bos.toByteArray());
      ois = new ObjectInputStream(bis);
      DeepProtoType copyObj = (DeepProtoType) ois.readObject();

      return copyObj;

    } catch (Exception e) {
      // TODO: handle exception
      e.printStackTrace();
      return null;
    } finally {
      try {
        bos.close();
        oos.close();
        bis.close();
        ois.close();
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
  }
}
```

1. 创建新的对象比较复杂时，可以利用原型模式简化对象的创建过程，同时也能够提高效率
    
2. 不用重新初始化对象，而是**动态地获得对象运行时的状态**
    
3. 如果原始对象发生变化（增加或者减少属性），其它克隆对象的也会发生相应的变化，无需修改代码
    
4. 在实现深克隆的时候可能需要比较复杂的代码
    
5. **缺点**：需要为每一个类配备一个克隆方法，这对全新的类来说不是很难，但对已有的类进行改造时，需要修改其源代码，违背了ocp 原则。