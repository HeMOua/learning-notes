# DO、PO、VO、DTO的区别

## POJO

POJO（Plain Ordinary Java Object），即简单Java对象，就是一个我们最常见的普通Java对象，这个概念是被大家叫出来的，它具有

+ 一些属性
+ 对应的getter和setter

示例代码如下：

```java
public class Foo {
    private String name;
    private String type;
    public String getName(){
        return this.name;
    }
    public void setName(String name){
        this.name = name;
    }
    public String getType(){
        return this.Type;
    }
    public void setType(String type){
        this.type = type;
    }
}
```

## JavaBean

JavaBean，可序列化的POJO，sun在早期有对其规范（[JavaBeans Spec](https://www.oracle.com/technetwork/java/javase/documentation/spec-136004.html)），它是Java中的可重用组件，主要规范约定如下：

- JavaBean为公共类，并且具有一个**空构造函数**
- 所有属性为私有属性，提供getter和setter，不应该有公共属性
- 实现序列化接口：`java.io.Serializable`

JavaBean已经成为Java的一种规范，也是Java社区的共同语言，许多工具框架也是遵循JavaBean的规范的，例如，Spring的BeanUtils，一些Json工具都是基于JavaBean的规范来实现的，这些都是基于约定，所以也有人把JavaBean叫为可以持久化的POJO。

## DO

DO（Domain Object），领域对象，也就是ORM框架中对应数据库的对象，业务实体，例如，对现实世界中的用户建模，抽象出来的DO可以叫为UserDO，通常情况下它用于与数据库的数据交互，通常也是一个JavaBean。

## PO

PO（Persistent Object），持久化对象，主要用于持久化层，与数据库对应，通常也是ORM框架中的实体对象，例如，使用JPA时候的Entity与数据库表做映射，通常是一个JavaBean。

## DTO

DTO（Data Transfer Object），数据传输对象，顾名思义就是用于传输数据的对象，通常用于处于不同架构层次或者不同子系统之间的数据传递，或者用于外部接口参数传递，以便提供不同粒度不同信息的数据，以免造成困惑干扰，通常也是一个JavaBean。

## VO

VO（Value Object），就是用于保存数据的对象；在提供给页面使用的时候，也有人解释为View Object，就是对应页面展示数据的对象。

## DAO

DAO（Data Access Object），数据访问对象，与数据库做交互的对象，提供不同的接口访问数据库来实现对数据库的操作，而接口使用的数据交互通常就是PO或者DO，通过它可以使用面向对象的方式来与数据库交互。

## 总结

1. DO、PO、VO、DTO等其实都是一个JavaBean，只是应用的范围不同，表示的意义不同，而这些不同是Java这么多年来形成的一种约定，这种习惯形成Java程序员之间一种共识
2. DO和PO大部分时间是一样的，不过PO更倾向于有状态的对象，例如，使用Hibernate的时候，通过DAO操作过的对象会被框架所持有，如果一旦对其操作，在Session关闭的时候会写回数据库，这个时候使用PO就要非常小心，特别是与其他业务层交互的时候，最好转成DTO或者VO提供数据，不能把持久化对象暴露出去，不然可能会导致数据被修改
3. 在使用这类对象的时候，我们还是要明白其意义，然后在合适的场景使用，毕竟这是Java形成的通用语言，想要读懂他人代码就要理解，想要他人能读懂自己的代码就要遵循