# 页面框架模板

## 前端Layout布局

### 参数解释

```js
// 当设置 true 的时候该路由不会在侧边栏出现 如401，login等页面，或者如一些编辑页面/edit/1
hidden: true // (默认 false)

//当设置 noRedirect 的时候该路由在面包屑导航中不可被点击
redirect: 'noRedirect'

// 当你一个路由下面的 children 声明的路由大于1个时，自动会变成嵌套的模式--如组件页面
// 只有一个时，会将那个子路由当做根路由显示在侧边栏--如引导页面
// 若你想不管路由下面的 children 声明的个数都显示你的根路由
// 你可以设置 alwaysShow: true，这样它就会忽略之前定义的规则，一直显示根路由
alwaysShow: true

name: 'router-name' // 设定路由的名字，一定要填写不然使用<keep-alive>时会出现各种问题
meta: {
  roles: ['admin', 'editor'] // 设置该路由进入的权限，支持多个权限叠加
  title: 'title' // 设置该路由在侧边栏和面包屑中展示的名字
  icon: 'svg-name' // 设置该路由的图标，支持 svg-class，也支持 el-icon-x element-ui 的 icon
  noCache: true // 如果设置为true，则不会被 <keep-alive> 缓存(默认 false)
  breadcrumb: false //  如果设置为false，则不会在breadcrumb面包屑中显示(默认 true)
  affix: true // 如果设置为true，它则会固定在tags-view中(默认 false)

  // 当路由设置了该属性，则会高亮相对应的侧边栏。
  // 这在某些场景非常有用，比如：一个文章的列表页路由为：/article/list
  // 点击文章进入文章详情页，这时候路由为/article/1，但你想在侧边栏高亮文章列表的路由，就可以进行如下设置
  activeMenu: '/article/list'
}
```

### 必填配置

+ name：用来设置路由的名字，不填的话可能会导致 kepp-alive 缓存出现问题
+ path：路径，与超链接的作用相同

### 类型

#### 不含子菜单

```js
{
    path: '/',
    component: Layout,
    redirect: 'index',
    children: [
      {
        path: 'index',
        component: () => import('@/views/index'),
        name: '首页',
        meta: { title: '首页', icon: 'dashboard', noCache: true, affix: true }
      }
    ]
}
```

#### 含子菜单

```js
 {
    name: 'Test',
    path: '/test',
    hidden: false,
    redirect: 'noRedirect',
    component: Layout,
    alwaysShow: true,
    meta: {
      title: '测试',
      icon: 'test',
      noCache: false
    },
    children: [
      {
        name: 'Test1',
        path: 'test1',
        hidden: false,
        component: () => import('@/views/test/index'),
        meta: {
          title: '测试1',
          icon: 'test',
          noCache: false
        }
      },
      {
        name: 'Test2',
        path: 'test2',
        hidden: false,
        component: () => import('@/views/test/child'),
        meta: {
          title: '测试2',
          icon: 'test',
          noCache: false
        }
      }
    ]
  }
```

#### 多级嵌套子菜单

```
[
  {
    'name': 'Log',
    'path': 'log',
    'hidden': false,
    'redirect': 'noRedirect',
    'component': 'ParentView',
    'alwaysShow': true,
    'meta': {
      'title': '日志管理',
      'icon': 'log',
      'noCache': false
    },
    'children': [
      {
        'name': 'Operlog',
        'path': 'operlog',
        'hidden': false,
        'component': 'monitor/operlog/index',
        'meta': {
          'title': '操作日志',
          'icon': 'form',
          'noCache': false
        }
      },
      {
        'name': 'Logininfor',
        'path': 'logininfor',
        'hidden': false,
        'component': 'monitor/logininfor/index',
        'meta': {
          'title': '登录日志',
          'icon': 'logininfor',
          'noCache': false
        }
      }
    ]
  }
]
```

需要将 'component' 设为 'ParentView'

