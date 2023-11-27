

## 组件基础样式方案

React组件基础的样式控制有俩种方式

1.行内样式（不推荐）

```jsx
function App() {
    return (
        <div>
            <div style={{ color: 'red' }}>this is div</div>
        </div>
    )
}
// 或者
const style = {
    color: 'red',
    fontSize: '20px'
}
function App() {
    return (
        <div>
            <div style={style}>this is div</div>
        </div>
    )
}
```

2.class类名控制

`index.css`

```css
.foo {
	color: red;
}
```

`App.js`

```jsx
import './index.css'

function App() {
    return (
        <div>
            <span className='foo'>this is div</span>
        </div>
    )
}
```

## `classnames ` 优化类名控制

原方法：

```jsx
<span 
    key={tab.type} 
    className={`nav-item ${activeTab === tab.type && 'active'}`}>
    {tab.label}
</span>
```

问题：字符串的拼接不够只管，也容易出错

新方法：

```shell
npm install classnames
```

```jsx
import classnames from 'classnames'

function App() {
    return (
        <span 
            key={tab.type} 
            className={classnames('nav-item', {active: activeTab === tab.type})}>
            {tab.label}
        </span>
    )
}
```

