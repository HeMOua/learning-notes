# C#笔记

>  https://www.bilibili.com/video/BV1wx411K7rb/
>
> [Net Framework,Net Core 和 Net Standard 区别](https://www.cnblogs.com/szlblog/p/11803623.html)

## 基础

### 基本类型

+ int
+ char
+ string
  + `String` 属于整个 dotnet 平台上的数据类型，`string` 独属于 C# 的关键字
  + 字符串可以用`+`进行连接
  + `@“xxx”` 类似于 python 中的 `“”“xxx”“”`，并且还能直接使用转义字符
  + `$"xxx{}xxx"` 类似于 python 中的 `f'xxx{}xxx'`
+ double
+ decimal
+ var，隐式类型，声明时必须初始化

*1、占位符*

```C#
int a, b, c;
a = b = c = 30;
Console.writeLine("一{0}二{1}三{2}", a, b, c);

Console.writeLine("一{0:0.00}二{1}三{2}", a, b, c); // 保留两位小数
```

占位符参数可以多写，但是不能少写



*2、命名规范*

+ Camel：首字母小写，其他单词首字母大写，**应用于变量或者参数**
+ Pascal：所有单词首字母大小，**应用于类或方法**



*3、类型转换*

```c#
// Convert
string s = "123";
double d = Convert.ToDouble(s);
int n = Convert.ToInt32(s);

// Parse
int n1 = int.Parse("123");
double d1 = double.Parse.("123");
// 【小结】，以上两个方法进行类型转换，失败了会抛出异常

// TryParse
int n2;
bool b = int.TryParse("123", out n2);
// 【小结】，TryParse 会返回当前转换是否成功的布尔值
// 如果成功，则第二个参数会被设置为转换后的值
// 如果失败，则第二个参数会被设置为 0

// 我们也可以直接在 out 后面定义数据，如下
int.TryParse("123", out int result);
Console.WriteLine(result);
```

### 可空数据

```c#
Nullable<int> a = null;
Console.WriteLine(a);

a.hasValue; // false
a.Value; // 若 a 为 null，则会报错
Console.WriteLine(a); // 输出空
```



### 异常捕获

```c#
try 
{
	...
}
catch
{
    ...
}
```

### 高级数据

*1、常量*

```c#
const int a = 1;
```

*2、枚举*

```c#
enum Week {
	Monday,
    ...
}
```

*3、结构*

```c#
struct Person {
    int _age;
    int _name;
}
```

结构变量内包含数个**字段**，通常需要前面加下划线

*4、数组*

```
int[] a = new int[10];
int[] a = {1, 2, 3};
int[] a = new int[] {1, 2, 3};
```

### 方法参数

在 C# 中，我们有三种方法参数的用法：out、ref 和 params。它们都具有不同的作用和使用方式。以下是它们的描述及需要注意的地方：

1. `out` 参数：
   - `out` 参数用于从方法中返回多个值。它要求在方法内部将参数赋予一个值，确保在方法结束时具有有效值。
   - 在调用方法之前，不需要为 `out` 参数赋初值。
   - `out` 参数在方法内部没有值的情况下，也必须在方法结束之前赋值。
   - 在调用方法时，必须在 `out` 参数前使用 `out` 关键字进行标记。
   - 示例：
     ```csharp
     void GetValues(out int a, out int b)
     {
         a = 10;
         b = 20;
     }
     
     // 调用方法
     int x, y;
     GetValues(out x, out y);
     Console.WriteLine($"x: {x}, y: {y}"); // 输出：x: 10, y: 20
     ```

2. `ref` 参数：
   - `ref` 参数用于将变量的引用传递给方法，使方法可以更改变量的值。
   - 在调用方法之前，必须为 `ref` 参数赋初值。
   - `ref` 参数在方法内部可以更改为新的值，并且对原始变量也会生效。
   - 在调用方法时，必须在 `ref` 参数前使用 `ref` 关键字进行标记。
   - 示例：
     ```csharp
     void ChangeValue(ref int num)
     {
         num = 20;
     }
     
     // 调用方法
     int value = 10;
     ChangeValue(ref value);
     Console.WriteLine(value); // 输出：20
     ```

3. `params` 参数：
   - `params` 参数允许方法接受可变数量的参数，而无需显式地指定参数数组的长度。
   - `params` 参数必须是方法的最后一个参数。
   - 可以将多个参数传递给 `params` 参数，也可以将数组作为参数传递。
   - 示例：
     ```csharp
     int Sum(params int[] numbers)
     {
         int sum = 0;
         foreach (int num in numbers)
         {
             sum += num;
         }
         return sum;
     }
     
     // 调用方法
     int result1 = Sum(1, 2, 3); // 传递多个参数
     int result2 = Sum(new int[] { 4, 5, 6 }); // 传递数组
     Console.WriteLine(result1); // 输出：6
     Console.WriteLine(result2); // 输出：15
     ```

需要注意的地方：

- 在使用 `out` 和 `ref` 参数时，必须在方法签名和方法调用处使用相应的关键字，以明确指示参数的特殊用法。
- 在使用 `out` 和 `ref` 参数时，确保在方法内部为它们赋予有效的值，否则会导致编译错误。
- 使用 `ref` 参数时，要注意方法内部对参数的修改会对原始变量产生影响。
- 使用 `params` 参数时，可以传递多个参数或者将数组作为参数传递，但它必须是方法的最后一个参数。
- 在调用带有 `params` 参数的方法时，可以不传递任何参数，此时将创建一个空数组。
- 在使用 `params` 参数时，要注意不要传递多个参数数组，否则会导致编译错误。

### 集合

#### Array

```c#
int[] nums = {3, 1, 2, 9, 8, 7};
Array.Sort(nums); // 直接修改，无返回值
Array.Reverse(nums);
```

#### ArrayList

```c#
ArrayList list = new ArrayList();

// 基础方法
list.Add(true);
list.AddRange(new int[] {1, 2, 3});
list.Clear();
list.Remove(xxx);
list.RemoveAt(index);
list.RemoveRange(start, end);
list.Sort();
list.Reverse();
list.Insert(index, obj);
list.InsertRange(index, new int[] {..});
bool isContain = list.Contains(obj);
int count = list.Count;

// 容量
int capacity = list.Capacity; // 初试为0
list.Add(x);  // list.Capacity = 4，首次容量为 4
for (int i = 0;i  < 3; i++) {
	list.Add(x);
}           // list.Capacity = 4，容器满后可容纳量不会变

list.Add(x);  // list.Capacity = 8，容器超额时会扩容一倍
```

#### Hashtable

```c#
Hashtable ht = new Hashtable();
ht[1] = "ad";
Console.WriteLine(ht[1]);
```

#### Dictionary

```c#
Dictionary<int, string> dit = new Dictionary<int, string>(); 
dit[1] = "asd";

Console.WriteLine(ht[1]);

foreach (KeyValuePair<int, string> kv in dit)
{
    Console.WriteLine($"{kv.Key}, {kv.Value}");
}
```



### 文件操作

#### Path

```C#
string path = @"C:\Users\fangq\Desktop\1.txt";
Console.WriteLine(Path.GetFileName(path)); // 1.txt
Console.WriteLine(Path.GetFileNameWithoutExtension(path)); // 1
Console.WriteLine(Path.GetExtension(path)); // .txt
Console.WriteLine(Path.GetDirectoryName(path)); // C:\Users\fangq\Desktop
Console.WriteLine(Path.GetFullPath(path)); // C:\Users\fangq\Desktop\1.txt
Console.WriteLine(Path.Combine("abc", "ad")); // abc\ad
```

#### File

```c#
// 基本操作
File.Create(path);
File.CreateText(path); // 创建 UTF-8 编码格式的文本
File.Delete(path);
File.Move(source, target);
File.Copy(source, target);   // 若文件存在则报错
File.Copy(source, target, true); // 若文件存在则覆盖
File.Exists();

// 读取文件
string path = @"C:\Users\fangq\Desktop\1.txt";
string content = "你真棒\n你大爷";
// (1) 写入字节
byte[] bytes = Encoding.GetEncoding("utf-8").GetBytes(content);
//byte[] bytes  = Encoding.Default.GetBytes(content);
File.WriteAllBytes(path, bytes);

// (2) 读取字节
byte[] bytes1 = File.ReadAllBytes(path);
string str = Encoding.Default.GetString(bytes1);
Console.WriteLine(str);

// (3) 读取所有行
string[] strings = File.ReadAllLines(path, Encoding.Default);
foreach (string s in strings)
{
    Console.Write(s);
}

// (4) 写入所有行
File.WriteAllLines(path, new string[] {"qian er", "beautify"});

// (5) 读取所有文本
string str2 = File.ReadAllText(path, Encoding.Default);
Console.WriteLine(str2);

// (6) 写入所有文本
File.WriteAllText(path, "qianer\n nice");

// (7) 追加文本
File.AppendAllText(path, "真的");
```

### 扩展

*1、Random*

生成随机数

```c#
Random r = new Random();
int number = r.Next(1, 11); // 左闭右开，即 1~10
```

*2、记录时间* ★★★

```c#
using System.Diagnostics;
StopWatch sw = new StopWatch();
sw.Start();
...
sw.Stop();
Console.WriteLine(sw.Elapsed);
```

*3、Console*

+ `Console.writeLine()`  输出换行
+ `Console.write()`           输出不换行
+ `Console.ReadLine()`    返回string
+ `Console.Read()`             返回int
+ `Console.ReadKey()`       等待用户按下

## 委托

### 基本概念

委托（Delegate）是 C# 中的一个重要概念，它允许将方法作为参数传递、将方法赋值给变量以及在运行时动态调用方法。以下是关于委托的详细知识点的整合和代码示例：

1. 委托的声明和使用：
   - 委托通过 `delegate` 关键字进行声明，定义了方法的签名（返回类型和参数列表）。
   - 委托变量可以用于存储具有相同签名的方法引用。
   - 可以使用 `new` 关键字和委托的构造函数来实例化委托对象。
   - 通过使用委托变量的调用运算符 `()` 来调用委托所引用的方法。

2. 委托的类型安全性：
   - 委托是类型安全的，它们的签名必须与所引用的方法的签名完全匹配。
   - 委托的类型安全性保证了在调用委托时不会发生参数类型不匹配或返回类型不匹配的错误。

3. 多播委托（Multicast Delegate）：
   - 多播委托可以引用多个方法，并按顺序调用它们。
   - 使用 `+` 运算符将方法添加到委托中，使用 `-` 运算符将方法从委托中移除。

4. 匿名方法和 Lambda 表达式：
   - 匿名方法是一种无需显式定义方法的方式，可以直接将方法逻辑定义在委托的实例化过程中。
   - 匿名方法使用 `delegate` 关键字，可以包含参数列表和方法体，并与委托的签名匹配。
   - Lambda 表达式是一种更简洁的方式来定义匿名方法，使用 `=>` 运算符，省略参数类型声明和花括号。

5. 预定义委托类型：
   - C# 提供了两个通用的预定义委托类型：`Action` 和 `Func`。
   - `Action` 是一个通用的无返回值委托类型，用于引用不带返回值的方法。
     - `Action` 委托可以接受 0 到 16 个参数。
     - `Action` 委托的最后一个类型参数表示委托的返回类型，它始终为 `void`。
   - `Func` 是一个通用的具有返回值的委托类型，用于引用带有返回值的方法。
     - `Func` 委托的最后一个类型参数表示委托的返回类型，前面的类型参数表示方法的参数类型。
     - `Func` 委托可以接受 0 到 16 个参数。

委托是 C# 中实现事件、回调函数和异步编程的重要机制。通过委托，可以在代码中以更灵活和可复用的方式处理方法的引用和调用。委托的使用可以使代码更具可读性和可维护性，并促进模块化和分离关注点的设计原则的实现。

下面是一个综合示例，演示了委托的声明、实例化、多播、匿名方法、Lambda 表达式以及预定义委托类型的使用：

```csharp
using System;

delegate void MyDelegate(string message);

class Program
{
    static void Main()
    {
        // 委托的实例化
        MyDelegate myDelegate = new MyDelegate(ShowMessage);

        // 委托的调用
        myDelegate("Hello, World!");
        // 输出: ShowMessage: Hello, World!

        // 多播委托
        myDelegate += ShowAnotherMessage;
        myDelegate("Hello again!");
        // 输出: ShowMessage: Hello again!
        // 输出: ShowAnotherMessage: Hello again!

        // 匿名方法
        MyDelegate anonymousDelegate = delegate (string message)
        {
            Console.WriteLine("Anonymous Delegate: " + message);
        };
        anonymousDelegate("Hello from anonymous delegate!");
        // 输出: Anonymous Delegate: Hello from anonymous delegate!

        // Lambda 表达式
        MyDelegate lambdaDelegate = (message) =>
        {
            Console.WriteLine("Lambda Delegate: " + message);
        };
        lambdaDelegate("Hello from lambda delegate!");
        // 输出: Lambda Delegate: Hello from lambda delegate!

        // 预定义委托类型
        Action<string> actionDelegate = (message) =>
        {
            Console.WriteLine("Action Delegate: " + message);
        };
        // actionDelegate.Invoke(); 或者
        actionDelegate("Hello from Action delegate!");
        // 输出: Action Delegate: Hello from Action delegate!

        Func<int, int, int> addFunc = (a, b) =>
        {
            return a + b;
        };
        int result = addFunc(5, 3);
        Console.WriteLine("Func Delegate: " + result);
        // 输出: Func Delegate: 8
    }

    static void ShowMessage(string message)
    {
        Console.WriteLine("ShowMessage: " + message);
    }

    static void ShowAnotherMessage(string message)
    {
        Console.WriteLine("ShowAnotherMessage: " + message);
    }
}
```

以上代码示例综合展示了委托的多个方面，包括委托的实例化、调用、多播、匿名方法、Lambda 表达式以及预定义委托类型 `Action` 和 `Func` 的使用。通过这些示例，展示了委托在不同场景下的灵活应用和用法。

**注意**：难精通+易使用+功能强大东西，一旦被滥用则后果非常严重

+ 缺点1：这是一种方法级别的紧耦合，现实工作中要慎之又慎
+ 缺点2：使可读性下降、debug的难度增加
+ 缺点3：把委托回调、异步调用和多线程纠缠在一起，会让代码变得难以阅读和维护
+ 缺点4：委托使用不当有可能造成内存泄漏和程序性能下降

### 同步以及异步

委托的异步调用和同步调用涉及到委托与多线程的结合。异步调用允许委托以非阻塞方式执行，而同步调用则会阻塞当前线程直到委托执行完成。以下是对委托异步调用和同步调用的描述，并提供相应的示例代码：

1. 委托的异步调用：
   - 异步调用允许委托在后台线程上执行，而不会阻塞主线程。
   - 使用 `BeginInvoke` 方法异步调用委托，并使用 `EndInvoke` 方法获取异步操作的结果。
   - 异步调用通常适用于执行耗时操作，以避免阻塞主线程，保持用户界面的响应性。

以下是异步调用委托的示例代码：

```csharp
using System;
using System.Threading;

delegate void MyDelegate(string message);

class Program
{
    static void Main()
    {
        MyDelegate myDelegate = new MyDelegate(ShowMessage);

        // 异步调用委托
        IAsyncResult asyncResult = myDelegate.BeginInvoke("Hello, World!", null, null);

        // 主线程继续执行其他操作

        // 等待异步调用完成并获取结果
        myDelegate.EndInvoke(asyncResult);

        Console.WriteLine("Async delegate call completed.");
        Console.ReadLine();
    }

    static void ShowMessage(string message)
    {
        Console.WriteLine("ShowMessage: " + message);
        Thread.Sleep(2000); // 模拟耗时操作
    }
}
```

在上述代码中，我们创建了一个委托 `MyDelegate`，并使用 `BeginInvoke` 方法异步调用委托的 `ShowMessage` 方法。在异步调用之后，主线程可以继续执行其他操作。然后，通过调用 `EndInvoke` 方法来等待异步调用完成，并获取异步操作的结果。最后，我们输出一条完成的消息。

2. 委托的同步调用：
   - 同步调用会阻塞当前线程，直到委托执行完成。
   - 使用委托的直接调用方式即可进行同步调用。
   - 同步调用适用于希望等待委托完成并获取结果的情况。

以下是同步调用委托的示例代码：

```csharp
using System;

delegate void MyDelegate(string message);

class Program
{
    static void Main()
    {
        MyDelegate myDelegate = new MyDelegate(ShowMessage);

        // 同步调用委托
        myDelegate("Hello, World!");

        Console.WriteLine("Sync delegate call completed.");
        Console.ReadLine();
    }

    static void ShowMessage(string message)
    {
        Console.WriteLine("ShowMessage: " + message);
        // 不进行任何延迟，立即返回
    }
}
```

在上述代码中，我们创建了一个委托 `MyDelegate`，然后直接调用委托的 `ShowMessage` 方法进行同步调用。由于同步调用会阻塞当前线程，因此程序会等待委托执行完成后再输出一条完成的消息。

无论是异步调用还是同步调用，委托都是实现回调函数、事件处理、异步编程等的有力工具。根据具体的需求和场景，可以选择适合的调用方式来达到预期的效果。

## 事件

事件（Event）是 C# 中一种特殊的委托使用方式，它提供了一种用于实现观察者设计模式的机制。以下是关于事件的详细知识点的描述：

1. 事件的声明和使用：
   - 事件是通过在委托声明前添加 `event` 关键字来声明的。
   - 事件只能在声明它的类内部触发和订阅，不能在外部直接触发和订阅。
   - 事件可以通过 `+=` 和 `-=` 运算符进行订阅和取消订阅。
2. 发布者（Publisher）和订阅者（Subscriber）模型：
   - 事件实现了发布者和订阅者模型，其中发布者负责触发事件，而订阅者负责处理事件。
   - 发布者通过调用事件的委托来触发事件。
   - 订阅者通过将方法与事件的委托进行绑定来订阅事件。
   - 当事件触发时，所有订阅者绑定的方法会被调用。
3. 事件处理方法：
   - 事件处理方法是与事件关联的方法，用于处理事件被触发时的逻辑。
   - **事件处理方法必须与事件的委托具有相同的签名（返回类型和参数列表）。**
   - 事件处理方法可以是普通的实例方法、静态方法或匿名方法。
4. 标准事件模式：
   - 标准事件模式是一种约定，用于定义事件的委托和事件的触发方法。
   - 标准事件模式使用 `EventHandler` 委托和 `EventArgs` 类作为事件的参数类型。
   - 标准事件模式可以提供更一致和可读性更高的事件定义和使用方式。

事件模型的五个组成部分：

1. 事件的拥有者（event publisher、event source，对象）
2. 事件的订阅者（event subscriber，对象）
3. 事件成员（event，成员）
4. 事件处理器（event handler，成员）—— 本质上是一个回调方法
5. 事件订阅 —— 把事件处理器和事件关联在一起

以下是一个示例代码来演示事件的使用：

```csharp
using System;

class Program
{
    static void Main()
    {
        // 发布者类
        Publisher publisher = new Publisher();

        // 订阅者类
        Subscriber subscriber1 = new Subscriber("Subscriber 1");
        Subscriber subscriber2 = new Subscriber("Subscriber 2");

        // 订阅事件
        publisher.MyEvent += subscriber1.HandleEvent;
        publisher.MyEvent += subscriber2.HandleEvent;

        // 发布事件
        publisher.RaiseEvent();
        // 输出:
        // Event raised by Publisher
        // Event handled by Subscriber 1
        // Event handled by Subscriber 2

        // 取消订阅事件
        publisher.MyEvent -= subscriber2.HandleEvent;

        // 再次发布事件
        publisher.RaiseEvent();
        // 输出:
        // Event raised by Publisher
        // Event handled by Subscriber 1
    }
}

class Publisher
{
    // 声明事件
    public event EventHandler MyEvent;

    public void RaiseEvent()
    {
        Console.WriteLine("Event raised by Publisher");

        // 触发事件
        OnMyEvent(EventArgs.Empty);
    }

    protected virtual void OnMyEvent(EventArgs e)
    {
        // 检查事件是否有订阅者
        if (MyEvent != null)
        {
            // 调用事件的委托，通知所有订阅者
            MyEvent(this, e);
        }
    }
}

class Subscriber
{
    private string name;

    public Subscriber(string name)
    {
        this.name = name;
    }

    public void HandleEvent(object sender, EventArgs e)
    {
        Console.WriteLine($"Event handled by {name}"); 
    }
}
```

在上述代码中，我们创建了一个发布者类 `Publisher` 和两个订阅者类 `Subscriber`。发布者声明了一个事件 `MyEvent`，订阅者通过订阅该事件来处理逻辑。

在 `Main` 方法中，我们创建了一个发布者实例和两个订阅者实例。然后，我们订阅了事件 `MyEvent`，发布了事件并触发了订阅者的处理 方法。之后，我们取消了其中一个订阅者的订阅，并再次发布了事件。

通过事件，我们可以实现松耦合的观察者设计模式，允许发布者和订阅者之间的解耦和灵活性。订阅者可以根据需要选择订阅事件并定义自己的事件处理方法，以响应事件的触发。

## 面向对象

### 字段属性

```c#
class Test
{
    private int age; // 称之为字段，通常是 readonly
    public int Age { get; set; }; // 称之为属性，反编译后会生成 2 个方法 和一个字段
}
```

### 访问级别

以下是按照可访问级别大小重新调整的表格，从最大访问级别到最小访问级别排列：

| 修饰符             | 级别         | 适用成员       | 描述                                                         |
| ------------------ | ------------ | -------------- | ------------------------------------------------------------ |
| public             | 公开         | 类、成员、类型 | 可以从任何位置访问，没有访问限制。                           |
| protected internal | 受保护的内部 | 成员、类型     | 可以在当前程序集及其派生类中访问，也可以在其他程序集中访问，前提是访问类必须是派生类的内部或可见类。 |
| internal           | 内部         | 类、成员、类型 | 可以在当前程序集（Assembly）中访问，但对于其他程序集不可见。 |
| protected          | 受保护       | 成员           | 仅在当前类或结构及其派生类中可访问。                         |
| private protected  | 私有受保护   | 成员           | 仅在当前程序集中的派生类中可访问。                           |
| private            | 私有         | 成员           | 仅在当前类或结构中可访问。                                   |

**简略描述：**

namespace 下的所有 “直属元素” 只能使用`public`、`internal`两种访问权限，默认为`internal`。 

而 “直属元素” 内部的 “元素” 可以使用所有的访问权限，对于

+ 接口、枚举，它内部元素默认访问权限为 `public`
+ 其他所有的 “直属元素” 内部的 “元素” 的访问权限为 `private`
+ 因此对于抽象类来说，它的方法访问权限必须在 private 之上才不会报错

>**当前程序集（Assembly）**是指正在执行的代码所属的程序集（Assembly）。一个程序集是一个逻辑上的单元，它可以包含一个或多个相关的代码文件、资源和元数据。程序集在运行时加载到应用程序域中，用于执行和管理代码。
>
>举个例子，假设你正在开发一个名为 "MyApp" 的应用程序，它由多个项目组成，其中包括一个名为 "MyApp.Core" 的类库项目。当你在 "MyApp.Core" 项目中编写代码时，这些代码将被编译成一个独立的程序集。
>
>在这个例子中，"MyApp.Core" 程序集就是当前程序集。它是包含你的代码的逻辑单元，在运行时加载到应用程序域中。只有在同一个程序集中的代码才能访问当前程序集中的成员，而其他程序集中的代码则无法直接访问。
>
>另外，可以使用 `Assembly.GetExecutingAssembly()` 方法来获取当前正在执行的代码所属的程序集。这可以在代码中动态获取当前程序集的信息，例如获取程序集的名称、版本等。这在一些反射操作和动态加载程序集的场景中非常有用。

### 空引用检查 ★★

知识点：C# 中的 null 条件成员访问运算符

在 C# 中，问号 `?` 是条件访问运算符，也称为 null 条件成员访问运算符。它用于在调用方法、访问属性或访问数组元素之前进行空引用检查，以避免在引用为 null 的情况下引发 NullReferenceException 异常。

使用 `?.` 运算符，您可以简化空引用检查的代码，并提高代码的安全性。它是 C# 6.0 引入的一种语法糖，可以在访问可能为 null 的成员时进行空引用检查。

以下是一个示例来说明 null 条件成员访问运算符的使用：

```csharp
public class Person
{
    public string Name { get; set; }
    public int? Age { get; set; }
}

Person person = null;
string personName = person?.Name; // 如果 person 为 null，则 personName 也为 null
int? personAge = person?.Age;     // 如果 person 为 null，则 personAge 也为 null
```

在上述示例中，我们创建了一个 `Person` 类，并将 `person` 对象设置为 null。通过使用 `?.` 运算符，我们可以安全地访问 `person` 对象的属性 `Name` 和 `Age`，而不需要手动进行空引用检查。如果 `person` 为 null，那么 `personName` 和 `personAge` 也会被赋值为 null。

**总结：**

问号 `?` 是 C# 中的 null 条件成员访问运算符，用于在访问可能为 null 的成员时进行空引用检查。它简化了代码，并提供了更安全的方式来处理可能为 null 的引用，避免了 NullReferenceException 异常。使用 `?.` 运算符可以提高代码的可读性和健壮性。

### 继承

```c#
public class Animal {
	private string name;
    public Animal(string name) {
        this.name = name;
    }
}

public class Cat : Animal{
    public Cat(string name): base(name) {}
}
```

### 接口

显式实现

```c#
interface IKiller {
	void Kill();
}
interface IGentleman {
    void Love();
}

class WarmKill: IGentleman, IKiller {
    public void Kill() {
        ...
    }
    
    void IGentleman.Love() {
        ...
    }
}

static void Main(string[] args) {
    var wk = new WarmKill(); 
    wk.Kill(); // 正常
    wk.Love(); // 报错
    ((IGentleman)wk).Love(); // 必须转为 显式接口 所表示的类型能调用接口方法
    (wk as IGentleman).Love();
}
```

**注意**：显示接口不能添加任何访问修饰符

需要注意的是，显式接口实现可以用于解决类实现多个接口时可能出现的成员名冲突的问题。同时，它还提供了一种将接口的实现细节隐藏起来的方式，只能通过接口类型来调用这些方法。

### 反射

```c#
ITank tank = new HeavyTank();
var t = tank.GetType();
object o = Activator.CreateInstance(t);
MethodInfo fireMi = t.GetMethod("fire");
fireMi.Invoke(o, null);
```

### 特性

```c#
namespace CommonLib
{
    public class UnfinishAttribute: Attribute
    {
    }
}
```

需要继承 `Attribute`，并且类名要以 `Attribute` 结尾

### *反射案例*

```c#
using CommonLib;
using System.Runtime.Loader; // AssemblyLoadContext 需要

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            string folder = Path.Combine(Environment.CurrentDirectory, "Lib");
            string[] files = Directory.GetFiles(folder);
            List<Type> assmeblyTypes = new List<Type>();
            foreach (string file in files)
            {
                var assmebly = AssemblyLoadContext.Default.LoadFromAssemblyPath(file);
                Type[] types = assmebly.GetTypes();
                foreach (Type type in types)
                {
                    if (type.GetMethod("Run") != null)
                    {
                        if (type.GetCustomAttributes(typeof(UnfinishAttribute), false).Length > 0) continue;
                        assmeblyTypes.Add(type);
                    }
                }
            }

            while(true)
            {
                for (int i = 0; i < assmeblyTypes.Count; i++)
                {
                    Console.WriteLine($"{i + 1} {assmeblyTypes[i].Name}");
                }
                Console.WriteLine("请输入宠物编号：");

                int index;
                if(!int.TryParse(Console.ReadLine(), out index))
                {
                    Console.WriteLine("输入错误请重新输入");
                    continue;
                }

                if (index > assmeblyTypes.Count || index < 1)
                {
                    Console.WriteLine("输入错误请重新输入");
                    continue;
                }

                int times = int.Parse(Console.ReadLine());
                Type type = assmeblyTypes[index - 1];
                IExec obj = Activator.CreateInstance(type) as IExec;
                obj.Run(times);
            }
        }
    }
}
```

## 添加外部依赖包

右击项目 -> `Manage NuGet Packages`  -> `Borwse` -> 搜索框

+ 依赖注入：`Microsoft.Extensions.DependencyInjection`
+ 测试类：`Moq`

### 依赖注入

```c#
using Microsoft.Extensions.DependencyInjection;

static void Main(string[] args)
{
    ServiceCollection sc = new ServiceCollection();
    sc.AddScoped(typeof(IGentleman), typeof(WarmKill));  // 接口以及实现类
    sc.AddScoped<MyKiller>(); // 直接指向具体的类
    
    var sp = sc.BuildServiceProvider();
    var man = sp.GetService<IGentleman>();
    man.Love();
    var killer = sp.GetService<MyKiller>();
    killer.Kill();
}
```

