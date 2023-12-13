# Dart 基础教程

## 主函数

```dart
main() {

}

// 或者
void main() {
	print('demo');
}
```

## 常量与变量

### 变量类型

可以自动推断类型

```dart
var str = 'test';
var num = 2;

// 也可以指定类型
String str = 'test string';
int num = 123;

// 但是推断类型后不能再修改变量类型
String str = 'string';
str = 123; // 会报错
String str = 123; // 会报错
```

### 常量

`const` 值不变一开始就得赋值，`final` 可以开始不赋值，但只能赋一次

而 `final` 不仅有 `const` 的编译时常量的特性，最重要的它是运行时常量，使用惰性初始化的方式

```dart
final name 'Bob'; // Without a type annotation
final String nickname ='Bobby';

const bar 1000000; //Unit of pressure (dynes/cm2)
const double atm =  1.01325 * bar; //Standard atmosphere

final num;
num = 123; // 正常
num = 4; // 报错

final time = new DateTime.now();  // 正常
const time = new DateTime.now();  // 报错
```

### 变量详解

#### 字符串 ★

1、定义，类似python

```dart
String str = 'str';
String str = "str";
String str = """str
sdf""";
```

2、拼接

```dart
String str1 = 'Hello';
String str2 = 'Dart';

print('$str1 $str2');
print('$str1 $str2');
print(str1 + ' ' + str2);
```

#### 数值

只有 `int` 和 `double`

```dart
int a = 123;
double b = 23;
b = 23.5;
```

#### 布尔

```dart
bool flag = true;
flag = false;
```

#### List 集合

```dart
var list = ['str', 123, true];
print(list);
print(list.length); // 集合长度
print(list[0]);    // 取值

list.length = 2;  // 可修改长度
print(list); // ['str', 123]

// 指定集合类型
var list2 = <int>[12, 23];

// 动态添加数据
var list3 = [];
list3.add('123');

// 另一种方法
var list4 = new List(); // 注意：新版本不可使用了

// 固定长度集合
var list5 = List.filled(length, fill);
list5.length = 2;  // 报错，不可修改
```

#### Set 集合

```dart
var s = new Set();
s.addAll([1, 2, 3]);
s.forEach((value) => print(value))
```



#### Map 字典

```dart
var person = {
	'age': 20,
    'name':  'zhangsan'
}
// 只能通过[]取值
print(person['name']);
    
// 另一种创建 Map 的方式
var map = new Map();
map['name'] = 2;
```

#### 类型判断

`is` 关键词

```dart
var str = '123';
if (str is String) {
    ...
}
```

### forEach map where any every

1、forEach

```dart
List myList = [12, 3, 4];

var myList.forEach((value) {
    print('$value')
})
```

2、map

```dart
List myList = [12, 3, 4];

var newList = myList.map((value) {
	return value*2
}).toList(); // [24, 6, 8]
```

3、where

```dart
List myList = [12, 3, 4];

var newList = myList.where((value) {
	return value > 3
}).toList(); // [24, 4]
```

4、any every

```dart
List myList = [12, 3, 4];

var isAny = myList.any ((value) {
	return value > 5
}); // true

var isEvery = myList.every((value) {
	return value > 5
}); // false
```

## 函数

### 单表达式函数

如何函数内部只有一条表达式

```dart
bool isNoble(int atomicNumber) {
    return atomicNumber == 1;
}
```

可以简写为

```dart
bool isNoble(int atomicNumber) => atomicNumber == 1;
```

### 可选参数

```dart
String printInfo(String username, [int age, String sex='男']) {
    return 'name: $username, gender: $sex, $age';
}
print(printInfo('lisi', 10));
// name: 
```

### 命名参数

```dart
String printInfo(String username, {int age, String sex='男'}) {
    return 'name: $username, gender: $sex, $age';
}

print(printInfo('lisi', age:10))
```

### 箭头函数

```dart
List list = ['a', 1, 3];
list.forEach((e) => print(e));

List list = ['a', 1, 3];
list.forEach((e) => {
    print(e)
});
```

注意，参数部分的括号不能少，箭头函数只能写一行，这一样可以不加分号。

### 匿名函数

```dart
List list = ['a', 1, 3];
list.forEach((e) => {print(e)});

list.forEach((element) {
    print('test');
    print(element);
});
```

## 类

1、构造函数

```dart
class Person {
	String name;
	int age;
	Person(String name, int age) {
		this.name = name;
        this.age = age
	}
    
    // 上面的默认构造函数与下面等效
    Person(this.name, this.age);
    
    // 命名构造函数
    Person.info(String name) {
        this.name = name
    }
}

class Car {
    String? type;
    
    Car({ this.type });
    
	Car(String type) {
        this.type = type;
    }
    // 上面两个构造函数是等效的
}

void main {
    Person p = new Person('zhangsan', 12);
    p2 = new Person.info('lisi');
}
```

2、私有方法属性

通过加前缀`_`，如`_name`；

```dart
// animal.dart
class Animal {
    String _name;
    Animal(this._name);
}

// main.dart
import 'animal.dart';
void main () {
    Animal a = new Animal('2');
    print(a._name); // 会报错
}
```

3、属性方法

```dart
Class Rect {
	num h;
	num w;
    get area{
        return this.h * this.w
    }
    set rectH(int i) {
        this.h = i
    }
}
void main {
    Rect r = new Rect(3, 5);
    print('面积：${r.area}'); 
    
    r.rectH = 10;
}
```

4、minxins

```dart
Class A {}   

Class B {}

Class C extends Person with A, B {
    
}
```

## 包管理

### 依赖

1、需要的库

https://pub.dev/packages?q=

https://pub.flutter-io.cn/packages

2、创建 `pubspec.yaml` 文件，内容如下：

```yaml
name: xxx
description: A new flutter project
dependencies:
	http: ^0.12.0+1
	date_format: ^1.0.6
```

3、配置 dependencies

4、`pub get` 下载依赖

### 引入

```
import ‘lib/xxx.dart’;
import 'lib/xxx.dart' as xxx;
import 'lib/xxx.dart' show xxx;
import 'lib/xxx.dart' hide xxx;
```

## 新特性

### null safety

可控类型

```dart
int a = null; // 会报错
int? a = null; // 不会报错
```

