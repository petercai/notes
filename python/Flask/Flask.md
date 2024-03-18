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
当URL规则中包含变量时，如果用户访问的URL中没有添加变量，比如/greet，那么Flask在匹配失败后会返回一个404错误响应。一个很常见的行为是在app.route（）装饰器里使用defaults参数设置URL变量的默认值，这个参数接收字典作为输入，存储URL变量和默认值的映射：
```
@app.route('/greet', defaults={'name': 'Programmer'})
@app.route('/greet/<name>')
def greet(name):
return '<h1>Hello, %s!</h1>' % name
```
这时如果用户访问/greet，那么变量name会使用默认值 Programmer。上面的用法实际效果等同于：
```
@app.route('/greet')
@app.route('/greet/<name>')
def greet(name='Programmer'):
return '<h1>Hello, %s!</h1>' % name
```

# 启动开发服务器

Flask内置了一个简单的开发服务器（由依赖包Werkzeug提供），足够在开发和测试阶段使用。旧的启动开发服务器的方式是使用app.run（）方法，目前已不推荐使用（deprecated）。

## 自动发现程序实例 (FLASK_APP)
一般来说，在执行flask run命令运行程序前，我们需要提供程序实例所在模块的位置。我们在上面可以直接运行程序，是因为Flask会自动探测程序实例，自动探测存在下面这些规则：
- 从当前目录寻找app.py和wsgi.py模块，并从中寻找名为app或application的程序实例。
- 从环境变量FLASK_APP对应的值寻找名为app或application的程序实例。
因为我们的程序主模块命名为app.py，所以flask run命令会自动在其中寻找程序实例。如果你的程序主模块是其他名称，比如hello.py，那么需要设置环境变量FLASK_APP，将包含程序实例的模块名赋值给这个变量。
Linux或macOS系统使用export命令：
 export FLASK_APP=hello
在Windows系统中使用set命令：
> set FLASK_APP=hello
> 
## 管理环境变量 (.env and .flaskenv)
Flask的自动发现程序实例机制还有第三条规则：==如果安装了python-dotenv，那么在使用flask run或其他命令时会使用它自动从.flaskenv文件和.env文件中加载环境变量==。当安装了python-dotenv时，Flask在加载环境变量的优先级是：手动设置的环境变量 -> .env中设置的环境变量 -> .flaskenv设置的环境变量。
我们在项目根目录下分别创建两个文件：.env和.flaskenv。.flaskenv
用来存储和Flask相关的公开环境变量，比如FLASK_APP；而.env用来
存储包含敏感信息的环境变量，比如后面我们会用来配置Email服务器的账户名与密码。在.flaskenv或.env文件中，环境变量使用键值对的形式定义，每行一个，以#开头的为注释：
```
SOME_VAR=1
# 这是注释
FOO="BAR"
```
注意
.env包含敏感信息，除非是私有项目，否则绝对不能提交到Git仓库
中。当你开发一个新项目时，记得把它的名称添加到.gitignore文件中.

## 更多的启动选项
### 1. 使服务器外部可见 (--host)
我们在上面启动的Web服务器默认是对外不可见的，可以在run命令后添加--host选项将主机地址设为0.0.0.0使其对外可见：
```
$ flask run --host=0.0.0.0
```
这会让服务器监听所有外部请求。。
### 2.改变默认端口 (--port)
Flask提供的Web服务器默认监听5000端口，你可以在启动时传入参数来改变它：
```
$ flask run --port=8000
```
这时服务器会监听来自8000端口的请求，程序的主页地址也相应变成了http://localhost:8000/ 。

## 设置运行环境 (FLASK_ENV and FLASK_DEBUG)
Flask提供了一个FLASK_ENV环境变量用来设置环境，默认为production（生产）。在开发时，我们可以将其设为development（开发），这会开启所有支持开发的特性。为了方便管理，我们将把环境变量FLASK_ENV的值写入.flaskenv文件中：
```
FLASK_ENV=development
```
在开发环境下，调试模式（Debug Mode）将被开启，这时执行flask run启动程序会自动激活Werkzeug内置的调试器（debugger）和重载器 （reloader）. 如果你想单独控制调试模式的开关，可以通过FLASK_DEBUG环境变量设置，设为1则开启，设为0则关闭，不过通常不推荐手动设置这个值。

### 调试器
Werkzeug提供的调试器非常强大，当程序出错时，我们可以在网页上看到详细的错误追踪信息，这在调试错误时非常有用。调试器允许你在错误页面上执行Python代码。单击错误信息右侧的命令行图标，会弹出窗口要求输入PIN码，也就是在启动服务器时命令行窗口打印出的调试器PIN码（Debugger PIN）。输入PIN码后，我们可以单击错误堆栈的某个节点右侧的命令行界面图标，这会打开一个包含代码执行上下文信息的Python Shell，我们可以利用它来进行调试。
2.重载器
当我们对代码做了修改后，期望的行为是这些改动立刻作用到程序上。重载器的作用就是监测文件变动，然后重新启动开发服务器。当我们修改了脚本内容并保存后，会在命令行看到下面的输出：
```
Detected change in '/path/to/app.py', reloading
* Restarting with stat
```
默认会使用Werkzeug内置的stat重载器，它的缺点是耗电较严重，而且准确性一般。为了获得更优秀的体验，我们可以安装另一个用于监测文件变动的Python库Watchdog，安装后Werkzeug会自动使用它来监测文件变动：
```
$ pipenv install watchdog --dev
```
因为这个包只在开发时才会用到，所以我们在安装命令后添加了一个--dev选项，这用来把这个包声明为开发依赖。在Pipfile文件中，这个包会被添加到dev-packages部分。

# Flask shell
在开发Flask程序时，我们使用flask shell命令：
```
$ flask shell
App: app [development]
Instance: Path/to/your/helloflask/instance
>>>
```
使用flask shell命令打开的Python Shell自动包含程序上下文，并且已经导入了app实例：
```
>>> app
<Flask 'app'>
>>> app.name
'app'
```
# Flask扩展
扩展（extension）即使用Flask提供的API接口编写的Python库，可以为Flask程序添加各种各样的功能。大部分Flask扩展用来集成其他库，作为Flask和其他库之间的薄薄一层胶水。因为Flask扩展的编写有一些约定，所以初始化的过程大致相似。大部分扩展都会提供一个扩展类，实例化这个类，并传入我们创建的程序实例app作为参数，即可完成初始化过程。通常，扩展会在传入的程序实例上注册一些处理函数，并加载一些配置。
早期版本的Flask扩展使用flaskext.foo或flask.ext.something的形式导入，在实际使用中带来了许多问题，因此Flask官方推荐以flask_something形式导入扩展。

# 项目配置
在很多情况下，你需要设置程序的某些行为，这时你就需要使用配置变量。在Flask中，配置变量就是一些大写形式的Python变量，你也可以称之为配置参数或配置键。使用统一的配置变量可以避免在程序中以硬编码（hard coded）的形式设置程序。
在一个项目中，你会用到许多配置：Flask提供的配置，扩展提供的配置，还有程序特定的配置。和平时使用变量不同，这些配置变量都通过==Flask对象的app.config属性==作为统一的接口来设置和获取，它指向的Config类实际上是字典的子类，所以你可以像操作其他字典一样操作它。
附注:
Flask内置的配置可以访问Flask文档的配置章节（flask.pocoo.org/docs/latest/config/）查看，扩展提供的配置也可以在对应的文档中查看。Flask提供了很多种方式来加载配置。比如，你可以像在字典中添加一个键值对一样来设置一个配置：
```
app.config['ADMIN_NAME'] = 'Peter'
```
配置的名称必须是全大写形式，==小写的变量将不会被读取==。使用update（）方法则可以一次加载多个值：
```
app.config.update(
	TESTING=True,
	SECRET_KEY='_5#yF4Q8z\n\xec]/'
)
```
除此之外，你还可以把配置变量存储在单独的Python脚本、JSON格式的文件或是Python类中。和操作字典一样，读取一个配置就是从config字典里通过将配置变
量的名称作为键读取对应的值：
```
value = app.config['ADMIN_NAME']
```

# URL与endpoint (url_for())
使用Flask提供的url_for（）函数获取URL，当路由中定义的URL规则被修改时，这个函数总会返回正确的URL。调用url_for（）函数时，第一个参数为端点（endpoint）值。在Flask中，endpoint用来标记一个视图函数以及对应的URL规则。endpoint的默认值为视图函数的名称。比如，下面的视图函数：
```
@app.route('/')
def index():
return 'Hello Flask!'
```
这个路由的端点即视图函数的名称index，调用url_for（'index'）即可获取对应的URL，即“/”。如果URL含有动态部分，那么我们需要在url_for（）函数里传入相应的参数，以下面的视图函数为例：
```
@app.route('/hello/<name>')
def greet(name):
return 'Hello %s!' % name
```
这时使用url_for（'say_hello'，name='Jack'）得到的URL为“/hello/Jack”。
我们使用url_for（）函数生成的URL是相对URL（即内部URL），即URL中的path部分，比如“/hello”，不包含根URL。相对URL只能在程序内部使用。如果你想要生成供外部使用的绝对URL，可以在使用url_for（）函数时，将_external参数设为True，这会生成完整的URL.

# Flask命令
除了Flask内置的flask run等命令，我们也可以自定义命令。在虚拟环境安装Flask后，包含许多内置命令的flask脚本就可以使用了。在前面我们已经接触了很多flask命令，比如运行服务器的flask run，启动shell的flask shell。
通过创建任意一个函数，并为其添加app.cli.command（）装饰器，我们就可以注册一个flask命令。创建自定义命令:
```
@app.cli.command()
def hello():
	click.echo('Hello, Human!')
```
函数的名称即为命令名称，这里注册的命令即hello，你可以使用flask hello命令来触发函数。作为替代，你也可以在app.cli.command（）装饰器中传入参数来设置命令名称，比如app.cli.command（'say-hello'）会把命令名称设置为say-hello，完整的命令即flask say-hello。
借助click模块的echo（）函数，我们可以在命令行界面输出字符。命令函数的文档字符串则会作为帮助信息显示（flask hello--help）。在命令行下执行flask hello命令就会触发这个hello（）函数：

```
$ flask hello
Hello, Human!
```

在命令下执行flask--help可以查看Flask提供的命令帮助文档，我们
自定义的hello命令也会出现在输出的命令列表中，如下所示：

```
$ flask --help
Usage: flask [OPTIONS] COMMAND [ARGS]...
A general utility script for Flask applications.
...
```

# 模板与静态文件
我们需要模板（template）和静态文件（static file）来生成更加丰富的网页。模板即包含程序页面的HTML文件，静态文件则是需要在HTML文件中加载的CSS和JavaScript文件，以及图片、字体文件等资源文件。默认情况下，模板文件存放在项目根目录中的templates文件夹中，静态文件存放在static文件夹下，这两个文件夹需要和包含程序实例的模块处于同一
个目录下，对应的项目结构示例如下所示：
```
hello/
- templates/
- static/
- app.py
```
## CDN - Content delivery network
CDN指分布式服务器系统。服务商把你需要的资源存储在分布于不同地理位置的多个服务器，它会根据用户的地理位置来就近分配服务器提供服务（服务器越近，资源传送就越快）。使用CDN服务可以加快网页资源的加载速度，从而优化用户体验。对于开源的CSS和JavaScript库，CDN提供商通常会免费提供服务。

# Flask与MVC架构
你也许会困惑为什么用来处理请求并生成响应的函数被称为“视图函数（view function）”，其实这个命名并不合理。在Flask中，这个命名的约定来自Werkzeug，而Werkzeug中URL匹配的实现主要参考了Routes（一个URL匹配库），再往前追溯，Routes的实现又参考了Ruby
on Rails（http://rubyonrails.org/ ）。在Ruby on Rails中，术语views用来表示MVC（Model-View-Controller，模型-视图-控制器）架构中的View。
如果套用MVC架构的内容，那么Flask中视图函数的名称其实并不严谨，使用控制器函数（Controller Function）似乎更合适些，虽然它也附带处理用户界面。严格来说，Flask并不是MVC架构的框架，因为它没有内置数据模型支持。使用了app.route（）装饰器的函数仍被称为视图函数。如果想要使用Flask来编写一个MVC架构的程序，那么视图函数可以作为控制器（Controller），视图（View）则是使用Jinja2渲染的HTML模板，而模型（Model）可以使用SQLAlchemy来创建数据库模型。

# 请求响应循环
Web服务器接收请求，通过WSGI将HTTP
格式的请求数据转换成我们的Flask程序能够使用的Python数据。在程序
中，Flask根据请求的URL执行对应的视图函数，获取返回值生成响应。
响应依次经过WSGI转换生成HTTP响应，再经由Web服务器传递，最终
被发出请求的客户端接收。
![[Flask-2024316-2.png]]
请求解析和响应封装实际上大部分是由Werkzeug完成的，Flask子类化Werkzeug的请求（Request）和响应（Response）对象并添加了和程序相关的特定功能。
- request对象常用的属性和方法:
![[Flask-2024316-3.png]]
Werkzeug的MutliDict类是字典的子类，它主要实现了同一个键对应多个值的情况。比如一个文件上传字段可能会接收多个文件。这时就可以通过getlist（）方法来获取文件对象列表。而ImmutableMultiDict类继承了MutliDict类，但其值不可更改。

当你访问http://localhost:5000/hello?name=Grey 时，页面加载后会显示“Hello，
Grey！”。这说明处理这个URL的视图函数从查询字符串中获取了查询参数name的值:
```
from flask import Flask, request
app = Flask(__name__)
@app.route('/hello')
def hello():
name = request.args.get('name', 'Flask') # 获取查询参数name的值
return '<h1>Hello, %s!<h1>' % name # 插入到返回值中
```
注意
和普通的字典类型不同，当我们从request对象的类型为MutliDict或ImmutableMultiDict的属性（比如files、form、args）中直接使用键作为索引获取数据时（比如request.args['name']），如果没有对应的键，那么会返回HTTP 400错误响应（Bad Request，表示请求无效），而不是抛出KeyError异常。为了避免这个错误，我们应该使用get（）方法获取数据，如果没有对应的值则返回None；get（）方法的第二个参数可以设置默认值，比如requset.args.get（'name'，'Human'）。

# HTTP请求
## 在Flask中处理请求
在Flask中，我们需要让请求的URL匹配对应的视图函数，视图函数返回值就是URL对应的资源。
### 1.路由匹配
为了便于将请求分发到对应的视图函数，程序实例中存储了一个路由表（app.url_map），其中定义了URL规则和视图函数的映射关系。当请求发来后，Flask会根据请求报文中的URL（path部分）来尝试与这个表中的所有URL规则进行匹配，调用匹配成功的视图函数。如果没有找到匹配的URL规则，说明程序中没有处理这个URL的视图函数，Flask会自动返回404错误响应（Not Found，表示资源未找到）。当请求的URL与某个视图函数的URL规则匹配成功时，对应的视图函数就会被调用。使用flask routes命令可以查看程序中定义的所有路
由，这个列表由app.url_map解析得到：
```
$ flask routes
Endpoint Methods Rule
-------- ------- -----------------------
hello GET /hello
go_back GET /goback/<int:age>
hi GET /hi
...
static GET /static/<path:filename>
```
### 2.设置监听的HTTP方法
我们可以在app.route（）装饰器中使用methods参数传入一个包含监听的HTTP方法的可迭代对象。比如，下面的视图函数同时监听GET请求和POST请求：
```
@app.route('/hello', methods=['GET', 'POST'])
def hello():
return '<h1>Hello, Flask!</h1>'
```
### 3.URL处理
URL规则中的变量<int:year>表示为year变量添加了一个int转换器，Flask在解析这个URL变量时会将其转换为整型。URL中的变量部分默认类型为字符串，但Flask提供了一些转换器可以在URL规则里使用:
![[Flask-2024316-4.png]]

```
@app.route('goback/<int:year>')
def go_back(year):
return '<p>Welcome to %d!</p>' % (2018 - year)
```
在用法上唯一特别的是any转换器，你需要在转换器后添加括号来给出可选值，即“<any（value1，value2，...）：变量名>”，比如：
```
@app.route('/colors/<any(blue, white, red):color>')
def three_colors(color):
return '<p>Love is patient and kind. Love is not jealous or boastful or proud or rude.</p>'
```

当你在浏览器中访问http://localhost:5000/colors/ 时，如果将<color>部分替换为any转换器中设置的可选值以外的任意字符，均会获得404错误响应。

如果你想在any转换器中传入一个预先定义的列表，可以通过格式化字符串的方式（使用%或是format（）函数）来构建URL规则字符串，比如：
```
colors = ['blue', 'white', 'red']
@app.route('/colors/<any(%s):color>' % str(colors)[1:-1])
...
```
# 请求钩子 (request hooks)
有时我们需要对请求进行预处理（preprocessing）和后处理（postprocessing），这时可以使用Flask提供的一些请求钩子（Hook），它们可以用来注册在请求处理的不同阶段执行的处理函数（或称为回调函数，即Callback）。这些请求钩子使用装饰器实现，通过程序实例app调用，用法很简单：以before_request钩子（请求之前）为例，当你对一个函数附加了app.before_request装饰器后，就会将这个函数注册为before_request处理函数，每次执行请求前都会触发所有before_request处理函数。Flask默认实现的五种请求钩子:
![[Flask-2024316-5.png]]

这些钩子使用起来和app.route（）装饰器基本相同，每个钩子可以注册任意多个处理函数，函数名并不是必须和钩子名称相同，下面是一个基本示例：

```
@app.before_request
def do_something():
pass # 这里的代码会在每个请求处理前执行
```
下面是请求钩子的一些常见应用场景：
- before_first_request：在玩具程序中，运行程序前我们需要进行一些程序的初始化操作，比如创建数据库表，添加管理员用户。这些工作可以放到使用before_first_request装饰器注册的函数中。
- before_request：比如网站上要记录用户最后在线的时间，可以通过用户最后发送的请求时间来实现。为了避免在每个视图函数都添加更新在线时间的代码，我们可以仅在使用before_request钩子注册的函数中调用这段代码。
- after_request：我们经常在视图函数中进行数据库操作，比如更新、插入等，之后需要将更改提交到数据库中。提交更改的代码就可以放到after_request钩子注册的函数中。

另一种常见的应用是建立数据库连接，通常会有多个视图函数需要建立和关闭数据库连接，这些操作基本相同。一个理想的解决方法是在请求之前（before_request）建立连接，在请求之后（teardown_request）关闭连接。通过在使用相应的请求钩子注册的函数中添加代码就可以实现。

after_request钩子和after_this_request钩子必须接收一个响应类对象作为参数，并且返回同一个或更新后的响应对象。

# HTTP响应
<center>常见的HTTP状态码</center>

![[Flask-2024316-6.png]]

## 在Flask中生成响应
响应在Flask中使用Response对象表示，响应报文中的大部分内容由服务器处理，大多数情况下，我们只负责返回主体内容。Flask会调用make_response（）方法将视图函数返回值转换为响应对象。
完整地说，视图函数可以返回最多由三个元素组成的元组：响应主体、状态码、首部字段。其中首部字段可以为字典，或是两元素元组组成的列表。
比如，普通的响应可以只包含主体内容：
```
@app.route('/hello')
def hello():
...
return '<h1>Hello, Flask!</h1>'
```
默认的状态码为200，下面指定了不同的状态码：

```
@app.route('/hello')
def hello():
...
return '<h1>Hello, Flask!</h1>', 201
```
有时你会想附加或修改某个首部字段。比如，要生成状态码为3XX的重定向响应，需要将首部中的Location字段设置为重定向的目标URL：

```
@app.route('/hello')
def hello():
...
return '', 302, {'Location', 'http://www.example.com'}
```
现在访问http://localhost:5000/hello，会重定向到http://www.example.com 。
### 1.重定向
对于重定向这一类特殊响应，Flask提供了一些辅助函数。除了像前面那样手动生成302响应，我们可以使用Flask提供的redirect（）函数来生成重定向响应，重定向的目标URL作为第一个参数。前面的例子可以简化为：

```
from flask import Flask, redirect
# ...
@app.route('/hello')
def hello():
return redirect('http://www.example.com')
```
使用redirect（）函数时，默认的状态码为302，即临时重定向。如果你想修改状态码，可以在redirect（）函数中作为第二个参数或使用code关键字传入。
如果要在程序内重定向到其他视图，那么只需在redirect（）函数中使用url_for（）函数生成目标URL即可:

```
from flask import Flask, redirect, url_for
...
@app.route('/hi')
def hi():
...
return redierct(url_for('hello')) # 重定向到/hello

@app.route('/hello')
def hello():
...
```
### 2.错误响应
大多数情况下，Flask会自动处理常见的错误响应。HTTP错误对应的异常类在Werkzeug的werkzeug.exceptions模块中定义，抛出这些异常即可返回对应的错误响应。如果你想手动返回错误响应，更方便的方法是使用Flask提供的abort（）函数。在abort（）函数中传入状态码即可返回对应的错误响应:

```
from flask import Flask, abort
...
@app.route('/404')
def not_found():
abort(404)
```

提示: 
abort（）函数前不需要使用return语句，但一旦abort（）函数被调用，abort（）函数之后的代码将不会被执行。

附注:
虽然我们有必要返回正确的状态码，但这不是必须的。比如，当某个用户没有权限访问某个资源时，返回404错误要比403错误更加友好。

### 3.响应格式 (Content-Type)
在HTTP响应中，数据可以通过多种格式传输。大多数情况下，我们会使用HTML格式，这也是==Flask中的默认设置==。在特定的情况下，我们也会使用其他格式。不同的响应数据格式需要设置不同的MIME类型，==MIME类型在首部的Content-Type字段中定义==，以默认的HTML类型为例：

```
Content-Type: text/html; charset=utf-8
```

附注:
MIME类型（又称为 Multipurpose Internet Mail Extensions, media type或content type）是一种用来标识文件类型的机制，它与文件扩展名相对应，可以让客户端区分不同的内容类型，并执行不同的操作。一般的格式为“类型名/子类型名”，其中的子类型名一般为文件扩展名。比如，HTML的MIME类型为“text/html”，png图片的MIME类型为“image/png”。完整的标准MIME类型列表可以在这里看到：https://www.iana.org/assignments/media-types/media-types.xhtml.

如果你想使用其他MIME类型，可以通过Flask提供的make_response（）方法生成响应对象，传入响应的主体作为参数，然后使用响应对象的mimetype属性设置MIME类型，比如：

```
from flask import make_response
@app.route('/foo')
def foo():
response = make_response('Hello, World!')
response.mimetype = 'text/plain'
return response
```

你也可以直接设置首部字段，比如response.headers['Content-Type']='text/xml；charset=utf-8'。但操作mimetype属性更加方便，而且不用设置字符集（charset）选项。

#### 1.纯文本
MIME类型：text/plain
#### 2.HTML
MIME类型：text/html
#### 3.XML
MIME类型：application/xml
XML是一种简单灵活的文本格式，被设计用来存储和交换数据。XML的出现主要就是为了弥补HTML的不足：对于仅仅需要数据的请求来说，HTML提供的信息太过丰富了，而且不易
于重用。XML和HTML一样都是标记性语言，使用标签来定义文本，但HTML中的标签用于显示内容，而XML中的标签只用于定义数据。XML一般作为AJAX请求的响应格式，或是Web API的响应格式。
#### 4.JSON
MIME类型：application/json

JSON是一种流行的、轻量的数据交换格式。它的出现又弥补了XML的诸多不足：XML有较高的重用性，但XML相对于其他文档格式来说体积稍大，处理和解析的速度较慢。JSON轻量，简洁，容易阅读和解析，而且能和Web默认的客户端语言JavaScript更好地兼容。JSON
的结构基于“键值对的集合”和“有序的值列表”，这两种数据结构类似Python中的字典（dictionary）和列表（list）。正是因为这种通用的数据结构，使得JSON在同样基于这些结构的编程语言之间交换成为可能。

Flask通过引入Python标准库中的json模块（或simplejson，如果可用）为程序提供了JSON支持。你可以直接从Flask中导入json对象，然后调用dumps（）方法将字典、列表或元组序列化（serialize）为JSON字符串，再使用前面介绍的方法修改MIME类型，即可返回JSON响:

```
from flask import Flask, make_response, json
...
@app.route('/foo')
def foo():
data = {
'name':'Grey Li',
'gender':'male'
}
response = make_response(json.dumps(data))
response.mimetype = 'application/json'
return response
```
不过我们一般并不直接使用json模块的dumps（）、load（）等方法，因为Flask通过包装这些方法提供了更方便的jsonify（）函数。借助jsonify（）函数，我们仅需要传入数据或参数，它会对我们传入的参数进行序列化，转换成JSON字符串作为响应的主体，然后生成一个响应对象，并且设置正确的MIME类型。使用jsonify函数可以将前面的例子简化为这种形式：
```
from flask import jsonify
@app.route('/foo')
def foo():
return jsonify(name='Grey Li', gender='male')
```
jsonify（）函数接收多种形式的参数。你既可以传入普通参数，也可以传入关键字参数。如果你想要更直观一点，也可以像使用dumps（）方法一样传入字典、列表或元组，比如：
```
from flask import jsonify
@app.route('/foo')
def foo():
return jsonify({name: 'Grey Li', gender: 'male'})
```
上面两种形式的返回值是相同的，都会生成下面的JSON字符串：

```
'{"gender": "male", "name": "Grey Li"}'
```

另外，jsonify（）函数默认生成200响应，你也可以通过附加状态码来自定义响应类型，比如：

```
@app.route('/foo')
def foo():
return jsonify(message='Error!'), 500
```

提示:
Flask在获取请求中的JSON数据上也有很方便的解决方案，具体可以参考我们在Request对象小节介绍的request.get_json（）方法和request.json属性。

## Cookie
HTTP是无状态（stateless）协议。也就是说，在一次请求响应结束后，服务器不会留下任何关于对方状态的信息。但是对于某些Web程序来说，客户端的某些信息又必须被记住。为了解决这类问题，就有了Cookie技术。Cookie技术通过在请求和响应报文中添加Cookie数据来保存客户端的状态信息。
附注:
Cookie指Web服务器为了存储某些数据（比如用户信息）而保存在浏览器上的小型文本数据。浏览器会在一定时间内保存它，并在下一次
向同一个服务器发送请求时附带这些数据。Cookie通常被用来进行用户会话管理，保存用户的个性化信息，以及记录和收集用户浏览数据以用来分析用户行为等。

在Flask中，如果想要在响应中添加一个cookie，最方便的方法是使用Response类提供的set_cookie（）方法。要使用这个方法，我们需要先使用make_response（）方法手动生成一个响应对象，传入响应主体作为参数。这个响应对象默认实例化内置的Response类。
<center>Response类常用的属性和方法</center>

![[Flask-2024316-7.png]]

附注：
- Respone类同样拥有和Request类相同的get_json（）方法、is_json（）方法以及json属性。

- set_cookie（）方法支持多个参数来设置Cookie的选项 ![[Flask-2024317-1.png]]
set_cookie视图用来设置cookie，它会将URL中的name变量的值设置到名为name的cookie里：

```
from flask import Flask, make_response
...
@app.route('/set/<name>')
def set_cookie(name):
response = make_response(redirect(url_for('hello')))
response.set_cookie('name', name)
return response
```

在这个make_response（）函数中，我们传入的是使用redirect（）函数生成的重定向响应。set_cookie视图会在生成的响应报文首部中创建一个Set-Cookie字段，即“Set-Cookie：name=Grey；Path=/”。因为过期时间使用默认值，所以会在浏览会话结束时（关闭浏览器）过期。

在Flask中，Cookie可以通过请求对象的cookies属性读取。在修改后
的hello视图中，如果没有从查询参数中获取到name的值，就从Cookie中
寻找：

```
from flask import Flask, request
@app.route('/')
@app.route('/hello')
def hello():
	name = request.args.get('name')
	if name is None:
		name = request.cookies.get('name', 'Human') # 从Cookie中获取name值
	return '<h1>Hello, %s</h1>' % name
```
Flask提供了session对象用来将Cookie数据加密储存。在Flask中，session对象用来加密Cookie。默认情况下，它会把数据存储在浏览器上一个名为session的cookie里。
### 1.设置程序密钥
session通过密钥对数据进行签名以加密数据，因此，我们得先设置一个密钥。这里的密钥就是一个具有一定复杂度和随机性的字符串，比如“Drmhze6EPcv0fN_81Bj-nA”。
程序的密钥可以通过Flask.secret_key属性或配置变量SECRET_KEY设置，比如：
```
app.secret_key = 'secret string'
```
更安全的做法是把密钥写进系统环境变量（在命令行中使用export或set命令），或是保存在.env文件中：
```
SECRET_KEY=secret string
```
然后在程序脚本中使用os模块提供的getenv（）方法获取：
```
import os
# ...
app.secret_key = os.getenv('SECRET_KEY', 'secret string')
```
我们可以在getenv（）方法中添加第二个参数，作为没有获取到对
应环境变量时使用的默认值。

### 2.模拟用户认证
下面我们会使用session模拟用户的认证功能。登入用户的login：

```
from flask import redirect, session, url_for
@app.route('/login')
def login():
	session['logged_in'] = True # 写入session
	return redirect(url_for('hello'))
```
session对象可以像字典一样操作，我们向session中添加一个logged-in cookie，将它的值设为True，表示用户已认证。
当我们使用session对象添加cookie时，数据会使用程序的密钥对其进行签名，加密后的数据存储在一块名为session的cookie里。你可以在Cookie的Content部分看到对应的加密处理后生成的session值。使用session对象存储的Cookie，用户可以看到其加密后的值，但无法修改它。因为session中的内容使用密钥进行签名，一旦数据被修改，签名的值也会变化。这样在读取时，就会验证失败，对应的session值也会随之失效。所以，除非用户知道密钥，否则无法对sessioncookie的值进行修改。![[Flask-2024317-2.png]]

当支持用户登录后，我们就可以根据用户的认证状态分别显示不同的内容。在login视图的最后，我们将程序重定向到hello视图：

```
from flask import request, session
@app.route('/')
@app.route('/hello')
def hello():
	name = request.args.get('name')
	if name is None:
		name = request.cookies.get('name', 'Human')
		response = '<h1>Hello, %s!</h1>' % name
		# 根据用户认证状态返回不同的内容
		 if 'logged_in' in session:
			response += '[Authenticated]'
		else:
			response += '[Not Authenticated]'
		return response
```

session中的数据可以像字典一样通过键读取，或是使用get（）方法。这里我们只是判断session中是否包含logged_in键，如果有则表示用户已经登录。通过判断用户的认证状态，我们在返回的响应中添加一行表示认证状态的信息：如果用户已经登录，显示[Authenticated]；否则显示[Not authenticated]。

登出用户的logout视图也非常简单，登出账户对应的实际操作其实就是把代表用户认证的logged-in cookie删除，这通过session对象的pop方法实现：

```
from flask import session
@app.route('/logout')
def logout():
	if 'logged_in' in session:
		session.pop('logged_in')
	return redirect(url_for('hello'))
```
提示:
默认情况下，session cookie会在用户关闭浏览器时删除。通过将session.permanent属性设为True可以将session的有效期延长为Flask.permanent_session_lifetime属性值对应的datetime.timedelta对象，也可通过配置变量PERMANENT_SESSION_LIFETIME设置，默认为31天。

# Flask context
Flask中有两种上下文，程序上下文（application context）和请求上下文（request context）。

## Context全局变量
我们可以直接从Flask导入一个全局的request对象，然后在视图函数里直接调用request的属性获取数据。Flask会在每个请求产生后自动激活当前请求的上下文，激活请求上下文后，request被临时设为全局可访问。而当每个请求结束后，Flask就销毁对应的请求上下文。
我们在前面说request是全局对象，但这里的“全局”并不是实际意义上的全局。我们可以把这些变量理解为动态的全局变量。
在多线程服务器中，在同一时间可能会有多个请求在处理。假设有三个客户端同时向服务器发送请求，这时每个请求都有各自不同的请求报文，所以请求对象也必然是不同的。因此，请求对象只在各自的线程内是全局的。Flask通过本地线程（thread local）技术将请求对象在特定的线程和请求中全局可访问。
Flask提供了四个上下文全局变量，这四个变量都是代理对象（proxy），即指向真实对象的代理。一般情况下，我们不需要太关注其中的区别。在某些特定的情况下，如果你需要获取原始对象，可以对代理对象调用_get_current_object（）方法获取被代理的真实对象。
![[Flask-2024317-3.png]]
- current_app
在不同的视图函数中，request对象都表示和视图函数对应的请求，也就是当前请求（current request）。而程序也会有多个程序实例的情况，为了能获取对应的程序实例，而不是固定的某一个程序实例，我们就需要使用current_app变量。
- g 
因为g存储在程序上下文中，而程序上下文会随着每一个请求的进入而激活，随着每一个请求的处理完毕而销毁，所以每次请求都会重设这个值。我们通常会使用它结合请求钩子来保存每个请求处理前所需要的全局变量，比如当前登入的用户对象，数据库连接等。借助g我们可以将这个操作移动到before_request处理函数中执行，然后保存到g的任意属性上：

```
from flask import g
@app.before_request
def get_name():
	g.name = request.args.get('name')
```
设置这个函数后，在其他视图中可以直接使用g.name获取对应的值。另外，g也支持使用类似字典的get（）、pop（）以及setdefault（）方法进行操作。

## 激活上下文
在下面这些情况下，Flask会自动帮我们激活程序上下文：
- 当我们使用flask run命令启动程序时。
- ~~使用旧的app.run（）方法启动程序时。~~
- 执行使用@app.cli.command（）装饰器注册的flask命令时。
- 使用flask shell命令启动Python Shell时。

当请求进入时，Flask会自动激活请求上下文，这时我们可以使用request和session变量。另外，当请求上下文被激活时，程序上下文也被自动激活。当请求处理完毕后，请求上下文和程序上下文也会自动销毁。也就是说，在请求处理时这两者拥有相同的生命周期。结合Python的代码执行机制理解，这也就意味着，==我们可以在视图函数中或在视图函数内调用的函数/方法中使用所有上下文全局变量==。
在使用flask shell命令打开的Python Shell中，或是自定义的flask命令函数中，我们可以使用current_app和g变量，也可以手动激活请求上下文来使用request和session。
如果我们在没有激活相关上下文时使用这些变量，Flask就会抛出RuntimeError异常：“RuntimeError：Working outside of applicationcontext.”或是“RuntimeError：Working outside of request context.”。
==提示==
同样依赖于上下文的还有url_for（）、jsonify（）等函数，所以你也只能在视图函数中使用它们。其中jsonify（）函数内部调用中使用了current_app变量，而url_for（）则需要依赖请求上下文才可以正常运行。
如果你需要在没有激活上下文的情况下使用这些变量，可以手动激活上下文。比如，下面是一个普通的Python shell，通过python命令打开。程序上下文对象使用app.app_context（）获取，我们可以使用with语句执行上下文操作：

```
>>> from app import app
>>> from flask import current_app
>>> with app.app_context():
... current_app.name
'app'
```

或是显式地使用push（）方法推送（激活）上下文，在执行完相关操作时使用pop（）方法销毁上下文：

```
>>> from app import app
>>> from flask import current_app
>>> app_ctx = app.app_context()
>>> app_ctx.push()
>>> current_app.name
'app'
>>> app_ctx.pop()
```
而请求上下文可以通过test_request_context（）方法临时创建：

```
>>> from app import app
>>> from flask import request
>>> with app.test_request_context('/hello'):
... request.method
'GET'
```

同样的，这里也可以使用push（）和pop（）方法显式地推送和销毁请求上下文。

## 上下文钩子
Flask也为context(上下文)提供了一个teardown_appcontext钩子，使用它注册的回调函数会
在程序上下文被销毁时调用，而且通常也会在请求上下文被销毁时调用。比如，你需要在每个请求处理结束后销毁数据库连接：

```
@app.teardown_appcontext
def teardown_db(exception):
...
db.close()
```

使用app.teardown_appcontext装饰器注册的回调函数需要接收异常对象作为参数，当请求被正常处理时这个参数值将是None，这个函数的返回值将被忽略。

# HTTP进阶
## 重定向回上一个页面
### 1.获取上一个页面的URL
要重定向回上一个页面，最关键的是获取上一个页面的URL。上一个页面的URL一般可以通过两种方式获取：
#### 1.HTTP referer
HTTP referer（起源为referrer在HTTP规范中的错误拼写）是一个用来记录请求发源地址的HTTP首部字段（HTTP_REFERER），即访问来源。当用户在某个站点单击链接，浏览器向新链接所在的服务器发起请求，请求的数据中包含的HTTP_REFERER字段记录了用户所在的原站
点URL。这个值通常会用来追踪用户，比如记录用户进入程序的外部站点，以此来更有针对性地进行营销。在Flask中，referer的值可以通过请求对象的referrer属性获取，即request.referrer（正确拼写形式）。现在do_something视图的返回值可以这样编写：
```
return redirect(request.referrer)
```
但是在很多种情况下，referrer字段会是空值，比如用户在浏览器的地址栏输入URL，或是用户出于保护隐私的考虑使用了防火墙软件或使用浏览器设置自动清除或修改了referrer字段。我们需要添加一个备选项：
```
return redirect(request.referrer or url_for('hello'))
```
#### 2.查询参数
除了自动从referrer获取，另一种更常见的方式是在URL中手动加入包含当前页面URL的查询参数，这个查询参数一般命名为next。比如，下面在foo和bar视图的返回值中的URL后添加next参数：

```
from flask import request
@app.route('/foo')
def foo():
	return '<h1>Foo page</h1><a href="%s">Do something and redirect</a>' % url_for('do_something', next=request.full_path)
@app.route('/bar')
def bar():
	return '<h1>Bar page</h1><a href="%s">Do something and redirect</a>' % url_for('do_something', next=request.full_path)
```
在程序内部只需要使用相对URL，所以这里使用request.full_path获取当前页面的完整路径。在do_something视图中，我们获取这个next值，然后重定向到对应的路径：

```
return redirect(request.args.get('next'))
```

用户在浏览器的地址栏直接访问时可以轻易地修改查询参数，为了避免next参数为空的情况，我们也要添加备选项，如果为空就重定向到hello视图：

```
return redirect(request.args.get('next', url_for('hello')))
```

为了覆盖更全面，我们可以将这两种方式搭配起来一起使用：首先获取next参数，如果为空就尝试获取referer，如果仍然为空，那么就重定向到默认的hello视图。因为在不同视图执行这部分操作的代码完全相同，我们可以创建一个通用的redirect_back（）函数：

```
def redirect_back(default='hello', **kwargs):
for target in request.args.get('next'), request.referrer:
if target:
return redirect(target)
return redirect(url_for(default, **kwargs))
```

通过设置默认值，我们可以在referer和next为空的情况下重定向到默认的视图。在do_something视图中使用这个函数的示例如下所示：

```
@app.route('/do_something_and_redirect')
def do_something():
	 # do something
	return redirect_back()
```
## 对URL进行安全验证
鉴于referer和next容易被篡改的特性，如果我们不对这些值进行验证，则会形成开放重定向（Open Redirect）漏洞。以URL中的next参数为例，next变量以查询字符串的方式写在URL
里，因此任何人都可以发给某个用户一个包含next变量指向任何站点的链接。举个简单的例子，如果你访问下面的URL：http://localhost:5000/do-something?next=http://helloflask.com
程序会被重定向到http://helloflask.com 。也就是说，如果我们不验证next变量指向的URL地址是否属于我们的应用内，那么程序很容易就会被重定向到外部地址。确保URL安全的关键就是判断URL是否属于程序内部，我们创建了一个URL验证函数is_safe_url（），用来验证next变量值是否属于程序内部URL。

```
from urlparse import urlparse, urljoin # Python3需要从urllib.parse导入
from flask import request
def is_safe_url(target):
	ref_url = urlparse(request.host_url)
	test_url = urlparse(urljoin(request.host_url, target))
	return test_url.scheme in ('http', 'https') and \
		ref_url.netloc == test_url.netloc
```

==注意==
如果你使用Python3，那么这里需要从urllib.parse模块导入urlparse和urljoin函数。示例程序仓库中实际的代码做了兼容性处理。这个函数接收目标URL作为参数，并通过request.host_url获取程序内的主机URL，然后使用urljoin（）函数将目标URL转换为绝对URL。接着，分别使用urlparse模块提供的urlparse（）函数解析两个URL，最后对目标URL的URL模式和主机地址进行验证，确保只有属于程序内部的URL才会被返回。在执行重定向回上一个页面的redirect_back（）函数中，我们使用is_safe_url（）验证next和referer的值：
```
def redirect_back(default='hello', **kwargs):
	for target in request.args.get('next'), request.referrer:
	if not target:
		continue
	if is_safe_url(target):
		return redirect(target)
	return redirect(url_for(default, **kwargs))
```

## HTTP服务器端推送 (server push)

## Web安全防范
### 1.注入攻击
### 2.XSS攻击
XSS（Cross-Site Scripting，跨站脚本）攻击历史悠久，最远可以追溯到90年代，但至今仍然是危害范围非常广的攻击方式。在OWASPTOP 10中排名第7。
==附注==
Cross-Site Scripting的缩写本应是CSS，但是为了避免和CascadingStyle Sheets的缩写产生冲突，所以将Cross（即交叉）使用交叉形状的X表示。
- 攻击原理
XSS是注入攻击的一种，攻击者通过将代码注入被攻击者的网站中，用户一旦访问网页便会执行被注入的恶意脚本。XSS攻击主要分为反射型XSS攻击（Reflected XSS Attack）和存储型XSS攻击（Stored XSSAttack）两类。
- 攻击示例
	- 反射型XSS又称为非持久型XSS（Non-Persistent XSS）。当某个站点存在XSS漏洞时，这种攻击会通过URL注入攻击脚本，只有当用户访问这个URL时才会执行攻击脚本。
	- 存储型XSS也被称为持久型XSS（persistent XSS），这种类型的XSS攻击更常见，危害也更大。它和反射型XSS类似，不过会把攻击代码储存到数据库中，任何用户访问包含攻击代码的页面都会被殃及。比如，某个网站通过表单接收用户的留言，如果服务器接收数据后未经处理就存储到数据库中，那么用户可以在留言中插入任意JavaScript代码。
- 主要防范措施
	- HTML转义
		防范XSS攻击最主要的方法是对用户输入的内容进行HTML转义，转义后可以确保用户输入的内容在浏览器中作为文本显示，而不是作为代码解析。
	- 验证用户输入
		XSS攻击可以在任何用户可定制内容的地方进行，例如图片引用、自定义链接。仅仅转义HTML中的特殊字符并不能完全规避XSS攻击，因为在某些HTML属性中，使用普通的字符也可以插入JavaScript代码。除了转义用户输入外，我们还需要对用户的输入数据进行类型验证。在所有接收用户输入的地方做好验证工作。

### 3.CSRF攻击
CSRF（Cross Site Request Forgery，跨站请求伪造）是一种近年来才逐渐被大众了解的网络攻击方式，又被称为One-Click Attack或SessionRiding。在OWASP上一次（2013）的TOP 10 Web程序安全风险中，它位列第8。随着大部分程序的完善，各种框架都内置了对CSRF保护的支持，但目前仍有5%的程序受到威胁。
- 攻击原理
CSRF攻击的大致方式如下：某用户登录了A网站，认证信息保存在cookie中。当用户访问攻击者创建的B网站时，攻击者通过在B网站发送一个伪造的请求提交到A网站服务器上，让A网站服务器误以为请求来自于自己的网站，于是执行相应的操作，该用户的信息便遭到了篡改。总结起来就是，攻击者利用用户在浏览器中保存的认证信息，向对应的站点发送伪造请求。在前面学习cookie时，我们介绍过用户认证通过保存在cookie中的数据实现。在发送请求时，只要浏览器中保存了对应的cookie，服务器端就会认为用户已经处于登录状态，而攻击者正是利用了这一机制。
- 攻击示例
假设我们网站是一个社交网站（example.com），简称网站A；攻击者的网站可以是任意类型的网站，简称网站B。在我们的网站中，删除账户的操作通过GET请求执行，由使用下面的delete_account视图处理：

```
@app.route('/account/delete')
def delete_account():
	if not current_user.authenticated:
		abort(401)
	current_user.delete()
	return 'Deleted!'
```
	当用户登录后，只要访问http://example.com/account/delete 就会删除账户。那么在攻击者的网站上，只需要创建一个显示图片的img标签，其中的src属性加入删除账户的URL：
```
<img src="http://example.com/account/delete">
```
	当用户访问B网站时，浏览器在解析网页时会自动向img标签的src属性中的地址发起请求。此时你在A网站的登录信息保存在cookie中，因此，仅仅是访问B网站的页面就会让你的账户被删除掉。当然，现实中很少有网站会使用GET请求来执行包含数据更改的敏感操作，这里只是一个示例。现在，假设我们吸取了教训，改用POST请求提交删除账户的请求。尽管如此，攻击者只需要在B网站中内嵌一个隐藏表单，然后设置在页面加载后执行提交表单的JavaScript函数，攻击仍然会在用户访问B网站时发起。
	虽然CSRF攻击看起来非常可怕，但我们仍然可以采取一些措施来进行防御。下面我们来介绍防范CSRF攻击的两种主要方式。
- 主要防范措施
	- 正确使用HTTP方法
	- CSRF令牌校验
	当处理非GET请求时，要想避免CSRF攻击，关键在于判断请求是否来自自己的网站。在前面我们曾经介绍过使用HTTP referer获取请求来源，理论上说，通过referer可以判断源站点从而避免CSRF攻击，但因为referer很容易被修改和伪造，所以不能作为主要的防御措施。
	除了在表单中加入验证码外，一般的做法是通过在客户端页面中加入伪随机数来防御CSRF攻击，这个伪随机数通常被称为CSRF令牌（token）。
	在HTML中，POST方法的请求通过表单创建。我们把在服务器端创建的伪随机数（CSRF令牌）添加到表单中的隐藏字段里和session变量（即签名cookie）中，当用户提交表单时，这个令牌会和表单数据一起提交。在服务器端处理POST请求时，我们会对表单中的令牌值进行
	验证，如果表单中的令牌值和session中的令牌值相同，那么就说明请求发自自己的网站。因为CSRF令牌在用户向包含表单的页面发起GET请求时创建，并且在一定时间内过期，一般情况下攻击者无法获取到这个令牌值，所以我们可以有效地区分出请求的来源是否安全。
	我们通常会使用扩展实现CSRF令牌的创建和验证工作，比如Flask-SeaSurf（https://github.com/maxcountryman/flask-seasurf ）、Flask-WTF内置的CSRFProtect（https://github.com/lepture/flask-wtf ）等。

# 模板 (templates)
模板引擎的作用就是读取并执行模板中的特殊语法标记，并根据传入的数据将变量替换为实际值，输出最终的HTML页面，这个过程被称为渲染（rendering）。Flask默认使用的模板引擎是Jinja2，它是一个功能齐全的Python模板引擎，除了设置变量，还允许我们在模板中添加if判断，执行for迭代，调用函数等，以各种方式控制模板的输出。对于Jinja2来说，模板可以是任何格式的纯文本文件，比如HTML、XML、CSV、LaTeX等。

## 基本用法
data model:

```
user = {
	'username': 'Grey Li',
	'bio': 'A boy who loves movies and music.',
}
movies = [
	{'name': 'My Neighbor Totoro', 'year': '1988'},
	{'name': 'Three Colours trilogy', 'year': '1993'},
	{'name': 'Forrest Gump', 'year': '1994'},
	{'name': 'Perfect Blue', 'year': '1997'},
	{'name': 'The Matrix', 'year': '1999'},
	{'name': 'Memento', 'year': '2000'},
	{'name': 'The Bucket list', 'year': '2007'},
	{'name': 'Black Swan', 'year': '2010'},
	{'name': 'Gone Girl', 'year': '2014'},
	{'name': 'CoCo', 'year': '2017'},
]
```
我们在templates目录下创建一个watchlist.html作为模板文件，然后使用Jinja2支持的语法在模板中操作这些变量:
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>{{ user.username }}'s Watchlist</title>
</head>
<body>
	<a href="{{ url_for('index') }}">&larr; Return</a>
	<h2>{{ user.username }}</h2>
	{% if user.bio %}
		<i>{{ user.bio }}</i>
	{% else %}
		<i>This user has not provided a bio.</i>
	{% endif %}
	{# 下面是电影清单（这是注释） #}
	<h5>{{ user.username }}'s Watchlist ({{ movies|length }}):</h5>
	<ul>
		{% for movie in movies %}
			<li>{{ movie.name }} - {{ movie.year }}</li>
		{% endfor %}
	</ul>
</body>
</html>
```
Jinja2里常见的三种定界符：
- （1）语句 
	比如if判断、for循环等：
	`{% ... %}`
- （2）表达式
	比如字符串、变量、函数调用等：
	`{{ ... }}`
（3）注释
	`{# ... #}`
另外，在模板中，==Jinja2支持使用“.”获取变量的属性，比如user字典中的username键值通过“.”获取，即user.username，在效果上等同于user['username']==。

## 模板语法
我们可以在模板中使用Python语句和表达式来操作数据的输出。但需要注意的是，Jinja2并不支持所有Python语法。而且出于效率和代码组织等方面的考虑，我们应该适度使用模板，仅把和输出控制有关的逻辑操作放到模板中。
Jinja2允许你在模板中使用大部分Python对象，比如
- 字符串、
- 列表、
- 字典、
- 元组、
- 整型、
- 浮点型、
- 布尔值。
它支持基本的
- 运算符号（+、-、*、/等）、
- 比较符号（比如==、！=等）、
- 逻辑符号（and、or、not和括号）
- in、
- is、
- None
- 布尔值（True、False）。

Jinja2提供了多种控制结构来控制模板的输出，其中for和if是最常用的两种。在Jinja2里，语句使用=={%...%}==标识，尤其需要注意的是，在语句结束的地方，我们必须添加结束标签：

```
{% if user.bio %}
	<i>{{ user.bio }}</i>
{% else %}
	<i>This user has not provided a bio.</i>
{% endif %}
```
和在Python里一样，for语句用来迭代一个序列：
```
<ul>
	{% for movie in movies %}
		<li>{{ movie.name }} - {{ movie.year }}</li>
	{% endfor %}
</ul>
```
在for循环内，Jinja2提供了多个特殊变量，常用的循环变量:
![[Flask-2024317-4.png]]
完整的for循环变量列表请访问http://jinja.pocoo.org/docs/2.10/templates/#for

## 渲染模板
渲染一个模板，就是执行模板中的代码，并传入所有在模板中使用的变量，渲染后的结果就是我们要返回给客户端的HTML响应。在视图函数中渲染模板时，我们并不直接使用Jinja2提供的函数，而是使用Flask提供的渲染函数render_template（）

```
from flask import Flask, render_template
...
@app.route('/watchlist')
def watchlist():
return render_template('watchlist.html', user=user, movies=movies)
```
在render_template（）函数中，我们首先传入模板的文件名作为参数。==Flask会在程序根目录下的templates文件夹里寻找模板文件==，所以这里传入的文件路径是相对于templates根目录的。除了模板文件路径，我们还以关键字参数的形式传入了模板中使用的变量值。
==提示==
除了render_template（）函数，Flask还提供了一个render_template_string（）函数用来渲染模板字符串。

传入Jinja2中的变量值可以是字符串、列表和字典，也可以是函数、类和类实例，这完全取决于你在视图函数传入的值。
```
<p>这是列表my_list的第一个元素：{{ my_list[0] }}</p>
<p>这是元组my_tuple的第一个元素：{{ my_tuple[0] }}</p>
<p>这是字典my_dict的键为name的值：{{ my_dict['name'] }}</p>
<p>这是函数my_func的返回值：{{ my_func() }}</p>
<p>这是对象my_object调用某方法的返回值：{{ my_object.name() }}</p>

```
如果你想传入函数在模板中调用，那么需要传入函数对象本身，而不是函数调用（函数的返回值），所以仅写出函数名称即可。当把函数传入模板后，我们可以像在Python脚本中一样通过添加括号的方式调用，而且你也可以在括号中传入参数。

## 模板辅助工具
除了基本语法，Jinja2还提供了许多方便的工具，这些工具可以让你更方便地控制模板的输出:
```
from flask import render_template
@app.route('/')
def index():
	return render_template('index.html')
```
### context(上下文)
模板上下文包含了很多变量，其中包括我们调用render_template（）函数时手动传入的变量以及Flask默认传入的变量。除了渲染时传入变量，你也可以在模板中定义变量，使用set标签：
```
{% set navigation = [('/', 'Home'), ('/about', 'About')] %}
```
你也可以将一部分模板数据定义为变量，使用set和endset标签声明开始和结束：
```
{% set navigation %}
	<li><a href="/">Home</a>
	<li><a href="/about">About</a>
{% endset %}
```
#### 内置上下文变量
Flask在模板上下文中提供了一些内置变量，可以在模板中直接使用:![[Flask-2024317-5.png]]
#### 自定义上下文
如果多个模板都需要使用同一变量，那么比起在多个视图函数中重复传入，更好的方法是能够设置一个模板全局变量。Flask提供了一个app.context_processor装饰器，可以用来注册模板上下文处理函数，它可以帮我们完成统一传入变量的工作。==模板上下文处理函数需要返回一个包含变量键值对的字典==:
```
@app.context_processor
def inject_foo():
foo = 'I am foo.'
return dict(foo=foo) # 等同于return {'foo': foo}
```
当我们调用render_template（）函数渲染任意一个模板时，所有使用app.context_processor装饰器注册的模板上下文处理函数（包括Flask内置的上下文处理函数）都会被执行，这些函数的返回值会被添加到模板中，因此我们可以在模板中直接使用foo变量。
==提示==
和在render_template（）函数中传入变量类似，除了字符串、列表等数据结构，你也可以传入函数、类或类实例。除了使用app.context_processor装饰器，也可以直接将其作为方法调
用，传入模板上下文处理函数：
```
...
def inject_foo():
 foo = 'I am foo.'
return dict(foo=foo)
app.context_processor(inject_foo)
```
使用lambda可以简化为：
```
app.context_processor(lambda: dict(foo='I am foo.'))
```

### 全局对象
全局对象是指在所有的模板中都可以直接使用的对象，包括在模板中导入的模板。

#### 1.内置全局函数
Jinja2在模板中默认提供了一些全局函数，常用的三个函数:![[Flask-2024317-6.png]]完整的全局函数列表请访问http://jinja.pocoo.org/docs/2.10/templates/#list-of-global-functions 查看. 除了Jinja2内置的全局函数，Flask也在模板中内置了两个全局函数: ![[Flask-2024317-7.png]]
==提示==
Flask除了把g、session、config、request对象注册为上下文变量，也将它们设为全局变量，因此可以全局使用。url_for（）用来获取URL，用法和在Python脚本中相同。在实际的代
码中，这个URL使用url_for（）生成，传入index视图的端点：
```
<a href="{{ url_for('index') }}">&larr; Return</a>
```
#### 2.自定义全局函数
除了使用app.context_processor注册模板上下文处理函数来传入函数，我们也可以使用app.template_global装饰器直接将函数注册为模板全局函数。比如把bar（）函数注册为模板全局函数:
```
@app.template_global()
def bar():
return 'I am bar.'
```
默认使用函数的原名称传入模板，在app.template_global（）装饰器中使用name参数可以指定一个自定义名称。app.template_global（）仅能用于注册全局函数。
==附注==
你可以直接使用app.add_template_global（）方法注册自定义全局函数，传入函数对象和可选的自定义名称（name），比如

```
app.add_template_global（your_global_function）
```


### 过滤器
在Jinja2中，过滤器（filter）是一些可以用来修改和过滤变量值的特殊函数，过滤器和变量用一个竖线（管道符号）隔开，需要参数的过滤器可以像函数一样使用括号传递。下面是一个对name变量使用title过滤器的例子：
```
{{ name|title }}
```
这会将name变量的值标题化，相当于在Python里调用name.title（）。再比如，使用length获取movies列表的长度，类似于在Python中调用len（movies）：
```
{{ movies|length }}
```
另一种用法是将过滤器作用于一部分模板数据，使用filter标签和endfilter标签声明开始和结束。比如，下面使用upper过滤器将一段文字转换为大写：

```
{% filter upper %}
This text becomes uppercase.
{% endfilter %}
```

#### 1.内置过滤器
Jinja2提供了许多内置过滤器![[Flask-2024317-8.png]]![[Flask-2024317-9.png]]完整的列表请访问http://jinja.pocoo.org/docs/2.10/templates/#builtin-filters 查看.
==附注==
在使用过滤器时，列表中过滤器函数的第一个参数表示被过滤的变量值（value）或字符串（s），即竖线符号左侧的值，其他的参数可以通过添加括号传入。另外，过滤器可以叠加使用，下面的示例为name变量设置默认值，并将其标题化：

```
<h1>Hello, {{ name|default('陌生人')|title }}!</h1>
```
==提示==
默认的自动开启转义仅针对.html、.htm、.xml以及.xhtml后缀的文件，用于渲染模板字符串的render_template_string（）函数也会对所有传入的字符串进行转义。
在确保变量值安全的情况下，这通常意味着你已经对用户输入的内容进行了“消毒”处理。这时如果你想避免转义，将变量作为HTML解析，可以对变量使用safe过滤器：
```
{{ sanitized_text|safe }}
```
另一种将文本标记为安全的方法是在渲染前将变量转换为Markup对象：
```
from flask import Markup
@app.route('/hello')
def hello():
	text = Markup('<h1>Hello, Flask!</h1>')
	return render_template('index.html', text=text)
```
这时在模板中可以直接使用{{text}}。

#### 2.自定义过滤器
如果内置的过滤器不能满足你的需要，还可以添加自定义过滤器。使用app.template_filter（）装饰器可以注册自定义过滤器：

```
from flask import Markup
@app.template_filter()
def musical(s):
	return s + Markup(' &#9835;')
```
和注册全局函数类似，你可以在app.template_filter（）中使用name关键字设置过滤器的名称，默认会使用函数名称。过滤器函数需要接收被处理的值作为输入，返回处理后的值。过滤器函数接收s作为被过滤的变量值，返回处理后的值。我们创建的musical过滤器会在被过滤的变量字符后面添加一个音符（single bar note）图标，因为音符通过HTML
实体&#9835；表示，我们使用Markup类将它标记为安全字符。在使用时和其他过滤器用法相同：

```
{{ name|musical }}
```
==附注==
你可以直接使用app.add_template_filter（）方法注册自定义过滤器，传入函数对象和可选的自定义名称（name），比如
```
app.add_template_filter（your_filter_function）
```

### 测试器
在Jinja2中，测试器（Test）是一些用来测试变量或表达式，返回布尔值（True或False）的特殊函数。比如，number测试器用来判断一个变量或表达式是否是数字，我们使用is连接变量和测试器：

```
{% if age is number %}
	{{ age * 365 }}
{% else %}
	 无效的数字。
{% endif %}
```

#### 1.内置测试器
Jinja2内置了许多测试器:![[Flask-2024317-10.png]]完整的内置测试器列表请访问
http://jinja.pocoo.org/docs/2.10/templates/#list-of-builtin-tests 查看。

在使用测试器时，is的左侧是测试器函数的第一个参数（value），其他参数可以添加括号传入，也可以在右侧使用空格连接，以sameas为例：
```
{% if foo is sameas(bar) %}...
```
等同于：
```
{% if foo is sameas bar %}...
```

#### 2.自定义测试器
和过滤器类似，我们可以使用Flask提供的app.template_test（）装饰器来注册一个自定义测试器。我们创建了一个没有意义的baz过滤器，仅用来验证被测值是否为baz:
```
@app.template_test()
def baz(n):
	if n == 'baz':
		return True
	return False
```
测试器的名称默认为函数名称，你可以在app.template_test（）中使用name关键字指定自定义名称。测试器函数需要接收被测试的值作为输入，返回布尔值。
==附注==
你可以直接使用app.add_template_test（）方法注册自定义测试器，传入函数对象和可选的自定义名称（name），比如
```
app.add_template_test（your_test_function）
```

### 模板环境对象
在Jinja2中，渲染行为由jinja2.Enviroment类控制，所有的配置选项、上下文变量、全局函数、过滤器和测试器都存储在Enviroment实例上。当与Flask结合后，我们并不单独创建Enviroment对象，而是使用Flask创建的Enviroment对象，它存储在app.jinja_env属性上。

在程序中，我们可以使用app.jinja_env更改Jinja2设置。比如，你可以自定义所有的定界符。下面使用variable_start_string和variable_end_string分别自定义变量定界符的开始和结束符号：
```
app = Flask(__name__)
app.jinja_env.variable_start_string = '[['
app.jinja_env.variable_end_string = ']]'
```
== 注意==
在实际开发中，如果修改Jinja2的定界符，那么需要注意与扩展提供模板的兼容问题，一般不建议修改。

模板环境中的全局函数、过滤器和测试器分别存储在Enviroment对象的globals、filters和tests属性中，这三个属性都是字典对象。除了使用Flask提供的装饰器和方法注册自定义函数，我们也可以直接操作这三个字典来添加相应的函数或变量，这通过向对应的字典属性中添加一个键值对实现，传入模板的名称作为键，对应的函数对象或变量作为值。下面是几个简单的示例:
#### 1.添加自定义全局对象
和app.template_global（）装饰器不同，直接操作globals字典允许我们传入任意Python对象，而不仅仅是函数，类似于上下文处理函数的作用。下面的代码使用app.jinja_env.globals分别向模板中添加全局函数bar和全局变量foo：
```
def bar():
	return 'I am bar.'
foo = 'I am foo.'

app.jinja_env.globals['bar'] = bar
app.jinja_env.globals['foo'] = foo
```

#### 2.添加自定义过滤器
下面的代码使用app.jinja_env.filters向模板中添加自定义过滤器smiling：
```
def smiling(s):
	return s + ' :)'

app.jinja_env.filters['smiling'] = smiling
```

#### 3.添加自定义测试器
下面的代码使用app.jinja_env.tests向模板中添加自定义测试器baz：
```
def baz(n):
	if n == 'baz':
		return True
	return False

app.jinja_env.tests['baz'] = baz
```
==附注==
访问http://jinja.pocoo.org/docs/latest/api/#jinja2.Environment 查看Enviroment类的所有属性及用法说明。

# 模板结构组织
除了使用函数、过滤器等工具控制模板的输出外，Jinja2还提供了一些工具来在宏观上组织模板内容。借助这些技术，我们可以更好地实践DRY（Don't Repeat Yourself）原则。

## 局部模板
在Web程序中，我们通常会为每一类页面编写一个独立的模板。比如主页模板、用户资料页模板、设置页模板等。这些模板可以直接在视图函数中渲染并作为HTML响应主体。除了这类模板，我们还会用到另一类非独立模板，这类模板通常被称为局部模板或次模板，因为它们仅包含部分代码，所以我们不会在视图函数中直接渲染它，而是插入到其他独立模板中。
==提示==
当程序中的某个视图用来处理AJAX请求时，返回的数据不需要包含完整的HTML结构，这时就可以返回渲染后的局部模板。

当多个独立模板中都会使用同一块HTML代码时，我们可以把这部分代码抽离出来，存储到局部模板中。这样一方面可以避免重复，另一方面也可以方便统一管理。比如，多个页面中都要在页面顶部显示一个提示条，这个横幅可以定义在局部模板_banner.html中。

我们使用include标签来插入一个局部模板，这会把局部模板的全部内容插在使用include标签的位置。比如，在其他模板中，我们可以在任意位置使用下面的代码插入_banner.html的内容：
```
{% include '_banner.html' %}
```

==提示==
为了和普通模板区分开，局部模板的命名通常以一个下划线开始。

## 宏
宏（macro）是Jinja2提供的一个非常有用的特性，它类似Python中的函数。使用宏可以把一部分模板代码封装到宏里，使用传递的参数来构建内容，最后返回构建后的内容。在功能上，它和局部模板类似，都是为了方便代码块的重用。

为了便于管理，我们可以把宏存储在单独的文件中，这个文件通常命名为macros.html或_macors.html。在创建宏时，我们使用macro和endmacro标签声明宏的开始和结束。在开始标签中定义宏的名称和接收的参数，下面是一个简单的示例：
```
{% macro qux(amount=1) %}
	{% if amount == 1 %}
		I am qux.
	{% elif amount > 1 %}
		We are quxs.
	{% endif %}
{% endmacro %}
```
使用时，需要像从Python模块中导入函数一样使用import语句导入它，然后作为函数调用，传入必要的参数，如下所示：
```
{% from 'macros.html' import qux %}
...
{{ qux(amount=5) }}
```
另外，在使用宏时我们需要注意上下文问题。在Jinja2中，出于性能的考虑，并且为了让这一切保持显式，默认情况下包含（include）一个局部模板会传递当前上下文到局部模板中，但导入（import）却不会。具体来说，当我们使用render_template（）函数渲染一个foo.html模板时，这个foo.html的模板上下文中包含下列对象：
- Flask使用内置的模板上下文处理函数提供的g、session、config、request。
- 扩展使用内置的模板上下文处理函数提供的变量。
- 自定义模板上下文处理器传入的变量。
- 使用render_template（）函数传入的变量。
- Jinja2和Flask内置及自定义全局对象。
- Jinja2内置及自定义过滤器。
- Jinja2内置及自定义测试器。

使用include标签插入的局部模板（比如_banner.html）同样可以使用上述上下文中的变量和函数。而导入另一个并非被直接渲染的模板（比如macros.html）时，这个模板仅包含下列这些对象：
- Jinja2和Flask内置的全局函数和自定义全局函数。
- Jinja2内置及自定义过滤器。
- Jinja2内置及自定义测试器。
因此，如果我们想在导入的宏中使用第一个列表中的2、3、4项，就需要在导入时显式地使用with context声明传入当前模板的上下文：
```
{% from "macros.html" import foo with context %}
```
==注意==
虽然Flask使用内置的模板上下文处理函数传入session、g、request和config，但它同时也使用app.jinja_env.globals字典将这几个变量设置为全局变量，所以我们仍然可以在不显示声明传入上下文的情况下，直接在导入的宏中使用它们。
==提示==
关于宏的编写，更多的细节请访问http://jinja.pocoo.org/docs/latest/templates/#macros 查看。

## 模板继承
Jinja2的模板继承允许你定义一个基模板，把网页上的导航栏、页脚等通用内容放在基模板中，而每一个继承基模板的子模板在被渲染时都会自动包含这些部分。使用这种方式可以避免在多个模板中编写重复的代码。
### 1.编写基模板
基模板存储了程序页面的固定部分，通常被命名为base.html或layout.html。示例程序中的基模板base.html中包含了一个基本的HTML结构，我们还添加了一个简单的导航条和页
```
<!DOCTYPE html>
<html>
<head>
	{% block head %}
		<meta charset="utf-8">
		<title>{% block title %}Template - HelloFlask{% endblock %}</title>
		{% block styles %}{% endblock %}
	{% endblock %}
</head>
<body>
	<nav>
		<ul><li><a href="{{ url_for('index') }}">Home</a></li></ul>
	</nav>
	<main>
		{% block content %}{% endblock %}
	</main>
	<footer>
		{% block footer %}
		...
		{% endblock %}
	</footer>
	{% block scripts %}{% endblock %}
</body>
</html>
```
当子模板继承基模板后，子模板会自动包含基模板的内容和结构。为了能够让子模板方便地覆盖或插入内容到基模板中，我们需要在基模板中定义块（block），在子模板中可以通过定义同名的块来执行继承操作。

块的开始和结束分别使用block和endblock标签声明，而且块之间可以嵌套。在这个基模板中，我们创建了六个块：head、title、styles、content、footer和scripts，分别用来划分不同的代码。其中，head块表示`<head>`标签的内容，title表示`<title>`标签的内容，content块表示页面主体内容，footer表示页脚部分，styles块和scripts块，则分别用来包含CSS文件和JavaScript文件引用链接或页内的CSS和JavaScript代码。

### 2.编写子模板
因为基模板中定义了HTML的基本结构，而且包含了页脚等固定信息，在子模板中我们不再需要定义这些内容，只需要对特定的块进行修改。这时我们可以修改前面创建的电影清单模板watchlist.html和主页模板index.html，将这些子模板的通用部分合并到基模板中，并在子模板
中定义块来组织内容，以便在渲染时将块中的内容插入到基模板的对应位置。以index.html为例，修改后的模板代码:

```
{% extends 'base.html' %}
{% from 'macros.html' import qux %}

{% block content %}
{% set name='baz' %}
<h1>Template</h1>
<ul>
	<li><a href="{{ url_for('watchlist') }}">Watchlist</a></li>
	<li>Filter: {{ foo|musical }}</li>
	<li>Global: {{ bar() }}</li>
	<li>Test: {% if name is baz %}I am baz.{% endif %}</li>
	<li>Macro: {{ qux(amount=5) }}</li>
</ul>
{% endblock %}
```
我们使用extends标签声明扩展基模板，它告诉模板引擎当前模板派生自base.html。
注意
extends必须是子模板的第一个标签。我们在基模板中定义了四个块，在子模板中，我们可以对父模板中的块执行两种操作：

#### （ 1）覆盖内容
当在子模板里创建同名的块时，会使用子块的内容覆盖父块的内容。比如我们在子模板index.html中定义了title块，内容为Home，这会把块中的内容填充到基模板里的title块的位置，最终渲染为`<title>Home</title>`，content块的效果同理。

#### （2）追加内容
如果想要向基模板中的块追加内容，需要使用Jinja2提供的super（）函数进行声明，这会向父块添加内容。比如，下面的示例向基模板中的styles块追加了一行<style>样式定义：
```
{% block styles %}
{{ super() }}
<style>
	.foo {
	color: red;
	}
</style>
{% endblock %}
```
当子模板被渲染时，它会继承基模板的所有内容，然后根据我们定义的块进行覆盖或追加操作，渲染子模板index.html的结果如下所示：

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>Template - HelloFlask</title>
</head>
<body>
	<nav>
		<ul><li><a href="/">Home</a></li></ul>
	</nav>
	<main>
		<h1>Template</h1>
		<ul>
			<li><a href="/watchlist">Watchlist</a></li>
			<li>Filter: I am foo. &#9835;</li>
			<li>Global: I am bar.</li>
			<li>Test: I am baz.</li>
			<li>Macro: We are quxs.
			</li>
		</ul>
	</main>
	<footer>
...
</footer>
</body>
</html>
```

## 模板进阶实践

### 空白控制

### 加载静态文件
一个Web项目不仅需要HTML模板，还需要许多静态文件，比如CSS、JavaScript文件、图片以及音频等。在Flask程序中，默认我们需要将静态文件存储在与主脚本（包含程序实例的脚本）同级目录的static文件夹中。
为了在HTML文件中引用静态文件，我们需要使用url_for（）函数获取静态文件的URL。Flask内置了用于获取静态文件的视图函数，端点值为static，它的默认URL规则为/static/<path：filename>，URL变量filename是相对于static文件夹根目录的文件路径。
==提示==
如果你想使用其他文件夹来存储静态文件，可以在实例化Flask类时使用static_folder参数指定，静态文件的URL路径中的static也会自动跟随文件夹名称变化。在实例化Flask类时使用static_url_path参数则可以自定义静态文件的URL路径。

在示例程序的static目录下保存了一个头像图片avatar.jpg，我们可以通过url_for（'static'，filename='avatar.jpg'）获取这个文件的URL，这个函数调用生成的URL为/static/avatar.jpg，在浏览器中输入http://localhost:5000/static/avatar.jpg 即可访问这个图片。在模板watchlist2.html里，我们在用户名的左侧添加了这个图片，使用url_for（）函数生成图片src属性所需的图片URL，如下所示：
```
<img src="{{ url_for('static', filename='avatar.jpg') }}" width="50">
```
另外，我们还创建了一个存储CSS规则的styles.css文件，我们使用下面的方式在模板中加载这个文件：
```
<link rel="stylesheet" type="text/css" href="{{ url_for('static', filename= 'styles.css' ) }}">
```
在浏览器中访问http://localhost:5000/watchlist2 可以看到添加了头像图片并加载了CSS规则的电影清单页面.

#### 1.添加Favicon
我们经常在命令行看到一条404状态的GET请求记录，请求的URL为/favicon.ico，如下所示：
```
127.0.0.1 - - [08/Feb/2018 18:31:12] "GET /favicon.ico HTTP/1.1" 404 -
```
这个favicon.ico文件指的是Favicon（favorite icon，收藏夹头像/网站头像），又称为shortcut icon、tab icon、website icon或是bookmark icon。顾名思义，这是一个在浏览器标签页、地址栏和书签收藏夹等处显示的小图标，作为网站的特殊标记。浏览器在发起请求时，会自动向根目录请求这个文件，在前面的示例程序中，我们没有提供这个文件，所以才会产生上面的404记录。
要想为Web项目添加Favicon，你要先有一个Favicon文件，并放置到static目录下。它通常是一个宽高相同的ICO格式文件，命名为favicon.ico。
==附注==
除了ICO格式，PNG和（无动画的）GIF格式也被所有主流浏览器支持。

Flask中静态文件的默认路径为/static/filename，为了正确返回Favicon，我们可以显式地在HTML页面中声明Favicon的路径。首先可以在`<head>`部分添加一个`<link`>元素，然后将rel属性设置为icon，如下所示：
```
<link rel="icon" type="image/x-icon" href="{{ url_for('static', filename='favicon.ico') }}">
```
==附注==
大部分教程将rel属性设置为shortcut icon，事实上，shortcut是多余的，可以省略掉。

#### 2.使用CSS框架
#### 3.使用宏加载静态资源

### 消息闪现
Flask提供了一个非常有用的flash（）函数，它可以用来“闪现”需要显示给用户的消息，比如当用户登录成功后显示“欢迎回来！”。在视图函数调用flash（）函数，传入消息内容即可“闪现”一条消息。当然，它并不是我们想象的，能够立刻在用户的浏览器弹出一条消息。实际上，使用功能flash（）函数发送的消息会存储在session中，我们需要在模板中使用全局函数get_flashed_messages（）获取消息并将其显示出来。

==提示==
通过flash（）函数发送的消息会存储在session对象中，所以我们需要为程序设置密钥。可以通过app.secret_key属性或配置变量SECRET_KEY设置。
你可以在任意视图函数中调用flash（）函数发送消息。为了测试消息闪现，我们添加了一个just_flash视图，在函数中发送了一条消息，最后重定向到index视图:
```
from flask import Flask, render_template, flash
app = Flask(__name__)
app.secret_key = 'secret string'
@app.route('/flash')
def just_flash():
flash('I am flash, who is looking for me?')
return redirect(url_for('index'))
```
Jinja2内部使用Unicode，所以你需要向模板传递Unicode对象或只包含ASCII字符的字符串。

Flask提供了get_flashed_message（）函数用来在模板里获取消息，因为程序的每一个页面都有可能需要显示消息，我们把获取并显示消息的代码放在基模板中content块的上面，这样就可以在页面主体内容的上面显示消息:
```
<main>
{% for message in get_flashed_messages() %}
<div class="alert">{{ message }}</div>
{% endfor %}
{% block content %}{% endblock %}
</main>
```
因为同一个页面可能包含多条要显示的消息，所以这里使用for循环迭代get_flashed_message（）返回的消息列表。

### 自定义错误页面
当程序返回错误响应时，会渲染一个默认的错误页面，。默认的错误页面太简单了，而且和其他页面的风格不符，导致用户看到这样的页面时往往会不知所措。我们可以注册错误处理函数来自定义错误页面。

错误处理函数和视图函数很相似，返回值将会作为响应的主体，因此我们首先要创建错误页面的模板文件。为了和普通模板区分开来，我们在模板文件夹templates里为错误页面创建了一个errors子文件夹，并在其中为最常见的404和500错误创建了模板文件，表示404页面的404.html
```
{% extends 'base.html' %}
{% block title %}404 - Page Not Found{% endblock %}
{% block content %}
<h1>Page Not Found</h1>
<p>You are lost...</p>
{% endblock %}
```
错误处理函数需要附加app.errorhandler（）装饰器，并传入错误状态码作为参数。错误处理函数本身则需要接收异常类作为参数，并在返回值中注明对应的HTTP状态码。当发生错误时，对应的错误处理函数会被调用，它的返回值会作为错误响应的主体。
```
from flask import Flask, render_template
...
@app.errorhandler(404)
def page_not_found(e):
	return render_template('errors/404.html'), 404
```
错误处理函数接收异常对象作为参数，内置的异常对象提供了下列常用属性![[Flask-2024317-11.png]]
如果你不想手动编写错误页面的内容，可以将这些信息传入错误页面模板，在模板中用它们来构建错误页面。不过需要注意的是，传入500错误处理器的是真正的异常对象，通常不会提供这几个属性，你需要手动编写这些值。

# 使用Flask-WTF处理表单
WTForms支持在Python中使用类定义表单，然后直接通过类定义生成对应的HTML代码，这种方式更加方便，而且使表单更易于重用。因此，除非是非常简单的程序，或者是你想让表单的定义更加灵活，否则我们一般不会在模板中直接使用HTML编写表单。扩展Flask-WTF集成了WTForms，使用它可以在Flask中更方便地使用WTForms。Flask-WTF将表单数据解析、CSRF保护、文件上传等功能与Flask集成，另外还附加了reCAPTCHA支持。

## 定义WTForms表单类
当使用WTForms创建表单时，表单由Python类表示，这个类继承从WTForms导入的Form基类。一个表单由若干个输入字段组成，这些字段分别用表单类的类属性来表示（字段即Field，你可以简单理解为表单内的输入框、按钮等部件）。下面定义了一个LoginForm类，最终会生成我们在前面定义的HTML表单：

```
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired, Length
class LoginForm(FlaskForm):
	username = StringField('Username', validators=[DataRequired()])
	password = PasswordField('Password', validators=[DataRequired(), Length(8, 128)])
	remember = BooleanField('Remember me')
	submit = SubmitField('Log in')
```
==字段属性名称大小写敏感，不能以下划线或validate开头==。
这里的LoginForm表单类中定义了四个字段：文本字段StringField、密码字段Password-Field、勾选框字段BooleanField和提交按钮字段SubmitField。字段类从wtforms包导入，常用的WTForms字段：![[Flask-2024317-12.png]]


通过实例化字段类时传入的参数，我们可以对字段进行设置，字段类构造方法接收的常用参数如表所示：![[Flask-2024317-13.png]]

在WTForms中，验证器（validator）是一系列用于验证字段数据的类，我们在实例化字段类时使用validators关键字来指定附加的验证器列表。验证器从wtforms.validators模块中导入，常用的验证器如表所示：![[Flask-2024317-14.png]]![[Flask-2024317-15.png]]

validators参数接收一个传入可调用对象组成的列表。==内置的验证器通过实现了__call__（）方法的类表示，所以我们需要在验证器后添加括号==。

name和password字段里，我们都使用了DataRequired验证器，用来验证输入的数据是否有效。另外，password字段里还添加了一个Length验证器，用来验证输入的数据长度是否在给定的范围内。验证器的第一个参数一般为错误提示消息，我们可以使用message关键字传递
参数，通过传入自定义错误信息来覆盖内置消息，比如：
```
name = StringField('Your Name', validators=[DataRequired(message=u'名字不能为空！')])
```

## 输出HTML代码
有些字段最终生成的HTML代码相同，不过WTForms会在表单提交后根据表单类中字段的类型对数据进行处理，转换成对应的Python类型，以便在Python脚本中对数据进行处理。

例化表单类，然后将实例属性转换成字符串或直接调用就可以获取表单字段对应的HTML
代码：

```
>>> form = LoginForm()
>>> form.username()
u'<input id="username" name="username" type="text" value="">'
>>> form.submit()
u'<input id="submit" name="submit" type="submit" value="Submit">'
```

字段的`<label>`元素的HTML代码则可以通过“form.字段名.label”的形式获取：

```
>>> form.username.label()
u'<label for="username">Username</label>'
>>> form.submit.label()
u'<label for="submit">Submit</label>'
```
默认情况下，WTForms输出的字段HTML代码只会包含id和name属性，属性值均为表
单类中对应的字段属性名称。如果要添加额外的属性，通常有两种方法。
### 1.使用render_kw属性
比如下面为username字段使用render_kw设置了placeholder HTML属性：

```
username = StringField('Username', render_kw={'placeholder': 'Your Username'})
```
这个字段被调用后输出的HTML代码如下所示：

```
<input type="text" id="username" name="username" placeholder="Your Username">
```
### 2.在调用字段时传入
在调用字段属性时，通过添加括号使用关键字参数的形式也可以传入字段额外的HTML属性：

```
>>> form.username(style='width: 200px;', class_='bar')
u'<i nput class="bar" id="username" name="username" style="width: 200px;" type="text">'
```
==附注==
class是Python的保留关键字，在这里我们==使用class_来代替class==，渲染后的`<input>`会获得正确的class属性，在模板中调用时则可以直接使用class。
 ==注意==
通过上面的方法也可以修改id和name属性，但表单被提交后，WTForms需要通过name属性来获取对应的数据，所以不能修改name属性值。

## 在模板中渲染表单
为了能够在模板中渲染表单，我们需要把表单类实例传入模板。首先在视图函数里实例化表单类LoginForm，然后在render_template（）函数中使用关键字参数form将表单实例传入模板:

```
from forms import LoginForm
@app.route('/basic')
def basic():
	form = LoginForm()
	return render_template('login.html', form=form)
```
在模板中，只需要调用表单类的属性即可获取字段对应的HTML代码，如果需要传入参数，也可以添加括号:

```
<form method="post">
{{ form.csrf_token }} <!-- 渲染CSRF令牌隐藏字段 -->
{{ form.username.label }}{{ form.username }}<br>
{{ form.password.label }}{{ form.password }}<br>
{{ form.remember }}{{ form.remember.label }}<br>
{{ form.submit }}<br>
</form>
```

需要注意的是，在上面的代码中，除了渲染各个字段的标签和字段本身，我们还调用了form.csrf_token属性渲染Flask-WTF为表单类自动创建的CSRF令牌字段。form.csrf_token字段包含了自动生成的CSRF令牌值，在提交表单后会自动被验证，为了确保表单通过验证，我们必须在表单中手动渲染这个字段。

Flask-WTF为表单类实例提供了一个form.hidden_tag（）方法，这个方法会依次渲染表单中所有的隐藏字段。因为csrf_token字段也是隐藏字段，所以当这个方法被调用时也会渲染csrf_token字段。渲染后获得的实际HTML代码如下所示：

```
<form method="post">
	<input id="csrf_token" name="csrf_token" type="hidden" value="IjVmMDE1ZmFjM2VjYmZjY...i.DY1QSg.IWc1WEWxr3TvmAWCTHRMGjIcDOQ">
	<label for="username">Username</label><br>
	<input id="username" name="username" type="text" value=""><br>
	<label for="password">Password</label><br>
	<input id="password" name="password" type="password" value=""><br>
	<input id="remember" name="remember" type="checkbox" value="y"><label for="remember">Remember me</label><br>
	<input id="submit" name="submit" type="submit" value="Log in"><br>
</form>
```
使用render_kw字典或是在调用字段时传入参数来定义字段的额外HTML属性，通过这种方式添加CSS类，我们可以编写一个Bootstrap风格的表单：

```
...
<form method="post">
	{{ form.csrf_token }}
	<div class="form-group">
	{{ form.username.label }}
	{{ form.username(class='form-control') }}
	</div>
	<div class="form-group">
	{{ form.password.label }}
	{{ form.password(class='form-control') }}
	</div>
	<div class="form-check">
	{{ form.remember(class='form-check-input') }}
	{{ form.remember.label }}
	</div>
	{{ form.submit(class='btn btn-primary') }}
</form>
...
```

## 处理表单数据
### 提交表单
在HTML中，当`<form>`标签声明的表单中类型为submit的提交字段被单击时，就会创建一个提交表单的HTTP请求，请求中包含表单各个字段的数据。表单的提交行为主要由三个属性控制: ![[Flask-2024317-16.png]]

### 验证表单数据
#### 客户端验证
客户端验证（client side validation）是指在客户端（比如Web浏览器）对用户的输入值进行验证。比如，使用HTML5内置的验证属性即可实现基本的客户端验证（type、required、min、max、accept等）。比如，下面的username字段添加了required标志：
`<input type="text" name="username" required>`
如果用户没有输入内容而按下提交按钮，会弹出浏览器内置的错误提示。和其他附加HTML属性相同，我们可以在定义表单时通过render_kw传入这些属性，或是在渲染表单时传入。像required这类布尔值属性，值可以为空或是任意ASCII字符，比如：
`{{ form.username(required='') }}`

#### 服务器端WTForms验证
WTForms验证表单字段的方式是在实例化表单类时传入表单数据，然后对表单实例调用validate（）方法。这会逐个对字段调用字段实例化时定义的验证器，返回表示验证结果的布尔值。如果验证失败，就把错误消息存储到表单实例的errors属性对应的字典中.
如果单纯使用WTForms，我们在实例化表单类时需要首先把request.form传入表单类，而使用Flask-WTF时，表单类继承的FlaskForm基类默认会从request.form获取表单数据，所以不需要手动传入。其数据会被Flask解析为一个字典，可以通过请求对象的form属性获取（request.form）；使用GET方法提交的表单的数据同样会被解析为字典，不过要通过请求对象的args属性获取（request.args）。

#### 在视图函数中验证表单
请求的HTTP方法可以通过request.method属性获取，我们可以使用下面的方式来组织视图函数：

```
from flask import request
...
@app.route('/basic', methods=['GET', 'POST'])
def basic():
	form = LoginForm() # GET + POST
	if request.method == 'POST' and form.validate():
	... # 处理POST请求
	return render_template('forms/basic.html', form=form) # 处理GET请求
```
其中的if语句等价于：
```
if 用户提交表单 and 数据通过验证:
	获取表单数据并保存
```
当请求方法是GET时，会跳过这个if语句，渲染basic.html模板；当请求的方法是POST时（说明用户提交了表单），则验证表单数据。这会逐个字段（包括CSRF令牌字段）调用附加的验证器进行验证。
==注意==
因为WTForms会自动对CSRF令牌字段进行验证，如果没有渲染该字段会导致验证出错，错误消息为“CSRF token is missing”。
Flask-WTF提供的validate_on_submit（）方法合并了这两个操作，因此上面的代码可以简化为：
```
@app.route('/basic', methods=['GET', 'POST'])
def basic():
	form = LoginForm()
	if form.validate_on_submit():
	...
	return render_template('basic.html', form=form)
```
==附注==
除了POST方法，如果请求的方法是PUT、PATCH和DELETE方法，form.validate_on_submit（）也会验证表单数据。

如果form.validate_on_submit（）返回True，则表示用户提交了表单，且表单通过验证，那么我们就可以在这个if语句内获取表单数据:
```
from flask import Flask, render_template, redirect, url_for, flash
...
@app.route('/basic', methods=['GET', 'POST'])
def basic():
	form = LoginForm()
	if form.validate_on_submit():
	username = form.username.data
	flash('Welcome home, %s!' % username)
	return redirect(url_for('index'))
return render_template('basic.html', form=form)
```
表单类的data属性是一个匹配所有字段与对应数据的字典，我们一般直接通过“form.字段属性名.data”的形式来获取对应字段的数据。例如，form.username.data返回username字段的值。
==提示==
在这个if语句内，如果不使用重定向的话，当if语句执行完毕后会继续执行最后的render_template（）函数渲染模板，最后像往常一样返回一个常规的200响应，但这会造成一个问题：在浏览器中，当单击F5刷新/重载时的默认行为是发送上一个请求。如果上一个请求是POST请求，那么就会弹出一个确认窗口，询问用户是否再次提交表单。为了避免出现这个容易让人产生困惑的提示，我们尽量不要让提交表单的POST请求作为最后一个请求。这就是为什么我们在处理表单后返回一个重定向响应，这会让浏览器重新发送一个
新的GET请求到重定向的目标URL。最终，最后一个请求就变成了GET请求。这种用来防止重复提交表单的技术称为PRG（Post/Redirect/Get）模式，即通过对提交表单的POST请求返回重定向响应将最后一个请求转换为GET请求。

#### 在模板中渲染错误消息
如果form.validate_on_submit（）返回False，那么说明验证没有通过。对于验证未通过的字段，WTForms会把错误消息添加到表单类的errors属性中，这是一个匹配作为表单字段的类属性到对应的错误消息列表的字典。我们一般会直接通过字段名来获取对应字段的错误消息列表，即“form.字段名.errors”。比如，form.name.errors返回name字段的错误消息列表。我们可以在模板里使用for循环迭代错误消息列表:

```
<form method="post">
	{{ form.csrf_token }}
	{{ form.username.label }}<br>
	{{ form.username() }}<br>
	{% for message in form.username.errors %}
		<small class="error">{{ message }}</small><br>
	{% endfor %}
	{{ form.password.label }}<br>
	{{ form.password }}<br>
	{% for message in form.password.errors %}
		<small class="error">{{ message }}</small><br>
	{% endfor %}
	{{ form.remember }}{{ form.remember.label }}<br>
	{{ form.submit }}<br>
</form>
```

附注
为了让错误消息更加醒目，我们为错误消息元素添加了error类，这个样式类在style.css文件中定义，它会将文字颜色设为红色。

## 使用Flask-CKEditor集成富文本编辑器
CKEditor（http://ckeditor.com/ ）是一个开源的富文本编辑器，它包含丰富的配置选项，而且有大量第三方插件支持。扩展Flask-CKEditor简化了在Flask程序中使用CKEditor的过程

# 使用Flask-SQLAlchemy管理数据库
实例化Flask-SQLAlchemy提供的SQLAlchemy类，传入程序实例app，以完成扩展的初始化：

```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
app = Flask(__name__)
db = SQLAlchemy(app)

```
虽然我们要使用的大部分类和函数都由SQLAlchemy提供，但在Flask-SQLAlchemy中，大多数情况下，我们不需要手动从SQLAlchemy导入类或函数。在sqlalchemy和sqlalchemy.orm模块中实现的类和函数，以及其他几个常用的模块和对象都可以作为db对象的属性调用。当我们创建这样的调用时，Flask-SQLAlchemy会自动把这些调用转发到对应的类、函数或模块。

## 连接数据库服务器
常用的数据库URI格式:![[Flask-2024318-1.png]]
Flask-SQLAlchemy中，数据库的URI通过配置变量SQLALCHEMY_DATABASE_URI设置，默认为SQLite内存型数据库（sqlite：///：memory：）。SQLite是基于文件的DBMS，不需要设置数据库服务器，只需要指定数据库文件的绝对路径。我们使用app.root_path来定位数据库文件的路径，并将数据库文件命名为data.db

```
import os
...
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URL', 'sqlite:///' + os.path.join(app.root_path, 'data.db'))
```
在生产环境下更换到其他类型的DBMS时，数据库URL会包含敏感信息，所以这里优先从环境变量DATABASE_URL获取. 设置好数据库URI后，在Python Shell中导入并查看db对象会获得下面的输出：
```
>>> from app import db
>>> db
<SQLAlchemy engine=sqlite:///Path/to/your/data.db>
```
## 定义数据库模型
用来映射到数据库表的Python类通常被称为数据库模型（model），一个数据库模型类对应数据库中的一个表。定义模型即使用Python类定义表模式，并声明映射关系。所有的模型类都需要继承Flask-SQLAlchemy提供的db.Model基类。

```
class Note(db.Model):
id = db.Column(db.Integer, primary_key=True)
body = db.Column(db.Text)
```
常用的SQLAlchemy字段类型如表所示:![[Flask-2024318-2.png]]
常用的SQLAlchemy字段参数: ![[Flask-2024318-3.png]]

## 创建数据库和表
创建模型类后，我们需要手动创建数据库和对应的表，也就是我们常说的建库和建表。这通过对我们的db对象调用create_all（）方法实现：

```
$ flask shell
>>> from app import db
>>> db.create_all()
```

==注意==
如果你将模型类定义在单独的模块中，那么必须在调用db.create_all（）方法前导入相应模块，以便让SQLAlchemy获取模型类被创建时生成的表信息，进而正确生成数据表。
通过下面的方式可以查看模型对应的SQL模式（建表语句）：

```
>>> from sqlalchemy.schema import CreateTable
>>> print(CreateTable(Note.__table__))
CREATE TABLE note (
id INTEGER NOT NULL,
body TEXT,
PRIMARY KEY (id)
)
```
## 数据库操作
SQLAlchemy使用数据库会话来管理数据库操作，这里的数据库会话也称为事务（transaction）。Flask-SQLAlchemy自动帮我们创建会话，可以通过db.session属性获取。数据库中的会话代表一个临时存储区，你对数据库做出的改动都会存放在这里。你可以调用add（）方法将新创建的对象添加到数据库会话中，或是对会话中的对象进行更新。只有当你对数据库会话对象调用commit（）方法时，改动才被提交到数据库，这确保了数据提交的一致
性。另外，数据库会话也支持回滚操作。当你对会话调用rollback（）方法时，添加到会话中且未提交的改动都将被撤销。

### create
```
>>> from app import db, Note
>>> note1 = Note(body='remember Sammy Jankis')
>>> note2 = Note(body='SHAVE')
>>> note3 = Note(body='DON'T BELIEVE HIS LIES, HE IS THE ONE, KILL HIM')
>>> db.session.add(note1)
>>> db.session.add(note2)
>>> db.session.add(note3)
>>> db.session.commit()
```
我们在创建模型类实例的时候并没有定义id字段的数据，这是因为主键由SQLAlchemy管理。模型类对象创建后作为临时对象（transient），当你提交数据库会话后，模型类对象才会转换为数据库记录写入数据库中，这时模型类对象会自动获得id值.

### read
用模型类提供的query属性附加调用各种过滤方法及查询方法可以完成这个任务。
一般来说，一个完整的查询遵循下面的模式：
`<模型类>.query.<过滤方法>.<查询方法>`
从某个模型类出发，通过在query属性对应的Query对象上附加的过滤方法和查询函数对模型类对应的表中的记录进行各种筛选和调整，最终返回包含对应数据库记录数据的模型类实例，对返回的实例调用属性即可获取对应的字段数据。
常用的SQLAlchemy查询方法:![[Flask-2024318-4.png]]![[Flask-2024318-5.png]]
- all（）返回所有记录：
```
>>> Note.query.all()
[<Note u'remember Sammy Jankis'>, <Note u'SHAVE'>, <Note u'DON'T BELIEVE HIS LIES, HE IS THE ONE, KILL HIM'>]
```

- first（）返回第一条记录：
```
>>> note1 = Note.query.first()
>>> note1
<Note u'remember Sammy Jankis'>
>>> note1.body
u'remember Sammy Jankis'
```
- get（）返回指定主键值（id字段）的记录：
```
>>> note2 = Note.query.get(2)
>>> note2
<Note u'SHAVE'>
```
- count（）返回记录的数量：

```
>>> Note.query.count()
3
```

SQLAlchemy还提供了许多过滤方法，使用这些过滤方法可以获取更精确的查询，比如获取指定字段值的记录。对模型类的query属性存储的Query对象调用过滤方法将返回一个更精确的Query对象。因为每个过滤方法都会返回新的查询对象，所以过滤器可以叠加使用。在查询对象上调用前面介绍的查询方法，即可获得一个包含过滤后的记录的列表。

完整的查询方法和过滤方法列表在http://docs.sqlalchemy.org/en/latest/orm/query.html 可以看到。 常用的SQLAlchemy过滤方法:![[Flask-2024318-6.png]]

filter（）方法是最基础的查询方法。它使用指定的规则来过滤记录，下面的示例在数据库里找出了body字段值为“SHAVE”的记录：

```
>>> Note.query.filter(Note.body='SHAVE').first()
<Note u'SHAVE'>
```
直接打印查询对象或将其转换为字符串可以查看对应的SQL语句：
```
>>> print(Note.query.filter_by(body='SHAVE'))
	SELECT note.id AS note_id, note.body AS note_body
	FROM note
	WHERE note.body = ?
```
在filter（）方法中传入表达式时，除了“==”以及表示不等于的“！=”，其他常用的查询操作符以及使用示例如下所示：
- LIKE：`filter(Note.body.like('%foo%'))`
- IN：`filter(Note.body.in_(['foo', 'bar', 'baz']))`
- NOT IN：`filter(~Note.body.in_(['foo', 'bar', 'baz']))`
- AND：
```
# 使用and_()
from sqlalchemy import and_
filter(and_(Note.body == 'foo', Note.title == 'FooBar'))
# 或在filter()中加入多个表达式，使用逗号分隔
filter(Note.body == 'foo', Note.title == 'FooBar')
# 或叠加调用多个filter()/filter_by()方法
filter(Note.body == 'foo').filter(Note.title == 'FooBar')
```
- OR：
```
from sqlalchemy import or_
filter(or_(Note.body == 'foo', Note.body == 'bar'))
```


完整的可用操作符列表可以访问http://docs.sqlalchemy.org/en/latest/core/sqlelement.html#sqlalchemy.sql.operators.ColumnOperators查看。

和filter（）方法相比，filter_by（）方法更易于使用。在filter_by（）方法中，你可以使用关键字表达式来指定过滤规则。更方便的是，你可以在这个过滤器中直接使用字段名称。下面的示例使用filter_by（）过滤器完成了同样的任务：

```
>>> Note.query.filter_by(body='SHAVE').first()
<Note u'SHAVE'>
```
### Update
更新一条记录非常简单，直接赋值给模型类的字段属性就可以改变字段值，然后调用commit（）方法提交会话即可：

```
>>> note = Note.query.get(2)
>>> note.body
u'SHAVE'
>>> note.body = 'SHAVE LEFT THIGH'
>>> db.session.commit()
```
### Delete
删除记录和添加记录很相似，不过要把add（）方法换成delete（）方法，最后都需要调用commit（）方法提交修改：

```
>>> note = Note.query.get(2)
>>> db.session.delete(note)
>>> db.session.commit()

```
## 在视图函数里操作数据库
在视图函数里操作数据库需要把查询结果作为参数传入模板渲染出来，或是获取表单的字段值作为提交到数据库的数据。
### 1.Create
为了支持输入笔记内容，我们先创建一个用于填写新笔记的表单，如下所示：

```
from flask_wtf import FlaskForm
from wtforms import TextAreaField, SubmitField
from wtforms.validators import DataRequired
class NewNoteForm(FlaskForm):
	body = TextAreaField('Body', validators=[DataRequired()])
	submit = SubmitField('Save')
```
我们创建一个new_note视图，这个视图负责渲染创建笔记的模板，并处理表单的提交:
```
@app.route('/new', methods=['GET', 'POST'])
def new_note():
	form = NewNoteForm()
	if form.validate_on_submit():
		body = form.body.data
		note = Note(body=body)
		db.session.add(note)
		db.session.commit()
		flash('Your note is saved.')
		return redirect(url_for('index'))
	return render_template('new_note.html', form=form)
```
我们先来看看form.validate_on_submit（）返回True时的处理代码。当表单被提交且通过验证时，我们获取表单body字段的数据，然后创建新的Note实例，将表单中body字段的值作为body参数传入，最后添加到数据库会话中并提交会话。这个过程接收用户通过表单提交的数据并保存到数据库中，最后我们使用flash（）函数发送提示消息并重定向到index视图。
表单在new_note.html模板中渲染：
```
{% block content %}
<h2>New Note</h2>
<form method="post">
	{{ form.csrf_token }}
	{{ form_field(form.body, rows=5, cols=50) }}
	{{ form.submit }}
</form>
{% endblock %}
```
index视图用来显示主页，目前它的所有作用就是渲染主页对应的模板：

```
@app.route('/')
def index():
return render_template('index.html')
```
在对应的index.html模板中，我们添加一个指向创建新笔记页面的链接：

```
<h1>Notebook</h1>
<a href="{{ url_for('new_note') }}">New Note</a>
```

### 2.Read
为了在主页列出所有保存的笔记，我们需要修改index视图，修改后的index视图

```
@app.route('/')
def index():
	form = DeleteForm()
	notes = Note.query.all()
	return render_template('index.html', notes=notes, form=form)
```
在模板中将笔记们显示出来
```
<h1>Notebook</h1>
<a href="{{ url_for('new_note') }}">New Note</a>
<h4>{{ notes|length }} notes:</h4>
{% for note in notes %}
	<div class="note">
		<p>{{ note.body }}</p>
	</div>
{% endfor %}
```
在模板中，我们迭代这个notes列表，调用Note对象的body属性（note.body）获取body字段的值。另外，我们还通过length过滤器获取笔记的数量。

### 3.Update
首先是编辑笔记的表单：
```
class EditNoteForm(FlaskForm):
body = TextAreaField('Body', validators=[DataRequired()])
submit = SubmitField('Update')
```
你会发现这和创建新笔记NewNoteForm唯一的不同就是提交字段的标签参数（作为`<input>`的value属性），因此这个表单的定义也可以通过继承来简化：
```
class EditNoteForm(NewNoteForm):
submit = SubmitField('Update')
```
用来渲染更新笔记页面和处理更新表单提交的edit_note视图

```
@app.route('/edit/<int:note_id>', methods=['GET', 'POST'])
def edit_note(note_id):
	form = EditNoteForm()
	note = Note.query.get(note_id)
	if form.validate_on_submit():
		note.body = form.body.data
		db.session.commit()
		flash('Your note is updated.')
		return redirect(url_for('index'))
	form.body.data = note.body
	return render_template('edit_note.html', form=form)
```
这个视图通过URL变量note_id获取要被修改的笔记的主键值（id字段），然后我们就可以使用get（）方法获取对应的Note实例。当表单被提交且通过验证时，我们将表单中body字段的值赋给note对象的body属性，然后提交数据库会话，这样就完成了更新操作。和创建笔记相同，我们接着发送提示消息并重定向到index视图。唯一需要注意的是，在GET请求的执行流程中，我们添加了下面这行代码：
`form.body.data = note.body`
因为要添加修改笔记内容的功能，那么当我们打开修改某个笔记的页面时，这个页面的表单中必然要包含笔记原有的内容。如果手动创建HTML表单，那么你可以通过将note记录传入模板，然后手动为对应字段中填入笔记的原有内容，比如：
`<textarea name="body">{{ note.body }}</textarea>`
其他input元素则通过value属性来设置输入框中的值，比如：
`<input name="foo" type="text" value="{{ note.title }}">`

使用WTForms可以省略这些步骤，当我们渲染表单字段时，如果表单字段的data属性不为空，WTForms会自动把data属性的值添加到表单字段的value属性中，作为表单的值填充进去，我们不用手动为value属性赋值。因此，将存储笔记原有内容的note.body属性赋值给表单body字段的data属性即可在页面上的表单中填入原有的内容。
最后的工作是在主页笔记列表中的每个笔记内容下添加一个编辑按钮，用来访问编辑页面：
```
{% for note in notes %}
<div class="note">
	<p>{{ note.body }}</p>
	<a class="btn" href="{{ url_for('edit_note', note_id=note.id) }}">Edit</a>
</div>
{% endfor %}
```
生成edit_note视图的URL时，我们传入当前note对象的id（note.id）作为URL变量note_id的值。

### delete
像删除这类修改数据的操作绝对不能通过GET请求实现，正确的做法是为删除操作创建一个表单，如下所示：

```
class DeleteNoteForm(FlaskForm):
submit = SubmitField('Delete')
```

这个表单类只有一个提交字段，因为我们只需要在页面上显示一个删除按钮来提交表单。删除表单的提交请求由delete_note视图处理:

```
@app.route('/delete/<int:note_id>', methods=['POST'])
def delete_note(note_id):
	form = DeleteForm()
	if form.validate_on_submit():
		note = Note.query.get(note_id) # 获取对应记录
		db.session.delete(note) # 删除记录
		db.session.commit() # 提交修改
		flash('Your note is deleted.')
	else:
		abort(400)
	return redirect(url_for('index'))
```
==注意==
在delete_note视图的app.route（）中，methods列表仅填入了POST，这会确保该视图仅监听POST请求。和编辑笔记的视图类似，这个视图接收note_id（主键值）作为参数。如果提交表单且通过验证（唯一需要被验证的是CSRF令牌），就使用get（）方法查询对应的记录，然后调用db.session.delete（）方法删除并提交数据库会话。如果验证出错则使用abort（）函数返回400错误响应。
因为删除按钮要在主页的笔记内容下添加，我们需要在index视图中实例化DeleteNote-Form类，然后传入模板。在index.html模板中，我们渲染这个表单：

```
{% for note in notes %}
<div class="note">
	<p>{{ note.body }}</p>
	<a class='btn' href="{{ url_for('edit_note', note_id=note.id) }}">Edit</a>
	<form method="post" action="{{ url_for('delete_note', note_id=note.id) }}">
		{{ form.csrf_token }}
		{{ form.submit(class='btn') }}
	</form>
</div>
{% endfor %}
```
我们将表单的action属性设置为删除当前笔记的URL。构建URL时，URL变量note_id的值通过note.id属性获取，当单击提交按钮时，会将请求发送到action属性中的URL。添加删除表单的主要目的就是防止CSRF攻击，所以不要忘记渲染CSRF令牌字段form.csrf_token。
==提示==
在HTML中，`<a>`标签会显示为链接，而提交按钮会显示为按钮，为了让编辑和删除笔记的按钮显示相同的样式，我们为这两个元素使用了同一个CSS类“.btn”，具体可以在static/style.css文件中查看。作为替代，你可以考虑使用JavaScript创建监听函数，当删除按钮按下时，提交对应的隐藏表单。

## 定义关系
### 配置Python Shell上下文
在上面的许多操作中，每一次使用flask shell命令启动Python Shell后都要从app模块里导入db对象和相应的模型类。为什么不把它们自动集成到Python Shell上下文里呢？就像Flask内置的app对象一样。这当然可以实现！我们可以使用app.shell_context_processor装饰器注册一个
shell上下文处理函数。它和模板上下文处理函数一样，也需要返回包含变量和变量值的字典:
```
# ...
@app.shell_context_processor
def make_shell_context():
return dict(db=db, Note=Note) # 等同于{'db': db, 'Note': Note}
```
当你使用flask shell命令启动Python Shell时，所有使用app.shell_context_processor装饰器注册的shell上下文处理函数都会被自动执行，这会将db和Note对象推送到Python Shell上下文里：

```
$ flask shell
>>> db
<SQLAlchemy engine=sqlite:///Path/to/your/data.db>
>>> Note
<class 'app.Note'>
```
### 一对多
一对多关系示例
```
# ...
class Author(db.Model):
	id = db.Column(db.Integer, primary_key=True)
	name = db.Column(db.String(70), unique=True)
	phone = db.Column(db.String(20))

class Article(db.Model):
	id = db.Column(db.Integer, primary_key=True)
	title = db.Column(db.String(50), index=True)
	body = db.Column(db.Text)
```

#### 1.定义外键
因为外键只能存储单一数据（标量），所以外键总是在“多”这一侧定义. 在Article模型中，我们定义一个author_id字段作为外键：

```
class Article(db.Model):
...
author_id = db.Column(db.Integer, db.ForeignKey('author.id'))
```
#### 2.定义关系属性
定义关系的第二步是使用关系函数定义关系属性。关系属性在关系的出发侧定义，即一对多关系的“一”这一侧。一个作者拥有多篇文章，在Author模型中，我们定义了一个articles属性来表示对应的多篇文章：

```
class Author(db.Model):
...
articles = db.relationship('Article')
```
个属性并没有使用Column类声明为列，而是使用了db.relationship（）关系函数定义为关系属性，因为这个关系属性返回多个记录，我们称之为集合关系属性。relationship（）函数的第一个参数为关系另一侧的模型名称，它会告诉SQLAlchemy将Author类与Article类建立关系。当这个关系属性被调用时，SQLAlchemy会找到关系另一侧（即article表）的外键字段（即author_id），然后反向查询article表中所有author_id值为当前表主键值（即author.id）的记录，返回包含这些记录的列表，也就是返回某个作者对应的多篇文章记录。

我们先创建一个作者记录和两个文章记录，并添加到数据库会话中：

```
>>> foo = Author(name='Foo')
>>> spam = Article(title='Spam')
>>> ham = Article(title='Ham')
>>> db.session.add(foo)
>>> db.session.add(spam)
>>> db.session.add(ham)
```

#### 3.建立关系
建立关系有两种方式，
- 第一种方式是为外键字段赋值，比如：
```
>>> spam.author_id = 1
>>> db.session.commit()
```
我们将spam对象的author_id字段的值设为1，这会和id值为1的Author对象建立关系。提交数据库改动后，如果我们对id为1的foo对象调用articles关系属性，会看到spam对象包括在返回的Article对象列表中：
```
>>> foo.articles
[<Article u'Spam'>, <Article u'Ham'>]
```

- 另一种方式是通过操作关系属性，将关系属性赋给实际的对象即可建立关系。集合关系属性可以像列表一样操作，调用append（）方法来与一个Article对象建立关系：
```
>>> foo.articles.append(spam)
>>> foo.articles.append(ham)
>>> db.session.commit()
```


