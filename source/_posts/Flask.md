### cookies

客户端端的会话技术
cookie本身由浏览器保存，通过Response将cookie写到浏览器上，下一次访问，浏览器会根据不同的规则携带cookie过来

特点:
    - 客户端会话技术，浏览器的会话技术
    - 数据全都是存储在客户端中
    - 存储使用的键值对结构进行的存储
    - 特性
    - 支持过期时间
        - 默认会自动携带本网站的所有cookie
           - 根据域名进行cookie存储
             - 不能跨域名
             - 不能跨浏览器
        - Cookie是通过服务器创建的Response来创建的

 设置cookie:
  	response.set_cookie(key,value[,max_age=None,exprise=None)]
        max_age: 整数，指定cookie过期时间
        expries: 整数，指定过期时间，可以指定一个具体日期时间
        max_age和expries两个选一个指定

 获取cookie:
  	request.cookie.get(key)

 删除cookie
	response.delete_cookie(key)

### session

~~~html
服务器端会话技术,依赖于cookie
特点:
    - 服务端的会话技术
    - 所有数据存储在服务器中
    - 默认存储在内存中
        - django是默认做了数据持久化（存在了数据库中）
    - 存储结构也是key-value形势，键值对
    - session 是离不开cookie的

常用操作:
	设置session
  		session[‘key’] = ‘value’ 
        
  	获取session
  		session.get(key,default=None) 根据键获取会话的值
        
  	删除session
  		pop(key) 删除某一值  
  		clear()   清除所有
~~~

### Flask环境搭建

~~~html
1、在项目中创建一个python package，如App会自动生成一个init文件
2、将项目中的static和templates文件放到App中
3、在init文件中创建app
   如： def create_app():
        app =Flask(__name__)
        return app
4、将项目中的py文件改名为manage.py，并导入app，Manager，将app.run()改为manager
   如： app = create_app()
        manager = Manager(app)
        if __name__ == '__main__':
            manager.run()
5、在App中创建views.py,models.py,settings.py,exts.py文件
6、在views.py中创建蓝图
  如：  from flask import Blueprint
    	blue = Blueprint('blue',__name__)
（**如果蓝图不想再init中注册则可以直接再创建的蓝图下面自定义一个函数并注册蓝图，
	如：def init_blue(app):
    app.register_blueprint(blue)
）
*7、在init.py中注册蓝图
	如：app.register_blueprint(blue)
（**如果蓝图没有再init中注册则直接在这里导入6中注册蓝图的函数。如：init_exts(app)
8、在settings中进行配置
如mysql数据库配置：
def get_db_uri(dbinfo):
    db = dbinfo.get('DB')
    driver = dbinfo.get('DRIVER')
    user = dbinfo.get('USER')
    password = dbinfo.get('PASSWORD')
    host = dbinfo.get('HOST')
    port = dbinfo.get('PORT')
    name = dbinfo.get('NAME')
    return "{}+{}://{}:{}@{}:{}/{}".format(db, driver, user, password, host, port, name)
class Config:
    DEBUG = False
    TESTING = False
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SECRET_KEY = '110'

class DevelopConfig(Config):
    DEBUG = True
    DATABASE = {
        'DB': 'mysql',
        'DRIVER': 'pymysql',
        'USER': 'root',
        'PASSWORD': 'rock1204',
        'HOST': 'localhost',
        'PORT': '3306',
        'NAME': 'testflask3',
    }
    
env = {
    'develop': DevelopConfig,
    'test': TestingConfig,
    'product': ProductConfig,
}
9、在init文件中加载setting配置信息
如：app.config.from_object(env.get('develop'))
10、在exts导入第三方插件（exts文件用于导入和初始化第三方插件）
如：在init中的Migrate，models中的Migrate
    from flask_migrate import Migrate
    from flask_sqlalchemy import SQLAlchemy
    #自定义第三方插件
    db = SQLAlchemy()
    migrate = Migrate()
    #初始化第三方插件
    def init_exts(app):
        db.init_app(app)
        migrate.init_app(app=app,db=db)
11、在init删除已经导入exts中的第三发插件，同时初始化第三方插件
在init删除已经导入exts中的第三发插件，同时初始化第三方插件
如：    init_exts(app)
12、RESTful
将views改为apis,导入Resource
from flask_restful import Resource
如:class Hello(Resource):
    def get(self):
        return {'msg':'GET请求'}
    def post(self):
        return {'msg':'Post请求'} #返回值是json格式的数据
13、App内新建一个urls用来设置访问路径,需要导入apis内的函数
如: from App.apis import *
    from App.exts import api
    api.add_resource(Hello,'/hello/')(其中Hello为apis函数,/hello/为路径)


~~~

### Flask创建数据库

~~~html
一、创建sql数据库
1、在models.py中设置db
from flask_sqlalchemy import SQLAlchemy
db = SQLAlchemy()
2、在init.py中配置数据库和初始化orm
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///sqlite3.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db.init_app(app=app)
3、在models中设置table（表格名：__tablename__ = 'People'）
4、在views.py中设置函数
如：@blue.route（‘/createdb/’）
def create_db():
	db.create_all()  #创建table（db.drop_all(),删除table）
	return 'success'
5、在浏览器中运行，可以App文件夹中看见又sqlite3.db文件生产
6、添加数据
如：p= Person()
p.name = 'xxx' + str(random.randeange(10.100))
p.age = random.randrange(10,100)
db.session.add(p)
db.session.commit()
return 'success'
7、获取数据
persons = Person.query.all()
8、删除数据
	p = Person.query.first()  # 获取第一条数据
    db.session.delete(p)
    db.session.commit()
    
9、修改数据
    p = Person.query.first()
    p.age = 100
    db.session.commit()
二、创建mysql数据库
app.config['SQLALCHEMY_DATABASE_URI'] = dialect+driver://username:password@host:port/database
DB_URI=‘mysql+pymsql://{}:{}@{}:{}/{}’.format(
  USERNAME,用户名
  PASSWORD,密码
  HOSTNAME,主机
  PORT,端口
  DATABASE)数据库名
三、数据迁移
安装
	pip install flask-migrate
    
初始化
	使用app和db进行migrate对象初始化
    	from flask_migrate import Migrate
		migrate = Migrate()
    	migrate.init_app(app=app, db=db)
    
    安装了flask-script后，可以在manager上添加迁移指令
    	from flask_migrate import MigrateCommand
		manager.add_command('db', MigrateCommand)

python manager.py db init  只调用一次, 这里的db是添加命令时给定的名称
python manager.py db migrate  生成迁移文件
python manager.py db upgrade  执行迁移中的升级
python manager.py db downgrade  执行迁移中的降级
~~~

### 字段格式化

```html
fields进行定义
marshal_with进行使用
	特性
        显示我们设计的数据结构
        默认返回的数据如果在预定义结构中不存在，数据会被自动过滤
        如果返回的数据在预定义的结构中存在，数据会正常返回
        如果返回的数据比预定义结构中的字段少，预定义的字段会呈现一个默认值
        
	定义字段输出
        使用字典进行定义
        常用都是基本类型: String, Integer
        
            # 格式化字段
            user_fields = {
                'msg': fields.String,
                'status': fields.Integer,
                'data': fields.String(attribute='private_data'),
                'default_data': fields.String(default='1')
            }

    	定义好的格式通过装饰器进行使用
            @marshal_with(需要返回的数据格式),  return返回字典就ok了

                class Users(Resource):
                    @marshal_with(user_fields)
                    def get(self):
                        return {'msg':'呵呵', 'data':'没有数据', 'age':'22', 'private_data':'表中数据'}


	级联数据: 嵌套字典
		Nested
        
            # 格式化字段
            usermodel_fileds = {
                'id': fields.Integer,
                'name': fields.String,
            }

            user2_fields = {
                'msg': fields.String(default='ok'),
                'status': fields.Integer(default=1),
                'data': fields.Nested(usermodel_fileds)
            }
        
	结构允许嵌套列表
		fields.List(fields.Nested) 
        
        	# 格式化字段
            usermodel_fileds = {
                'id': fields.Integer,
                'name': fields.String,
            }
            users3_fields = {
                'status': fields.String(default=1),
                'msg': fields.String,
                'data': fields.List(fields.Nested(usermodel_fileds))
            }
```

### URL

~~~html
连接字段
    就是将当前数据的操作api暴露出来
    根据提供的url和唯一标识进行数据操作

# 格式化字段
usermodel_fileds = {
    'id': fields.Integer,
    'name': fields.String,
    'url': fields.Url('id', absolute=True)
}

# 在add_resource中提供对应的endpoint
api.add_resource(Users4, '/user4/', endpoint='id')
~~~





### 参数解析

```html
可以不通过request.form或request.args获取参数, 而是通过reqparse.RequestParser来解析

    # 参数转换器
    parser = reqparse.RequestParser()
    parser.add_argument('name', type=str, action='append')  # 支持多个name
    parser.add_argument('id', type=int, required=True, help='id是必须的') # 必需参数
    parser.add_argument('fldt_active', type=str, location='cookies')  # 获取cookies中的数据

    class Users4(Resource):
        def get(self):
            parse = parser.parse_args()
            user_name = parse.get('name')
            id = parse.get('id')
            fldt_active = parse.get('fldt_active')
            return {'name': user_name, 'id': id, 'fldt_active':fldt_active}
    
```

### 导入json文件内数据

```html
1.制定表格执行迁移
2.从json文件中获取表格内数据(**eval(),json.load()的使用)
3.导入pymysql将获取的数据导入表格内
如:goods.py文件
# 获取商品数据
import pymysql
from flask import jsonify

def get_goods_data():
	#先连接数据库
    db = pymysql.Connect(host='localhost',port=3306,user = 'root',password='rock1204',database='testflask3',charset='utf8')
	#设置游标
    cursor = db.cursor()
	#打开文件
    with open('goods.json','r') as fp:
        goods = eval(fp.read())
		#事务开启
		db.begin()
        for good in goods:
            gid = good.get('gid')
            name = good.get('name')
            price = good.get('price')
            unit = good.get('unit')
            headImg = good.get('headImg')
			#执行
            cursor.execute('insert into goods(gid,name,price,unit,headImg) values("%d","%s","%d","%s","%s")'%(gid,name,price,unit,headImg))
           
            #提交事务
            db.commit()
		#关闭游标
        cursor.close()
		#结束事务
        db.close()
if __name__ == '__main__':
    get_goods_data()
```

```html
发送邮箱验证:
1.exts导入Mail,cache
2.setting中配置邮箱MAIL_SERVER="smtp.163.com",MAIL_USERNAME='zkr744@163.com',MAIL_PASSWORD = '123abc'
3.给指定邮箱发送邮件
# 临时存储5分钟, 请在5分钟之内激活
            cache.set(user.token, user.id, timeout=300)
# 给指定的邮箱发送邮件
            msg = Message(subject='淘票票邮箱激活', sender='niejeff@163.com', recipients=[email])
            # 邮箱内容
            msg.html = render_template('email_active.html', username=username, active_url='http://127.0.0.1:8002/accountactive/?u_token=%s' % user.token)
            # 发送
            mail.send(msg)
4.激活邮箱
# 获取当前u_token临时存储userid
        userid = cache.get(u_token)
        if userid:
            user = UserModel.query.get(userid)
            user.is_active = True  # 激活
            db.session.commit()

            cache.delete(u_token)  # 删除token

密码加密与解密
1.导入generate_password_hash()函数
如:user.passwd = generate_password_hash(password)  # 加密
密码解密
1.导入check_password_hash()函数
如:check_password_hash(user.passwd, password),两个参数,第一个是密文,第二个是输入的密码,判断两个是否相同.

用户权限:abort





```

