# Flask简介

> Fask是一个基于 Python开发并且依赖jinja2模板和 Werkzeug WSGI服务的一个微型框架，对于 Werkzeug本质是 Socket服务端，其用于接收http请求井对请求进行预处理，然后触发Flask框架，开发人员基于Fask框架提供的功能对请求进行相应的处理，并返回给用户，如果要返回给用户复杂的内容时，需要借助 jinja2模板来实现对模板的处理，即：将模板和数据进行渲染，将渲染后的字符返回给用户浏览器。
>
> “微”（micro）并不表示你需要把整个Web应用塞进单个 Python文件（虽然确实可以），也不意味着Fask在功能上有所欠缺。微框架中的“微”意味看 Flask 旨在保持核心简单而易于扩展。Flask不会替你做出太多决策——比如使用何种数据库。而那些 Flask所选择的——比如使用何种模板引擎——则很容易替换。除此之外的—切都由可由你掌握。

## PythonWeb框架

+ django
  + 大而全，重武器
  + 内部提供：ORM、Admin、中间件、Form、ModelForm、Session、缓存、信号、CSRF
+ flask
  + 短小精悍，可拓展强，第三方库丰富
+ tornado
  + 短小精悍，异步非阻塞
+ 其他
  + web.py
  + bottle.py，非常小，包含注释才1000行左右

## WSGI

Web服务器网关接口（Python Web Server Gateway Interface，缩写为WSGI）是为**Python语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口**。自从WSGI被开发出来以后，许多其它语言中也出现了类似接口。

+ django：wsgiref
+ flask：werkzeug 

## 快速使用

1、安装依赖

```shell
pip install flask
```

2、编码

2.1 使用较底层的werkzeug

```python
from werkzeug.wrappers import Request, Response
from werkzeug.serving import run_simple

@Request.application
def hello(request):
    return Response('hello')

if __name__ == '__main__':
    run_simple("localhost", 4000, hello)
```

请求一旦到来，执行`run_simple`方法的第三个参数的方法

2.2 使用Flask

```python
from flask import Flask

app = Flask(__name__)

@app.route('/index')
def index():
    return 'hello'

if __name__ == '__main__':
    app.run()
```

查看`app.run()`的源码能发现Flask底层调用的还是werkzeug的`run_sample`方法

## 管理系统项目

### app.py

```python
from flask import Flask, render_template, request, redirect, session

app = Flask(__name__)
app.secret_key = 'aueilkavlaskjfqoijwef'
app.debug = True

user_dict = {
    '1': {'age': 1, 'name': '李四'},
    '2': {'age': 2, 'name': '王五'},
    '3': {'age': 3, 'name': '赵六'},
}


@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    elif request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        print(request.form)
        if username == 'hemou' and password == '123':
            session['user_info'] = username
            return redirect('/')
        else:
            return render_template('login.html', msg='用户名或密码错误')
            # return render_template('login.html', **{ 'msg': '用户名或密码错误' })


@app.route('/')
def index():
    user_info = session.get('user_info')
    if not user_info:
        return redirect('/login')
    return render_template('index.html', user_dict=user_dict)


@app.route('/logout')
def logout():
    if session.get('user_info'):
        del session['user_info']
    return redirect('/login')


@app.route('/detail')
def detail():
    user_info = session.get('user_info')
    if not user_info:
        return redirect('/login')

    uid = request.args.get('id')
    user = user_dict.get(uid)
    return render_template('detail.html', user=user)


if __name__ == '__main__':
    app.run()
```

注意点：

+ 如果在session中设置值时，如`session['key'] = value`，需要给项目设置一个盐值`app.secret_key = 'xxx'`
+ `app.debug = True`可以开启调试模式，py文件变化后会立即重启项目
+ 请求方法必须都是大写，如`GET`、`POST`
+ 使用`render_template`渲染数据时，传递的数据模型有两种写法
  + `render_template('page', key=value)`
  + `render_template('page', **{ 'key': 'value' })`
+ 使用如`redirect`的方法时，一定要记得return
+ 删除session中的值时，如果这个值不存在会报错，所以删除前，最好先判断一下

### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
</head>
<body>
    <ul>
        {% for k, v in user_dict.items() %}
        <li>{{v.name}} <a href="detail?id={{k}}">查看详细</a></li>
        {% endfor %}
    </ul>
</body>
</html>
```

### login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
</head>
<body>
<h2 style="text-align: center">登陆</h2>
<form action="login" method="post">
    <input type="text" placeholder="用户名" name="username">
    <input type="password" placeholder="密码" name="password">
    <button>提交</button>{{msg}}
</form>
</body>
</html>
```

### detail.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
</head>
<body>
    用户名：{{user.name}}
    年龄：{{user.age}}
</body>
</html>
```

