[TOC]

# 一. 配置管理

Flask 可以通过配置相关属性来控制属性开关，详细配置可以查看文档：https://dormousehole.readthedocs.io/en/latest/config.html

- 常用配置


`JSON_AS_ASCII`，默认配置为 True，但会导致传递中文时出现异常，因此可以设置其为 False

```python
from flask import Flask

app = Flask(__name__)
app.config['JSON_AS_ASCII'] = False

...
```

- 通过配置文件配置

当配置项较多时，可以使用配置文件来配置相关属性，如创建配置文件 `config.py`

```python
JSON_AS_ASCII = False
...
```

则可以在主文件中加载该配置文件

```python
from flask import Flask
import config # 导入上述配置文件

app = Flask(__name__)
app.config.from_object(config) # 从配置文件中加载相关配置

...
```

# 二. URL 与视图

## 1. URL 与函数的映射

一个 URL 要与执行函数进行映射，使用的时 `@app.route()` 装饰器，在该装饰器中，可以指定 URL 的规则来进行更加详细的映射，比如现在要映射一个文章详情的 URL，文章详情的 URL 是 /article/id/，那么文章的id是不固定的，可能取值为1/2/3或者其他，因此可以通过以下方式传递参数

```python
@app.route('/article/<id>')
def article(id):
	return '%s article detail' % id
```

其中`<id>`是一种固定写法，其语法为`<variable>`，variable 默认的数据类型是字符串。如果需要指定数据类型，则需要写作`<converter: variable>`，其中`converter`就是类型名称。数据类型分类有以下几种：

- string: 默认数据类型，接收任何没有`/`的字符串
- int: 整形
- float: 浮点型
- path: 和 string 类似，但是可以传递`/`符号
- uuid: uuid 类型的字符串
- any: 可以指定多种路径

> <font size=6>**Tips**</font>: Flask <font color=blue>返回对象数据类型必须是 string/dict/tuple/Response instance/WSGI callable</font>，如果返回值是<font color=red size=5>json数组</font>，则会抛出数据异常，可以通过 Flask 的<font color=red size=5> `jsonify`将其转化为可返回数据对象</font>
>
> ```python
> from flask import Flask, jsonify
> 
> ...
> data = [{'key1': 'value1'}, {'key2': 'value2'}, ...]
> 
> 
> @app.route('/test')
> def test():
>     return jsonify(data)
> ```

## 2. 构造 URL

一般通过 URL 就可以执行到某一个函数，反过来，可以利用 `url_for` 可以实现通过知道函数名来获得其 URL。`url_for()`函数接受两个及以上参数，函数名作为第一个参数，接收对应 URL 规则的命名参数，如果还出现其他参数，则会添加到 URL 后作为查询参数

通过构建 URL 的方式而选择直接在代码中拼 URL 的原因有两点：

1. 将来如果修改了 URL，但没有修改该 URL 对应的函数名，因此就不用多处替换 URL
2. `url_for()`函数会定义一些转义字符和 unicode 字符串

```python
from flask import Flask, jsonify, url_for

app = flask(__name__)

@app.route("/article/<id>")
def article(book_id):
    return "%s article detail:" % book_id

@app.route('/')
def index(request):
    print(url_for('article', id=1))
    return "首页"
```

访问首页时，即可看到后台打印出结果 /article/1

## 3. 指定 HTTP 方法

在 `@app.route()` 中可以传入关键字参数 `methods` 来指定本方法支持的 HTTP 方法，默认情况下，只能使用 GET 请求

```python
@app.route('/test', methods=['GET', 'POST'])
def test():
    return 'test'
```

## 4. 页面跳转和重定向

重定向分为永久性重定向和暂时性重定向，在页面上体现的操作就是浏览器会从一个页面自动跳转到另一个页面

- 永久性重定向：http 的状态码是 301，多用于旧网址被废弃要转到一个新的网址确保用户的访问
- 暂时性重定向：http 的状态码是 302，表示页面的暂时性跳转

在 flask 中，重定向是可以通过 `flask.redirect(location, code=302)` 函数实现，`location` 表示需要重定向到的 URL，应该配合之前的`url_for()`函数使用，`code` 表示使用哪个重定向，默认为 302 也即暂时性重定向，可以修改为 301 来实现永久性重定向

```python
from flask import redirect, url_for

...

@app.route('test')
def test():
    user_id = request.args.get('id')
    if user_id:
        return '存在'
    else:
        return redirect(url_for('index'))

@app.route('/')
def index():
    return "首页"
```

# 三. Jinja2 模板

## 1. 渲染模板

渲染 Jinja2 模板，可以通过 `render_template()`方法

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/test')
def test():
    return render_template('about.html')
```

当访问 /test 路径时，`test()`函数会在当前目录下的 templates 文件夹下寻找 `about.html` 模板文件

如果想更改模板文件地址，则应该在创建 Flask 对象时，给 Flask 传递一个关键字参数 `template_folder` 以指定具体的路径

```python
from flask import Flask

app = Flask(__name__, template_folder=r'C:/templates')
...
```

# 四. 蓝图和子域名

蓝图可以将各部分模块化，然后在主程序中，通过`app.register_blueprint()`方法将蓝图注册进 URL 映射中，如现有蓝图文件

```python
from flask import Blueprint

bp = Blueprint('user', __name__, url_prefix='/user/')

@bp.route('/')
def index():
    return '用户首页'

@bp.route('profile/')
def profile():
    return '个人简介'
```

则在主程序中，可以通过上述函数将蓝图注册

```python
from flask import Flask
import user # 上述蓝图文件

app = Flask(__name__)
app.register_blueprint(user.bp)

...
```

通过上述操作，即可正常访问 /user/profile 等路径

# 五. SQLAlchemy

## 1. SQLAlchemy 与 Flask-SQLAlchemy

SQLAlchemy：一个独立的 ORM 框架，可以独立于 Flask 存在，也可以在其他项目中使用

Flask-SQLAlchemy：对 SQLALchemy 的一个封装，能够更适合在 Flask 中使用

- SQLALchemy 的使用

```python
from sqlalchemy import create_engine

HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'database'
USERNAME = 'root'
PASSWORD = 'password'
DB_URI = 'mysql+pymysql://{}:{}@{}:{}/{}'.format(USERNAME, PASSWORD, HOSTNAME, PORT, DATABASE)

engine = create_engine(DB_URI)

with engine.connect() as conn:
    rs = conn.execute("sql")
    print rs.fetchone()
```

- flask-SQLAlchemy 的使用

```python
from flask import Flask
from flask-sqlalchemy import SQLAlchemy

HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'database'
USERNAME = 'root'
PASSWORD = 'password'
DB_URI = 'mysql+pymysql://{}:{}@{}:{}/{}'.format(USERNAME, PASSWORD, HOSTNAME, PORT, DATABASE)
app.config['SQLALCHEMY_DATABASE_URI'] = DB_URI
db = SQLAlchemy(app)

@app.route('/')
def test():
    # 测试数据库是否连接成功
    engine = db.get_engine()
    with engine.connect() as conn:
        rs = conn.execute('sql')
        print(rs.fetchone())
```

## 2. ORM 映射与数据的增删改查

### 2.1 ORM 映射

定义 ORM 模型（实体类），要继承 `db.Model` 类

```python
...
db = SQLAlchemy(app)

class article(db.Model):
    __tablename__ = 'article' # 对应的表名
    id = db.Column(db.Integer, primary_key=True, autoincrement=True) # 设置 id 字段数据类型为整形，主键，自增
    title = db.Column(db.String(200), nullable=False) # 设置 title 字段数据类型为最大长度为200的字符串，不允许为空
    content = db.Column(db.Text, nullable=False) # 设置 content 字段数据类型为文本，不允许为空
   
db.create_all()
```

如果执行`db.create_all()`，且 article 表不存在，则代码会自动在数据库中创建该表

### 2.2. 添加数据

```python
@app.route('/add')
def add():
    article = Article(title='标题', content='内容')
    db.session.add(article)
    db.session.commit() # 提交
    return '操作成功'
```

### 2.3. 查询数据

```python
@app.route('query')
def query():
    article = Article.query.filter_by(id=1)[0]
    return '查询成功'
```

`filter_by` 查询结果为一个类列表对象，如确定查询为单例，直接返回第一条数据即可

### 2.4. 修改数据

```python
@app.route('update')
def update():
    article = Article.query.filter_by(id=1)[0] # 修改前先查确保数据存在
    article.content = 'new content'
    db.session.commit()
```

### 2.5. 删除数据

```python
@app.route('delete')
def delete():
    Article.query.filter_by(id=1).delete() # 删除前先查确保数据存在
    db.session.commit()
```

## 3. 一对多关系实现

```python
class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    username = db.Column(db.String(200), nullable=False)

class article(db.Model):
    __tablename__ = 'article'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    title = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text, nullable=False)
    # 外键设置
    # 1. 外键数据类型一定要是所引用字段的数据类型
    # 2. db.ForeignKey('表名.字段名')
    # 3. 外键是数据数据库层面的，不推荐直接在 ORM 中使用
    author_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    
    # relationship
    # 1. 第一个参数是模型的名字，必须要和模型的名字保持一致
    # 2. backref：代表反向引用，表示对方访问该类时的字段名称
    author = db.relationship('User', backref= 'articles')
    
db.drop_all()
db.create_all()

@app.route('/test')
def test():
    article = Article(title='title1', content='content1')
    user = User(username='username1')
    article.author = user
    db.session.add(article)
    db.session.commit()
    
    user.articles # 由 user 到 article 的反向引用，因为是一对多关系，因此得到的是一个对象列表
```

## 4. 一对一关系时间

```python
class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    username = db.Column(db.String(200), nullable=False)
    
class UserExtension(db.Model):
    __tablename__ = 'user_extension'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    school = db.Column(db.String(200))
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    
    # db.backref
    # 1. 在反向引用时，如果需要传递一些其他的参数，那么就需要用到这个函数，否则不需要使用，只要在 relationship 的 backref 参数上，设置反向引用的名称就可以
    # 2. userlist=False，代表反向引用的时候，不是一个列表，而是一个对象
    user = db.relationship("User", backref=db.backref('extension', uselist=False))
    
    
@app.route('test')
def test():
    user = User.query.filter_by(id=1).first()
    extension = UserExtension(school='清华大学')
    user.extension = extension
    db.session.commit()
```

# 六. Flask-WTF 表单验证

Flask-WTF 是简化了 WTForms 操作的一个第三方库，WTForms 表单的两个主要功能是验证用户提交数据的合法性以及渲染模板，当然还包括 CORF 保护、文件上传等

## 1. 字段验证

现有 forms.py 文件，在内部创建 RegistForm 的注册验证表单

```python
import wtforms
from wtforms.validators import length, email

class LoginForm(wtforms.Form):
    email = wtforms.StringField(validators=[length(min=5, max=10), email()]) # 邮箱为字符串，最小长度为5，最大长度为10，格式为邮箱
    password = wtforms.StringField(validators=[length(min=6, max=20)]) # 密码为字符串，最小长度为6，最大长度为20
```

则在业务逻辑中可以通过如下方式使用

```python
from flask import Flask, request, render_template
from forms import LoginForm

app = Flask(__name__)

@app.route('/login')
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        form = LoginForm(request.form) # 验证 POST 表单
         # form = LoginForm(request.args) 验证 GET 表单
        if form.validate():
            return '通过验证'
        else:
            return '参数验证失败'
```

Flask-WTF 包含多种表单字段类型的定义，如下为几种常用类型

| 标准表单字段  |                   描述                    |
| :-----------: | :---------------------------------------: |
|   TextFiled   |   表示`<input type='text'>HTML表单元素`   |
| BooleanField  | 表示`<input type='checkbox'>HTML表单元素` |
| DecimalField  |      用于显示带小数的数字的文本字段       |
| IntegerField  |          用于显示整数的文本字段           |
|  RadioField   |  表示`<input type='radio'>`HTML表单元素   |
|  SelectField  |             表示选择表单元素              |
| TextAreaField |       表示`<textarea>`HTML表单元素        |
| PasswordField | 表示`<input type='password'>`HTML表单元素 |
|  SubmitField  |  表示`<input type='submit'>`HTML表单元素  |

同时包含多种不同验证器，如下为几种常用验证器

|    验证器    |                     描述                     |
| :----------: | :------------------------------------------: |
| DataRequired |             检查输入字段是否为空             |
|    Email     |    检查字段中的文本是否遵循电子邮件ID约定    |
|  IPAddress   |            在输入字段中验证IP地址            |
|    Length    | 验证输入字段中的字符串的长度是否在给定范围内 |
| NumberRange  |        验证给定范围内输入字段中的数字        |
|     URL      |          验证在输入字段中输入的URL           |

## 2. 密码加密与验证

Flask 依赖于 Werkzeug，安装 Flask 时会自动安装 Werkzeug 包，可以利用 Werkzeug 包进行数据加密及数据验证

```python
from werkzeug.security import generate_password_hash, check_password_hash

hash_password = generate_password_hash(password) # 密码加密

...
user = UserModel.query.filter_by(account=account).first() # 从数据库中查找用户信息
if check_password_hash(user.password, hash_password): # 数据库存储密码与本次提交密码的比较与验证
    print('验证成功')
else:
    print('验证失败')
```

即可实现密码加密与密码验证操作

# 七. 钩子函数和上下文处理器

## 1. 钩子函数

钩子函数，类似于过滤器，在请求对应接口前先进行某一步操作，之后再分发处理，使用 Flask 中 `@app.before_request` 即可实现此功能

```python
from flask import Flask, session, g # g 为 Flask 中的全局变量，可以在蓝图中获取

...

@app.before_request
def before_request():
    user_id = session.get('user_id')
    if user_id:
        try:
            user = UserModel.query.get(user_id)
            setattr(g, 'user', user) # 给全局变量 g 绑定一个 user 对象
            g.user = user # 功能同上，二选一即可
        except:
            pass
```

## 2. 上下文处理器

上下文处理器，是渲染模板后的操作，可以使用 Flask 中的 `@app.context_processor` 来实现此功能

```python
from flask import Flask, g

...

@app.context_processor
def context_processor():
    if hasattr(g, 'user'):
        return {'user': g.user}
    else:
        return {}
```

事件顺序：

发送请求 -> before_request -> 视图函数 -> 视图函数中返回模板 -> context_processor

# 八. 异常处理

实现自定义异常类，并通过 `@app.errorhandler()` 来捕捉对应的异常，实现全局异常处理

```python
from flask import Flask

class InvalidAPIUsage(Exception): # 自定义异常类，继承自 Exception
    status_code = 400 # 默认状态码
    
    def __init__(self, message, status_code=None, payload=None):
        super().__init__()
        self.message = message
        if status_code == None:
            self.status_code = status_code
        self.paylaod = payload
        
    def to_dict(self):
        rv = dict(self.payload or ())
        rv['message'] = self.message
        return rv
    
    
@app.errorhandler(InvalidAPIUsage) # 异常处理
def invalid_api_usage(e):
    return jsonify(e.to_dict())

@app.route('/test') # 接口
def test():
    user_id = request.args.get('user_id')
    
    if not user_id:
        raise InvalidAPIUsage('No User is Provided') # 提供异常信息
        
    user = get_user(user_id)
    if not user:
        raise InvalidAPIUsage("No such user!", status_code=404) # 提供异常信息与状态码
        
    return user
```

# 九. 基本结构

+ main.py - Flask 主程序
+ forms.py - 表单验证
+ config.py - 项目配置
+ static - 静态资源包
+ templates - 页面模板包
+ exception - 异常处理包
+ blueprints - 蓝图包
+ models - ORM包
