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

```shell
pip install flask-sqlalchemy
```

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

Flask-SQLAlchemy 连接 SQL Server 时有可能会出现<font color=red>**“未发现数据源名称并且未指定默认驱动程序 ”**</font>的异常，不能正确连接数据库，异常信息：

> sqlalchemy.exc.InterfaceError: (pyodbc.InterfaceError) ('IM002', '[IM002] [Microsoft][ODBC 驱动程序管理器] 未发现数据源名称并且未指定默认驱动程序 (0) (SQLDriverConnect)')

导致此状况可能是因为没有指定驱动程序导致，只需要在 URI 后添加驱动参数即可解决

```python
DB_URI = 'mssql+pyodbc://{}:{}@{}:{}/{}?driver=SQL+Server+Native+Client+11.0'.format(USERNAME, PASSWORD, HOST_NAME, PORT, DATABASE)
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

## 4. 一对一关系实现

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

## 5. 操作汇总

```python
# 原生sql语句操作
sql = 'select * from user'
result = db.session.execute(sql)

# 查询全部
User.query.all()

# 主键查询
User.query.get(1)

# 条件查询
User.query.filter_by(User.username='name')
res = User.query.filter(User.username=='john').first()

# 多条件查询
from sqlalchemy import and_
User.query.filter_by(and_(User.username =='name',User.password=='passwd'))

# 比较查询
User.query.filter(User.id.__lt__(5)) # 小于5
User.query.filter(User.id.__le__(5)) # 小于等于5
User.query.filter(User.id.__gt__(5)) # 大于5
User.query.filter(User.id.__ge__(5)) # 大于等于5

# in查询
User.query.filter(User.username.in_('A','B','C','D'))

# 排序
User.query.order_by('age') # 按年龄排序，默认升序，在前面加-号为降序'-age'

# 限制查询
User.query.filter(age=18).offset(2).limit(3)  # 跳过二条开始查询，限制输出3条

# 增加
use = User(id,username,password)
db.session.add(use)
db.session.commit() 

# 删除
User.query.filter_by(User.username='name').delete()

# 修改
User.query.filter_by(User.username='name').update({'password':'newdata'})
```

## 6. 数据类型

| SQLAlchemy类型名 |    Python类型     |                     说明                     |
| :--------------: | :---------------: | :------------------------------------------: |
|     Integer      |        int        |              普通整数，一般32位              |
|   SmallInteger   |        int        |          取值范围小的整数，一般16位          |
|    BigInteger    |    int 或 long    |               不限制精度的整数               |
|      Float       |       float       |                    浮点数                    |
|      String      |        str        |                  变长字符串                  |
|     Boolean      |       bool        |                    布尔值                    |
|       Date       |   datetime.date   |                     日期                     |
|       Time       |   datetime.time   |                     时间                     |
|     DateTime     | datetime.datetime |                  日期和时间                  |
|       Text       |        str        | 变长字符串，对较长或不限长度的字符串做了优化 |
|     Numeric      |  decimal.Decimal  |                   定点小数                   |

## 7. 列选项

|   选项名    |                          说明                           |
| :---------: | :-----------------------------------------------------: |
| primary_key |             如果设为True，这列就是表的主键              |
|   unique    |             如果设为True，这列不允许重复值              |
|    index    |       如果设为True，为该列创建索引，提升查询效率        |
|  nullable   | 如果设为True，这列允许null值，如果设为false，不允许为空 |
|   default   |                    为这列定义默认值                     |

## 8. 存储过程

对于 SQL Server 数据库存储过程，如需查看执行结果（即：存储过程有 SELECT 结果），需要添加`SET NOCOUNT ON`语句，否则无法获取执行结果

```python
sql = 'SET NOCOUNT ON; exec query_pinxi_list;';
connection = db.engine.raw_connection() 
try:
    cursor = connection.cursor()
    cursor.execute(sql)
    results = list(cursor.fetchall())
    cursor.close()
finally:
    connection.close()
```

# 六. Flask-WTF 表单验证

Flask-WTF 是简化了 WTForms 操作的一个第三方库，WTForms 表单的两个主要功能是验证用户提交数据的合法性以及渲染模板，当然还包括 CORF 保护、文件上传等

```shell
pip install flask-wtf
```

## 1. 字段验证

现有 forms.py 文件，在内部创建 RegistForm 的注册验证表单

```python
from flask_wtf import FlaskForm
import wtforms
from wtforms.validators import length, email

class LoginForm(FlaskForm):
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

# 九. 项目基本结构

+ main.py - Flask 主程序
+ forms.py - 表单验证
+ config.py - 项目配置
+ static - 静态资源包
+ templates - 页面模板包
+ exception - 异常处理包
+ blueprints - 蓝图包
+ models - ORM包

# 十. Swagger接口文档

Swagger支持多种语言，可以[点击此处](https://swagger.io/tools/open-source/open-source-integrations/)查询其支持语言

Swagger 对 Flask 具有良好的支持，有 Flasgger、flask-swagger等框架，在此处使用的的是 Flasgger，[点击此处](https://github.com/flasgger/flasgger)查看 Flasgger github 仓库，内含多种实例及相关文档

## 1. 安装相关依赖

```shell
pip install flasgger
```

## 2. 接口文档地址

如在本地测试，则可以访问http://localhost:5000/apidocs，查看接口文档内容

## 3. 使用配置

### 3.1 注册启用 flasgger

在主程序中，进行如下操作：

```python
from flask import Flask
from flasgger import Swagger

app = Flask(__name__)
Swagger(app)
```

即可启用 Swagger，蓝图文件中不需要再重复引入，只在主文件进行上述设置即可

### 3.2 使用

```python
def test():
    """
      接口描述
      ---
      tags:
        - 接口 tag
      parameters:
        - name: 参数名1
          in: query/form/path/...
          type: string/integer/file/...
          required: true/false
          description: 参数描述
        - name: 参数名2
          ...
      responses:
        200: # 状态码
    """
```

#### 3.2.1 描述参数

```yaml
parameters:
- in: path
  name: userId
  schema:
    type: integer
  required: true
  description: 根据id得到用户
```

因为传递参数是数组类型，所以在yaml中要在每个参数前使用 - 短线符号标注

##### 3.2.1.1 参数种类

根据参数位置的不同将参数进行了分类，而参数的位置则是根据参数的 <font color=red>in</font> 字段来控制，比如：`in: query` 和 `in: path`，常见的参数位置（种类）有如下几种：

- path 参数

    路径参数，参数通过url传递，例如 `/users/{id}`

- query 参数

    （字符）查询参数，参数通过URL字段传递，常见于get方式提交，例如`/users?role=admin`

- formData 参数

    表单查询参数，一般是通过 post 方式提交的表单

- header 参数

    头部信息传递参数，参数在请求头中，例如`X-MyHeader: Value`

- cookie 参数

    cookie 传递参数，参数附加在 cookie 中，例如`Cookie: debug=0; csrftoken=BUSe35dohU3O1MZvDCU`

#### 3.2.2 数据类型

基本数据类型有：

- string（包含了日期和文件类型）
- number
- integer
- boolean
- array
- object

同时支持混合数据类型，但是直接使用 array 来表达混合类型是错误的，正确的应该是使用 `oneOf` 或者 `anyOf` 来表达混合类型，比如：

```yaml
oneOf:
  - type: string
  - type: integer
```

对于文件，可以使用 `string` 来表达

```yaml
type: string
format: binary # 二进制文件

type: string
format: byte # base64编码的文件
```

#### 3.2.3 引用定义 `$ref`

通常一个资源会被多个接口使用，因此可以创建一个代码片段，以便在需要时多次使用，使用 `$ref` 来引用定义，示例：

在一 yaml 文件中有如下定义：

```yaml
componts:
  schemas:
    User:
      properties:
        id:
          type: integer
        name:
          type: string
```

在返回体中需要引用上述定义，则可以通过下述方法：

```yaml
responses:
  200:
    description: 回复体
    schema:
      $ref: '#/componts/schemas/User'
```

## 4. 完整示例

```yaml
tags:
  - '车辆筛选API'
summary: '提交车辆配置'

parameters:
  - name: date
    in: formData
    type: string
    required: True
    description: '查询月份'
    schema:
      example: '202205'
  - name: type
    in: formData
    type: integer
    required: False
    descrption: '类型'
    scheml:
      example: 1
      
definitions:
  item:
    type: string
    description: '数据体'
    
  resp_body:
    type: object
    description: '返回信息'
    properties:
      status_code:
        type: integer
        descritpion: '状态码'
      message:
        type: string
        description: '提示信息'
      data:
        type: query
        items:
          $ref: '#/definitions/item'
    example:
      status_code: 200
      message: '操作成功'
      data: ['string1', 'string2']

responses:
  200:
    description: 'OK'
    schema:
      $ref: '#/definitions/resp_body'
```

