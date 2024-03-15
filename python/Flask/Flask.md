# 创建程序实例

```
from flask import Flask
app = Flask(__name__)
```

传入Flask类构造方法的第一个参数是模块或包的名称，我们应该使用特殊变量__name__。Python会根据所处的模块来赋予__name__变量相应的值，对于我们的程序来说（app.py），这个值为app。除此之外，这也会帮助Flask在相应的文件夹里找到需要的资源，比如模板和静态文件。Flask类是Flask的核心类，它提供了很多与程序相关的属性和方法。在后面，我们经常会直接在程序实例app上调用这些属性和方法来实现相关功能。在第一次提及Flask类中的某个方法或属性时，我们会直接以实例方法/属性的形式写出，比如存储程序名称的属性为app.name。

# 注册路由

只需为函数附加app.route（）装饰器，并传入URL规则作为参数，我们就可以让URL与
函数建立关联。这个过程我们称为注册路由（route），路由负责管理URL和函数之间的映射，而这个函数则被称为视图函数（view function）

```
@app.route('/')
def index():
	return '<h1>Hello, World!</h1>'
```

在这个程序里，app.route（）装饰器把根地址/和index（）函数绑定起来，当用户访问这个URL时就会触发index（）函数。这个视图函数可以像其他普通函数一样执行任意操作，比如从数据库中获取信息，获取请求信息，对用户输入的数据进行计算和处理等。最后，视图函数
返回的值将作为响应的主体，一般来说，响应的主体就是呈现在浏览器窗口的HTML页面。在最小程序中，视图函数index（）返回一行问候.
route()装饰器的第一个参数是URL规则，用字符串表示，必须以斜杠（/）开始。这里的URL是相对URL（又称为内部URL），即不包含域名的URL。

## 为视图绑定多个URL
一个视图函数可以绑定多个URL，比如下面的代码把/hi和/hello都绑定到say_hello（）函数上，这就会为say_hello视图注册两个路由，用户访问这两个URL均会触发say_hello（）函数，获得相同的响应.
```
@app.route('/hi')
@app.route('/hello')
def say_hello():
	return '<h1>Hello, Flask!</h1>'
```

## 动态URL
我们不仅可以为视图函数绑定多个URL，还可以在URL规则中添加变量部分，使用“<变量名>”的形式表示。Flask处理请求时会把变量传入视图函数，所以我们可以添加参数获取这个变量值。
```
@app.route('/greet/<name>')
def greet(name):
return '<h1>Hello, %s!</h1>' % name
```

因为URL中可以包含变量，所以我们将传入app.route（）的字符串称为URL规则，而不是URL。Flask会解析请求并把请求的URL与视图函数的URL规则进行匹配。比如，这个greet视图的URL规则为/greet/<name>，那么类似/greet/foo、/greet/bar的请求都会触发这个视图函数。
当URL规则中包含变量时，如果用户访问的URL中没有添加变量，
比如/greet，那么Flask在匹配失败后会返回一个404错误响应。一个很常
见的行为是在app.route（）装饰器里使用defaults参数设置URL变量的默
认值，这个参数接收字典作为输入，存储URL变量和默认值的映射。在
下面的代码中，我们为greet视图新添加了一个app.route（）装饰器，
为/greet设置了默认的name值：
@app.route('/greet', defaults={'name': 'Programmer'})
@app.route('/greet/<name>')
def greet(name):
return '<h1>Hello, %s!</h1>' % name
这时如果用户访问/greet，那么变量name会使用默认值
Programmer，视图函数返回<h1>Hello，Programmer！</h1>。上面的用
法实际效果等同于：
@app.route('/greet')
@app.route('/greet/<name>')
def greet(name='Programmer'):
return '<h1>Hello, %s!</h1>' % name