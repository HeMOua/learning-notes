# 事件绑定

1、React基础事件绑定

语法：on + 事件名称 = {事件处理程序}，整体上遵循驼峰命名法

2、使用事件对象参数

语法：在事件回调函数中设置形参 e

3、传递自定义参数
语法：事件绑定的位置改造成**箭头函数**的写法，在执行clickHandler实际处理业务函数的时候传递实参

注意：不能直接写函数调用，这里事件绑定需要一个**函数引用**

4、同时传递事件对象和自定义参数

语法：在事件绑定的位置传递事件实参 e 和自定义参数，clickHandler 中声明形参，注意顺序对应

```jsx
function clickHandler() {
  console.log('click');
}

function App() {
  const clickHandler2 = (e) => {
    console.log('click2', e);
  }

  const clickHandler3 = (name) => {
    console.log('click3', name);
  }

  return (
    <div className="App">
      <button onClick={clickHandler}>Click</button>
      <button onClick={clickHandler2}>Click2</button>
      <button onClick={() => clickHandler3('hemou')}>Click3</button>
    </div>
  );
}

export default App;
```

