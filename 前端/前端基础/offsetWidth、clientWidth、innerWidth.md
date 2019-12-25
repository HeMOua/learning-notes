# offsetWidth、clientWidth、innerWidth

+ 元素视图属性
     * offsetWidth 水平方向 width + 左右padding + 左右border-width
     * offsetHeight 垂直方向 height + 上下padding + 上下border-width
     * clientWidth 水平方向 width + 左右padding
     * clientHeight 垂直方向 height + 上下padding
     * offsetTop 获取当前元素到 定位父节点 的top方向的距离
     * offsetLeft 获取当前元素到 定位父节点 的left方向的距离
     * scrollWidth 元素内容真实的宽度，内容不超出盒子高度时为盒子的clientWidth
     * scrollHeight 元素内容真实的高度，内容不超出盒子高度时为盒子的clientHeight
+ Window视图属性（低版本IE浏览器[<IE9]不支持） 【自测包含滚动条，但网络教程都说不包含？？？】
  + innerWidth 浏览器窗口可视区宽度（不包括浏览器控制台、菜单栏、工具栏） 
  + innerHeight 浏览器窗口可视区高度（不包括浏览器控制台、菜单栏、工具栏）
+ Document文档视图（低版本IE的innerWidth、innerHeight的代替方案）
  + document.documentElement.clientWidth 浏览器窗口可视区宽度（不包括浏览器控制台、菜单栏、工具栏、滚动条）
  + document.documentElement.clientHeight 浏览器窗口可视区高度（不包括浏览器控制台、菜单栏、工具栏、滚动条）
  + document.documentElement.offsetHeight 获取整个文档的高度（包含body的margin）
  + document.body.offsetHeight 获取整个文档的高度（不包含body的margin）
  + document.documentElement.scrollTop 返回文档的滚动top方向的距离（当窗口发生滚动时值改变）
  + document.documentElement.scrollLeft 返回文档的滚动left方向的距离（当窗口发生滚动时值改变）
+ 元素方法
  + getBoundingClientRect() 获取元素到body
    + bottom: 元素底边（包括border）到可视区最顶部的距离
    + left: 元素最左边（不包括border）到可视区最左边的距离
    + right: 元素最右边（包括border）到可视区最左边的距离
    + top: 元素顶边（不包括border）到可视区最顶部的距离
    + height: 元素的offsetHeight
    + width: 元素的offsetWidth
    + x: 元素左上角的x坐标 
    + y: 元素左上角的y坐标 
  + scrollIntoView() 让元素滚动到可视区



