---
title: "Uml 类图"
datePublished: Thu Oct 12 2023 12:29:49 GMT+0000 (Coordinated Universal Time)
cuid: clnn5qctq000i09mgcr4h9nl0
slug: uml
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/hGV2TfOh0ns/upload/c6c9eca1ac4e2d3d9b024a2a729703a3.png
tags: uml

---

用于描述系统中的类（对象）本身的组成和类（对象）之间的各种`静态关系`。

类之间的关系：依赖、泛化（继承）、实现、关联、聚合与组合。

# 依赖

只要是在**类中用到了对方，那么他们之间就存在依赖关系**。如果没有对方，连编绎都通过不了。

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697096542463/c9a05ad7-476e-47ef-8007-9f8f4415e78f.png align="center")

依赖关系包括一下几种形式

1. 类中用到了对方
    
2. 如果是类的成员属性
    
3. 如果是方法的返回类型
    
4. 是方法接收的参数类型
    
5. 方法中使用到
    

```java
public class PersonServiceBean {
    // 类
    private PersonDao personDao;
    
    // 构造器
    public PersonServiceBean (Person person) {
        // .....
    }    

    // 方法形参
    public void savePerson(Person person) {

    }
    // 方法返回值类型
    public IDCard getIdCard(Integer idCard) {
        return null;
    }
    // 方法里面的变量
    // ps 虽然违反了迪米特 法则, 但是这的确是一种表现形式
    public void modify() {
        Department department = new Department();
    }
}

class Person{


}

class IDCard {


}

class PersonDao {

}

class Department {


}
```

# 泛化

**泛化关系**实际上就是**继承关系**，TA 是依赖关系的特例

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697096508996/6c149a25-e6de-4c3a-aedc-8ad20a997a87.png align="center")

# 实现

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697096801787/cb7a1544-9b8f-4c2a-8d45-ad1046072a44.png align="center")

如何记住箭头的方向？

> 对于这个来说：是 MyThread 实现了 Runable 所以 箭头指向 Runable

# 关联关系

类与类之间的关系，TA 是依赖关系的特例

关联有导航性、多重性

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697097029315/317a83a7-1109-4710-ab7e-507c633c72a2.png align="center")

# 聚合关系

整体和部分的关系，**整体和部分可以分开**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697097793867/a6ce139a-c911-49c5-a8d7-a4991e409430.png align="center")

# 组合关系

整体和部分的关系，比聚合关系更加密不可分，**整体和部分不能分开**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697097780118/8a7de491-6c61-4b0e-b9d3-2dd73275c3df.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697113759483/42bb531b-ea31-4c92-a17e-5487d4e65cfe.png align="center")

```java
public class PersonServiceBean {
    // 这种就是不可以分离的，是组合关系
    private PersonDao personDao = new PersonDao();
    // 这个就是可以分离的属于是，聚合关系
    private IDCard cardId;
    
    public void setIDCard(IDCard idCard) {
        cardId = idCard;
    }
}
```