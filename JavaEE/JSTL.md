# JSTL

## 引入

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
```

如果使用的是`idea`编译器，敲一个`jsp`然后根据智能提示找`core`就能快速找到

prefix="c" 表示后续的标签使用都会以**<c:** 开头

## 变量设置

1、set

```jsp
<c:set var="name" value="${'gareen'}" scope="request" />
```

在作用域request中设置name,相当于

```jsp
<%request.setAttribute("name","gareen")%>
```

2、out

```jsp
<c:out value="${name}" />
```

相当于

```jsp
<%=request.getAttribute("name")%>
```

 3、删除

```jsp
<c:remove var="name" scope="request" />
```

在作用域request中删掉name,相当于

```
<%request.removeAttribute("name")%>
```

作用域可以是pageContext, request, session, application

## 条件判断

JSTL通过`<c:if test="">` 进行条件判断

但是JSTL**没有**`<c:else>`，所以常用的办法是在`<c:if`的条件里取反

配合if使用的还有通过`empty`进行为空判断

empty可以判断**对象**是否为null，**字符串长度**是否为0，**集合长度**是否为0

## 循环标签

```jsp
<c:forEach begin="1"end="4"step="2">
    循环休
</c:forEach>
```

1、作用：
			循环内容进行处理

2、使用：

+ begin：声明循环开始位置
+ end：声明循环结束位置
+ step：设置步长
+ varstatus：声明变量记录每次循环的数据（角标，次数，是否是第一次循环，是否是最后一次循环）
  注意：数据存储在作用域中，需要使用EL表达式获取。
  例如：`${vs.index}--${vs.count}--${vs.first}--${vs.last}`
+ items：声明要遍历的对象。结合EL表达式获取对象var：声明变量记录每次循环的结果。存储在作用域中，需要使用EL表达式获取。