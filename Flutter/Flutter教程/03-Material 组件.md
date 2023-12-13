# Material 组件

官方将 Material 组件分为为几个类型：

- 应用程序结构和导航
- Button
- 输入和选择
- 对话框，警告弹框和面板
- 信息显示
- 布局

## 应用程序结构和导航

### MaterialApp

- `MaterialApp`是整个应用程序的根小部件，它通常包裹整个应用程序并提供一些全局配置，例如主题、语言环境、导航等。
- 它是Material Design风格应用程序的起点，提供了整个应用程序的整体外观和行为。
- `MaterialApp`通常包含`home`、`routes`等属性，用于定义应用程序的主页和路由。

```dart
void main() {
  runApp(
    MaterialApp(
      title: 'TableCalendar Example123',
      home: MyHomePage(),
      theme: ThemeData(
        // 定义应用程序的主题
      ),
      routes: {
        // 定义其他路由
      },
    ),
  );
}
```

+ title：指定应用程序的整体标题，该标题可能会在应用程序切换任务视图（recents view）时显示，也可能会显示在设备的应用程序切换器中。

### Scaffold

- `Scaffold`是应用程序界面的框架，它定义了应用程序的基本结构，如顶部栏、底部栏、抽屉等。
- `Scaffold`通常包含`appBar`、`body`、`bottomNavigationBar`等属性，用于定义应用程序的布局。
- 它是一个Material Design布局的基本单元，为应用程序提供了标准的界面元素。

```dart
class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('My App'),
      ),
      body: Center(
        child: Text('Hello, World!'),
      ),
      // 可以包含底部导航栏等其他属性
    );
  }
}
```

#### AppBar

顶部栏

```dart
class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(// 顶部栏
        title: Text('My App'), // 页面标题
        centerTitle: true, // 标题是否居中
        backgroundColor: Colors.white, // 背景色
      ),
    );
  }
}
```

