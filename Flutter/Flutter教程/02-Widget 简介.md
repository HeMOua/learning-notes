# Widget简介

## Widget概念

在 Flutter 中几乎任何对象都是一个 Widget，它可以涵盖到以下内容：

+ UI 组件
+ 功能性组件
  + 手势

## Widget接口

Widget 其实是一个描述 UI 的元素的配置信息，它并不表示最终展示在页面中的元素

Widget 类的关键代码如下所示：

```dart
@immutable // 不可变的
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });

  final Key? key;

  @protected
  @factory
  Element createElement();

  @override
  String toStringShort() {
    final String type = objectRuntimeType(this, 'Widget');
    return key == null ? type : '$type-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  @override
  @nonVirtual
  bool operator ==(Object other) => super == other;

  @override
  @nonVirtual
  int get hashCode => super.hashCode;

  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
  ...
}
```

+ `@immutable`：表示不可变的，Widget标注了这个注解表明它是不可变的，因此它内部的属性全都是不可变的。
  + 为什么不可变？因为每当 Widget 发生变化，他都要进行重建，然后替换老的 Widget，这样设置 Widget 属性可变没有意义，反正都要被替换
+ `DiagnosticableTree`：所继承的类，用来提供诊断信息的
+ `key`：类似于 Vue / React 中的 key，和 canUpdate 函数一起来判断 Widget 是否可以复用，同时也可以使重建构建时变得高效

## Flutter 的四棵树

1、Widget -> Element：根据 Widget 树会生成一个 Element 树，Element 树和 Widget 是一一对应的

2、Element->Render：根据 Element 树会生成 Render 树，他并不与 Render 是一一对应的，他主要用于渲染

3、Render->Layer：在上屏渲染之前，会根据 Render 树会生成 Layer 树



Element 的主要作用是作为 Widget 和 Render 之间的一个代理。

## StatelessWidget

无状态 Widget 比较简单，它继承了 widget，并重写了 `createElement` 方法

```dart
@override
StatelessElement createElement() => StatelessElement(this);
```

其中 `StatelessElement` 间接继承自`Element`类，与`StatelessWidget`相对应（作为其配置数据）。



下面是一个简单的例子

```dart
class Echo extends StatelessWidget  {
  const Echo({
    Key? key,  
    required this.text,
    this.backgroundColor = Colors.grey, //默认为灰色
  }):super(key:key);
    
  final String text;
  final Color backgroundColor;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        color: backgroundColor,
        child: Text(text),
      ),
    );
  }
}
```

主要用来显示一个文本，其中的 `build(BuildContext context)` 方法，传递了一个参数 `context`：

+ 它是一个 `BuildContext ` 实例，用来表示当前 Widget 的上下文

+ 每个 Widget 都有一个 `BuildContext`

+ 它提供了从当前 Widget 向上遍历 Widget 的方法，可以用来查找父级元素

  ```dart
  class ContextRoute extends StatelessWidget  {
    @override
    Widget build(BuildContext context) {
      return Scaffold(
        appBar: AppBar(
          title: Text("Context测试"),
        ),
        body: Container(
          child: Builder(builder: (context) {
            // 在 widget 树中向上查找最近的父级`Scaffold`  widget 
            Scaffold scaffold = context.findAncestorWidgetOfExactType<Scaffold>();
            // 直接返回 AppBar的title， 此处实际上是Text("Context测试")
            return (scaffold.appBar as AppBar).title;
          }),
        ),
      );
    }
  }
  ```

## StatefulWidget

和 StatelessWidget 一样，它也继承了 widget，并重写了 `createElement` 方法，但是额外添加了一个 `createState()` 方法。

```dart
@override
StatefulElement createElement() => StatefulElement(this);

@protected
State createState();
```

其中

+ `StatefulElement` 间接继承自`Element`类，与`StatefulWidget`相对应（作为其配置数据）。
+ `StatefulElement`中可能会多次调用`createState()`来创建状态（State）对象。

每个 Element 都会对应一个 State。当一个 StatefulWidget 同时插入到 widget 树的多个位置时，Flutter 框架就会调用该方法为每一个位置生成一个独立的State实例

## State

### 简介

一个 StatefulWidget 类会对应一个 State 类，但并不是永久的，因为 State 只有第一次插入到树中时会被创建，当 Widget 重新构建后，这个 State 会被自动设置到新的 Widget 上。

State 有两个关键属性：

+ `widget`：与 State 对应的 widget 实例，它由 flutter 动态设置值
+ `context`：是 `BuildContext` 实例，作用与 `StatelessWidget` 中的一致。

### 生命周期

1. `initState`：当 widget 第一次插入到 widget 树时会被调用，对于每一个State对象，Flutter 框架只会调用一次该回调
2. `didChangeDependencies`：当State对象的依赖发生变化时会被调用（Locale变化）
3. `build`：主要是用于构建 widget 子树的，会在如下场景被调用
   + 在调用`initState()`之后。
   + 在调用`didUpdateWidget()`之后。
   + 在调用`setState()`之后。
   + 在调用`didChangeDependencies()`之后。
   + 在State对象从树中一个位置移除后（会调用deactivate）又重新插入到树的其他位置之后。
4. `reassemble`：此回调是专门为了开发调试而提供的，在热重载(hot reload)时会被调用，此回调在Release模式下永远不会被调用
5. `didUpdateWidget `：在 widget 重新构建时，Flutter 框架会调用`widget.canUpdate`来检测 widget 树中同一位置的新旧节点，然后决定是否需要更新
6. `deactivate`：当 State 对象从树中被移除时，会调用此回调。如果移除后没有重新插入到树中则紧接着会调用`dispose()`方法。
7. `dispose`：当 State 对象从树中被永久移除时调用；通常在此回调中释放资源。

![2-5.a59bef97](img/2-5.a59bef97-1702350998506-2.jpg)

### 在 Widget 获取 State

#### 通过Context获取

`context`对象有一个`findAncestorStateOfType()`方法，该方法可以从当前节点沿着 widget 树向上查找指定类型的 StatefulWidget 对应的 State 对象。

```dart
class GetStateObjectRoute extends StatefulWidget {
  const GetStateObjectRoute({Key? key}) : super(key: key);

  @override
  State<GetStateObjectRoute> createState() => _GetStateObjectRouteState();
}

class _GetStateObjectRouteState extends State<GetStateObjectRoute> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("子树中获取State对象"),
      ),
      body: Center(
        child: Column(
          children: [
            Builder(builder: (context) {
              return ElevatedButton(
                onPressed: () {
                  // 查找父级最近的Scaffold对应的ScaffoldState对象
                  ScaffoldState _state = context.findAncestorStateOfType<ScaffoldState>()!;
                  // 打开抽屉菜单
                  _state.openDrawer();
                },
                child: Text('打开抽屉菜单1'),
              );
            }),
          ],
        ),
      ),
      drawer: Drawer(),
    );
  }
}
```

+ 一般来说，如果 StatefulWidget 的状态是私有的（不应该向外部暴露），那么我们代码中就不应该去直接获取其 State 对象；如果StatefulWidget的状态是希望暴露出的（通常还有一些组件的操作方法），我们则可以去直接获取其State对象。
+ 但是通过 `context.findAncestorStateOfType` 获取 StatefulWidget 的状态的方法是通用的，我们并不能在语法层面指定 StatefulWidget 的状态是否私有
+ 所以在 Flutter 开发中便有了一个默认的约定：如果 StatefulWidget 的状态是希望暴露出的，应当在 StatefulWidget 中提供一个`of` 静态方法来获取其 State 对象，开发者便可直接通过该方法来获取；如果 State不希望暴露，则不提供`of`方法。这个约定在 Flutter SDK 里随处可见。所以，上面示例中的`Scaffold`也提供了一个`of`方法，我们其实是可以直接调用它的：

```dart
Builder(builder: (context) {
  return ElevatedButton(
    onPressed: () {
      // 直接通过of静态方法来获取ScaffoldState
      ScaffoldState _state=Scaffold.of(context);
      // 打开抽屉菜单
      _state.openDrawer();
    },
    child: Text('打开抽屉菜单2'),
  );
}),
```

#### 通过 GlobalKey 获取

Flutter还有一种通用的获取`State`对象的方法——通过GlobalKey来获取！ 步骤分两步：

1. 给目标`StatefulWidget`添加`GlobalKey`。

   ```dart
   //定义一个globalKey, 由于GlobalKey要保持全局唯一性，我们使用静态变量存储
   static GlobalKey<ScaffoldState> _globalKey= GlobalKey();
   ...
   Scaffold(
       key: _globalKey , //设置key
       ...  
   )
   ```

2. 通过`GlobalKey`来获取`State`对象

   ```dart
   _globalKey.currentState.openDrawer()
   ```

GlobalKey 是 Flutter 提供的一种在整个 App 中引用 element 的机制。如果一个 widget 设置了`GlobalKey`，那么我们便可以通过

+ `globalKey.currentWidget`获得该 widget 对象
+ `globalKey.currentElement`来获得 widget 对应的element对象，
+ 如果当前 widget 是`StatefulWidget`，则可以通过 `globalKey.currentState`来获得该 widget 对应的state对象。则可以通过

> 注意：使用 GlobalKey 开销较大，如果有其他可选方案，应尽量避免使用它。另外，同一个 GlobalKey 在整个 widget 树中必须是唯一的，不能重复。