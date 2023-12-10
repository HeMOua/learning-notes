# Java8 新特性

## Lambda

**一、Lambda表达式的基础语法**：

Java8中引入了一个新的操作符"->"该操作符称为箭头操作符或Lambda操作符，箭头操作符将Lambda表达式拆分成两部分：

+ 左侧：Lambda表达式的参数列表
+ 右侧：Lambda表达式中所需执行的功能，即Lambda体

1. 语法格式一：无参数，无返回值
   `() -> System.out.println("Hello Lambda！");`

2. 语法格式二：有一个参数，并且无返回值

   ```java
   Consumer<String> c = x -> System.out.println(x);
   c.accept("the world is so big !");
   ```

3. 语法格式三：若只有一个参数，小括号可以省略不写

4. 语法格式四：有两个以上的参数，有返回值，并且Lambda体中有多条语句

   ```java
   Comparator<Integer> com = (x, y) -> {
       System.out.println("function interface");
       return Integer.compare(x, y);
   }
   ```

5. 语法格式五：若Lambda体中只有一条语句，`return`和大括号都可以省略不写

   ```java
   Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
   ```

6. 语法格式六：Lambdar表达式的参数列表的数据类型可以省略不写，因为JVM编译器通过上下文推断出，数据类型，即"类型推断"

**二、Lambda表达或需要"函数式接口”的支持**

函数式接口：接口中只有一个抽象方法的接口，称为函数式接口。

**三、方法引用与构造器应用**

**方法引用**：若Lambda体中的内容有方法已经实现了，我们可以使用"方法引用"（可以理解为方法引用是Lambda表达式的另外一种表现形式）

主要有三种语法格式：

+ 对象::实例方法名
+ 类::静态方法名
+ 类::实例方法名

```java
// 第一种方式
Consumer<String> con = x -> System.out.println(x);

// 第二种方式
PrintStream ps = System.out;
Consumer<String> con1 = ps::println;
```

上例中 `println()`方法接受一个参数，返回值为 `void`。而`Consumer`函数式接口和 `println()`的参数返回一致，所以可以写成第二种方式

还需要注意的是 `类::实例方法名`

```java
BiPredicate<String, String> bp = (x, y) -> x.equals(y);

BiPredicate<String, String> bp = String::equals;
```

若Lambda参数列表中的第一参数是实例方法的调用者，而第二个参数是实例方法的参数时，可以使用`ClassName::method`

**构造器应用**

格式：`ClassName::new`

```java
Supplier<Employee> sup = () -> new Employee();

Supplier<Employee> sup2 = Employee::new;
```

注意：需要调用的构造器的参数列表要与函数式接口中抽象方法的参数列表保持一致！

**数组引用**

```
Function<Integer, String[]> fun = x -> new String[x];
Stirng[] strs = fun.apply(10);

Function<Integer, String[]> fun2 = String[]::new;
Stirng[] strs2 = fun.apply(10);
```

## 四大函数式接口

```java
Consumer<T> //消费性接口
	void accept(T t);
	
Supplier<T> //供给性接口
	T get();
	
Function<T, R> //函数性接口
	R apply(T t);
	
Predicate<T> //断言性接口
	boolean test(T t);
```

## Stream 流

**特点**

+ Stream自己不会存储元素。
+ Stream不会改变源对象。相反，他们会返回一个持有结果的新Stream 
+ Stream操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。

**1.创建流**

+ 可以使用Collection系列的 stream() 或 parallelStream()

```java
List<String> list = new ArrayList<>();
Stream<String> stream1 = list.stream();
```

+ 通过Arrays中的静态方法 stream() 获取数组流

```java
User[] users = new User[10];
Stream<User> stream2 = Arrays.stream(users);
```

+ 通过Stream类中的静态方法of()

```java
Stream<String> stream3 = Stream.of("aa", "bb", "cc");
```

+ 无限流

```java
Stream<Integer> stream4 = Stream.iterate(0, (x) -> x + 2);
stream4.limit(10).forEach(System.out::println);
```

**2.中间操作**

+ 筛选与切片
  + filter —— 接受Lambda，从流中排除某些元素
  + limit —— 截断流，使其元素不超过给定的限度
  + skip(n) —— 跳过元素，返回一个扔掉了前n个元素的流。若流中元素不足n个，则返回一个空流。与limit（n）互补
+ 映射
  + map —— 将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素
  + flatMap —— 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流
+ 排序
  + sorted() —— 自然排序
  + sorted(Comparator com) —— 定制排序

**3.终止操作**

+ 查找与匹配：接受一个 Predicate 函数式接口

  + allMatch - 检查是否匹配所有元素
  + anyMatch - 检查是否至少匹配一个元素
  + noneMatch - 检查是否没有匹配所有元素
  + findFirst - 返回第一个元素
  + findAny - 返回当前流中的任意元素
  + count - 返回流中元素的总个数
  + max - 返回流中最大值
  + min - 返回流中最小值

+ 规约与收集

  + reduce(T identity, BinaryOperator) / reduce(BinaryOperator)

    ```java
    Integer sum = list.stream().reduce(0, (x, y) -> x + y);
    System.out.println(sum);
    
    Optional<Double> op = employee.stream().map(Employee::getSalary).reduce(Double::sum);
    System.out.println(op.get());
    ```

  + collect：将流转换为其他形式。接收一个collector接口的实现，用于给Stream中元素做汇总的方法

    ```java
    // 汇总为集合
    List<String> list = employee.stream()
        .map(Employee::getName)
        .collect(Collectors.toList());
    
    List<String> list = employee.stream()
        .map(Employee::getName)
        .collect(Collectors.toSet());
    
    List<String> list = employee.stream()
        .map(Employee::getName)
        .collect(Collectors.toCollection(HashSet::new));
    
    // 总数
    Long count = employee.stream()
        .collect(Collectors.counting());
    
    // 平均值
    Double count = employee.stream()
        .collect(Collectors.averagingDouble(Employee::getSalary));
    
    // 综合
    Double sum = employee.stream()
        .collect(Collectors.summingDouble(Employee::getSalary));
    
    // 最大值
    Optional<Employee> max = employee.stream()
        .collect(Collectors.maxBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));
    System.out.println(max.get());
    
    // 最小值
    Optional<Double> min = employee.stream()
        .map(Employee::getSalary)
        .collect(Collectors.minBy(Double::compare));
    System.out.println(min.get());
    ```

    



