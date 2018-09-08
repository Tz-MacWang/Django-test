#### 默认创建项目时产生的文件：

```
mysite/
	manage.py
	templates
	mysite/
		__init__.py
		settings.py
		urls.py
		wsgi.py
	polls/
		migrations/
			__init__.py
		__init__.py
		admin.py
		apps.py
		models.py
		tests.py
		views.py
```

这些文件分别是：

* 最外层的 `mysite/` 目录：项目的根目录，可以任意起名，也就是项目名
  * `manage.py`：命令行工具，用于在命令行操作Django工程。
  * `templates/`：项目的模板目录
  * 第二层 `mysite/` 目录：项目的实际操作函数文件存放目录
    * `mysite/__init__.py`：用来表明该目录为一个python包
    * `mysite/setting.py`：项目的配置文件，用来对项目进行设置
    * `mysite/urls.py`：项目的URL声明文件，存放着项目所有用到的网络站点
    * `mysite/wsgi.py`：支持WSGI接口的项目的web服务的接入点，细节见**WSGI接口**
  * `polls/` 目录：项目的application目录
    * `polls/__init__.py` 
    * `polls/models.py` 在这个文件下，创建应用所需的models
    * `polls/views.py` 在这个文件下，创建应用所需的视图
    * `polls/migrations/` 数据库与models交互

以上是项目的初始化文件，如果在创建Django项目时忘记创建Application目录，可以使用如下方法添加Application。

1、工具栏Tools

2、Run manage.py Task（快捷键 Ctrl + Alt + R）

3、在控制窗口输入startapp hello，回车（其中hello为app名字）



### WSGI接口：

Web应用的本质是：

1、浏览器发送一个HTTP请求

2、服务器收到请求，生成一个HTML文档

3、服务器将HTML文档作为HTTP响应的Body发送给浏览器

4、浏览器收到HTTP响应，从HTTP Body中取出HTML文档并显示

所以，最简单的Web应用就是把HTML用文件保存好，用一个现成的HTTP服务器软件，接收用户请求，从文件中读取HTML，返回给浏览器。常见的静态服务器是这么做的。

但如果需要动态生成HTML，则不是使用上面的做法。

正确的做法是底层代码由专门的服务器软件实现，而上成的HTML文档用python实现。这就需要使用一个接口，这个接口就是WSGI：Web Server Gateway Interface

WSGI接口定义非常简单，只需要实现一个函数，就可以响应HTTP请求。

```python
# 最简单的Web：'hello web!'
def application(environ, start_response):
  start_response('200 OK', [('Content-Type', 'text/html')])
  return '<h1> HELLO, WEB!</h1>'
```

这个 `application()` 函数就是符合WSGI标准的HTTP处理函数，它接收两个参数：

* environ：一个包含所有HTTP请求信息的dict对象（字典对象）
* start_response：一个发送HTTP响应的函数

在 `application()` 函数中，调用

```python
start_response('200 OK', [('Content-Type', 'text/html')])
```

就发送了HTTP响应的Header，注意：Header只能发送一次，也就是只能调用一次 `start_response()` 函数。
`start_response()` 函数接收两个参数：HTTP响应码、一组List表示的HTTP header。每个header用一个包含两个str的tuple表示。

通常情况下，都应该把 `Content-Type` 头发送给浏览器。很多其他常用的 HTTP header 也应该发送。

然后，函数的返回值 `'<h1>HELLO, WEB!</h1>'` 将作为HTTP响应的HTML代码发送给浏览器。

有了WSGI，我们关心的就是如何从 `environ` 这个`dict` 对象拿到HTTP请求信息，然后构造HTML，再通过 `start_response()` 发送给Header，最后返回Body。

整个 `application()` 函数本身不涉及任何解析HTTP的部分。

但是 `application()` 怎么调用？如果我们自己调用，两个参数`environ`和`start_response`我们没法提供，返回的HTML代码也没法发给浏览器。

所以 `application()` 函数必须由WSGI服务器来调用。Python内置了一个WSGI服务器模块，模块名叫`wsgiref`，但是这个模块不考虑任何运行效率，仅供开发和测试使用。

#### 运行WSGI服务

WSGI处理函数部分：

```python
# hello.py

def application(environ, start_response):
  start_response('200 OK', [('Content-Type', 'text/html')])
  return '<h1>Hello, web!</h1>'
```

启动WSGI服务器部分：

```python
# server.py

from wsgiref.simple_server import make_server
from hello import application

httpd = make_server('', 8000, application)
print('Server HTTP on port 8000...')

httpd.server_forever()
```

运行 server.py 文件，即可启动WSGI服务器

#### 总结

无论多么复杂的Web应用程序，入口都是 WSGI 处理函数。HTTP请求的所有输入信息都可以通过`environ`获得，HTTP响应的输出都可以通过`start_response()` 加上返回值作为HTTP的Body