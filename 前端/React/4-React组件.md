# React 组件

## 简介

> 概念：一个组件就是用户界面的一部分，它可以有自己的逻辑和外观，组件之间可以互相嵌套，也可以复用多次
>
> 组件化开发可以让开发者像搭积木一样构建一个完整的庞大的应用

在 React 中，一个组件就是首字母大写的函数，内部存放了组件的逻辑和视图 UI，渲染组件只需要把组件当成标签书写即可

```jsx
function Button() {
  return (
    <button>Click me!</button>
  );
}

function App() {

  return (
    <div className="App">
      {/* 自闭和 */}
      <Button />
      {/* 成对标签 */}
      <Button></Button>
    </div>
  );
}

export default App;
```

