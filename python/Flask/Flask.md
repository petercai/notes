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

# 在Flask中处理请求
在Flask中，我们需要让请求的URL匹配对应的视图函数，视图函数返回值就是URL对应的资源。
## 1.路由匹配
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
## 2.设置监听的HTTP方法
我们可以在app.route（）装饰器中使用methods参数传入一个包含监听的HTTP方法的可迭代对象。比如，下面的视图函数同时监听GET请求和POST请求：
```
@app.route('/hello', methods=['GET', 'POST'])
def hello():
return '<h1>Hello, Flask!</h1>'
```
## 3.URL处理
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
# 请求钩子 (hooks)
有时我们需要对请求进行预处理（preprocessing）和后处理（postprocessing），这时可以使用Flask提供的一些请求钩子（Hook），它们可以用来注册在请求处理的不同阶段执行的处理函数（或称为回调函数，即Callback）。这些请求钩子使用装饰器实现，通过程序实例app调用，用法很简单：以before_request钩子（请求之前）为例，当你对一个函数附加了app.before_request装饰器后，就会将这个函数注册为before_request处理函数，每次执行请求前都会触发所有before_request处理函数。Flask默认实现的五种请求钩子:
![[Flask-2024316-5.png]]