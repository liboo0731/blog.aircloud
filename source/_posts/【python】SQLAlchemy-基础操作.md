---
title: 【python】SQLAlchemy-入门模板
date: 2018-05-04 10:41:19
tags:
- python
- sqlalchemy
---

### SQLAlchemy-入门模板

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# DB_CONNECT_STRING 就是连接数据库的路径。“mysql+mysqldb”指定了使用 MySQL-Python 来连接
DB_CONNECT_STRING = 'mysql+mysqldb://root:mysql@localhost/test?charset=utf8'
# echo 参数为 True 时，会显示每条执行的 SQL 语句，生产环境下可关闭
engine = create_engine(DB_CONNECT_STRING, echo=True)
DB_Session = sessionmaker(bind=engine)
session = DB_Session()


from sqlalchemy import Column
from sqlalchemy.types import CHAR, Integer, String
from sqlalchemy.ext.declarative import declarative_base

BaseModel = declarative_base()
def init_db():
    BaseModel.metadata.create_all(engine)
def drop_db():
    BaseModel.metadata.drop_all(engine)

class User(BaseModel):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(CHAR(30)) # or Column(String(30))

init_db()

```



### SQLAlchemy-CRUD基础

```python


from sqlalchemy import func, or_, not_

user = User(name='a')
session.add(user)
user = User(name='b')
session.add(user)
user = User(name='a')
session.add(user)
user = User()
session.add(user)
session.commit()
query = session.query(User)
print query # 显示SQL 语句
print query.statement # 同上
for user in query: # 遍历时查询
    print user.name
print query.all() # 返回的是一个类似列表的对象
print query.first().name # 记录不存在时，first() 会返回 None
# print query.one().name # 不存在，或有多行记录时会抛出异常
print query.filter(User.id == 2).first().name
print query.get(2).name # 以主键获取，等效于上句
print query.filter('id = 2').first().name # 支持字符串
query2 = session.query(User.name)
print query2.all() # 每行是个元组
print query2.limit(1).all() # 最多返回 1 条记录
print query2.offset(1).all() # 从第 2 条记录开始返回
print query2.order_by(User.name).all()
print query2.order_by('name').all()
print query2.order_by(User.name.desc()).all()
print query2.order_by('name desc').all()
print session.query(User.id).order_by(User.name.desc(), User.id).all()
print query2.filter(User.id == 1).scalar() # 如果有记录，返回第一条记录的第一个元素
print session.query('id').select_from(User).filter('id = 1').scalar()
print query2.filter(User.id > 1, User.name != 'a').scalar() # and
query3 = query2.filter(User.id > 1) # 多次拼接的 filter 也是 and
query3 = query3.filter(User.name != 'a')
print query3.scalar()
print query2.filter(or_(User.id == 1, User.id == 2)).all() # or
print query2.filter(User.id.in_((1, 2))).all() # in
query4 = session.query(User.id)
print query4.filter(User.name == None).scalar()
print query4.filter('name is null').scalar()
print query4.filter(not_(User.name == None)).all() # not
print query4.filter(User.name != None).all()
print query4.count()
print session.query(func.count('*')).select_from(User).scalar()
print session.query(func.count('1')).select_from(User).scalar()
print session.query(func.count(User.id)).scalar()
print session.query(func.count('*')).filter(User.id > 0).scalar() # filter() 中包含 User，因此不需要指定表
print session.query(func.count('*')).filter(User.name == 'a').limit(1).scalar() == 1 # 可以用 limit() 限制 count() 的返回数
print session.query(func.sum(User.id)).scalar()
print session.query(func.now()).scalar() # func 后可以跟任意函数名，只要该数据库支持
print session.query(func.current_timestamp()).scalar()
print session.query(func.md5(User.name)).filter(User.id == 1).scalar()
query.filter(User.id == 1).update({User.name: 'c'})
user = query.get(1)
print user.name
user.name = 'd'
session.flush() # 写数据库，但并不提交
print query.get(1).name
session.delete(user)
session.flush()
print query.get(1)
session.rollback()
print query.get(1).name
query.filter(User.id == 1).delete()
session.commit()
print query.get(1)

```



### SQLAlchemy-非ORM操作

#### 创建表格，初始化数据库

```
>>> users_table = Table('users', metadata,
...     Column('id', Integer, primary_key=True),
...     Column('name', String(40)),
...     Column('email', String(120)))
>>>
>>> users_table.create()
2014-01-09 10:03:32,436 INFO sqlalchemy.engine.base.Engine
CREATE TABLE users (
    id INTEGER NOT NULL,
    name VARCHAR(40),
    email VARCHAR(120),
    PRIMARY KEY (id)
)       
2014-01-09 10:03:32,436 INFO sqlalchemy.engine.base.Engine ()
2014-01-09 10:03:32,575 INFO sqlalchemy.engine.base.Engine COMMIT
```

##### 基本操作，插入

如果已经table表已经存在， 第二次运行就不许要 create了， 使用 autoload 设置

```
>>> from sqlalchemy import *
>>> from sqlalchemy.orm import *
>>> engine = create_engine('sqlite:///./sqlalchemy.db', echo=True)
>>> metadata = MetaData(engine)
>>> users_table = Table('users', metadata, autoload=True)
2014-01-09 10:20:01,580 INFO sqlalchemy.engine.base.Engine PRAGMA table_info("users")
2014-01-09 10:20:01,581 INFO sqlalchemy.engine.base.Engine ()
2014-01-09 10:20:01,582 INFO sqlalchemy.engine.base.Engine PRAGMA foreign_key_list("users")
2014-01-09 10:20:01,583 INFO sqlalchemy.engine.base.Engine ()
2014-01-09 10:20:01,583 INFO sqlalchemy.engine.base.Engine PRAGMA index_list("users")
2014-01-09 10:20:01,583 INFO sqlalchemy.engine.base.Engine ()
>>> users_table
Table('users', MetaData(bind=Engine(sqlite:///./sqlalchemy.db)), Column('id', INTEGER(), table=<users>, primary_key=True, nullable=False), Column('name', VARCHAR(length=40), table=<users>), Column('email', VARCHAR(length=120), table=<users>), schema=None)
>>>
```

