---
title: 'SQLAlchemy ORM教程之一：Create '
date: 2016-09-27 18:36:23
tags: 'SQLAlchemy'
---

所谓`ORM（Object Relational Mapping）`，就是建立其由`Python`类到数据库表的映射关系：一个`Python`实例(`instance`)对应数据库中的一行(`row`)。

这种映射包含两层含义:
- 实现对象和与之关联的的行的状态同步
- 二是将涉及数据库的查询操作，表达为Python类的相互关系
<!-- more -->
注意`ORM`和`SQLAlchemy`的`Expression Language`不同。

后者可以视为对原始SQL的封装。ORM是基于Expression Language而构建的，其抽象层次要高于Expression Language。很多时候我们都是使用ORM，有时需要一些高度定制化的功能时，就需要使用到Expression Language。

下面就是教程的正文了。

**Version Check**
注意这里的教程都是基于1.0版本的，你可以同下面的命令来查看SQLAlchemy的版本：
```python
>>> import sqlalchemy
>>> sqlalchemy.__version___
```

# Connecting

在这个教程中我们使用`in-memory`的SQLite数据库，你也可以根据自己的需要配置对应的数据库设置。为了建立同数据库的链接，我们需要使用到create_engine
```python
>>> from sqlalchemy import create_engine
>>> engine = create_engine('sqlite:///:memory:', echo=True)
```
这里的echo设置为True可以使得后面我们可以在控制台看到操作涉及的SQL语言。如果你觉得麻烦，可以将其设置为False。我们这里就不贴出了。

create_engine返回的是一个Engine实例，它代表了指向数据库的一些非常核心的接口。他会根据你选择的数据库配置而调用对应的DBAPI。

当第一次如Engine.execute()或者Engine.connect()的方法被调用时，Engine才会真正的建立起到数据库的DBAPI连接。实际上，我们一般并不会直接使用Engine。

# Declaring a Mapping

当我们使用ORM的时候，其配置过程主要分为两个部分：一是描述我们要处理的数据库表的信息，二是将我们的Python类映射到这些表上。这两个过程在SQLAlchemy中是一起完成的，我们将这个过程称之为Declarative。

使用Declarative参与ORM映射的类需要被定义成为一个指定基类的子类，这个基类应当含有ORM映射中相关的类和表的信息。这样的基类我们称之为declarative base class。在我们的应用中，我们一般只需要一个这样的基类。这个基类我们可以通过declarative_base来创建
```python
>>> from sqlalchemy.ext.declarative import declarative_base

>>> Base = declarative_base()
```
现在我们已经有了一个基类，我们可以基于这个基类来创建我们的自定义类了。我们以建立一个用户类为例子。从Base派生一个名为User的类，在这个类里面我们可以定义将要映射到数据库的表上的属性（主要是表的名字，列的类型和名称等）：
```python
>>> from sqlalchemy import Column, Integer, String
>>> class User(Base):
...     __tablename__ = 'users'
...
...     id = Column(Integer, primary_key=True)
...     name = Column(String)
...     fullname = Column(String)
...     password = Column(String)
...
...     def __repr__(self):
...        return "<User(name='%s', fullname='%s', password='%s')>" % (
...                             self.name, self.fullname, self.password)
```
通过Declarative生成的类至少应该包含一个名为tablename的属性来给出目标表的名称，以及至少一个Column来给出表的主键(Primary Key)。SQLAlchemy不会对于类名和表名之间的关联做任何假设，也不会自动涉及数据类型以及约束的转换。一般的你可以自己创建一个模板来建立这些自动转换，这样可以减少你的很多重复劳动。

当我们的类声明完成后，Declarative将会将所有的Column成员替换成为特殊的Python访问器(accessors)，我们称之为descriptors。这个过程我们称为instrumentation，经过instrumentation的映射类可以让我们能够读写数据库的表和列。

注意除了这些涉及ORM的映射意外，这些mapping类的其他部分仍然是不变的。

# Create a schema
我们通过Declarative系统构建好我们的User类之后，与之同时的关于表的信息也已经创建好了，我们称之为table metadata。描述这些信息的类为Table。我们可以通过__table__这个类变量来查看表信息
```python
>>> User.__table__ 
Table('users', MetaData(bind=None),
            Column('id', Integer(), table=<users>, primary_key=True, nullable=False),
            Column('name', String(), table=<users>),
            Column('fullname', String(), table=<users>),
            Column('password', String(), table=<users>), schema=None)
```
当我们完成类声明时，Declarative用一个Python的metaclass来为这个类进行了加工。在这个阶段，它依据我们给出的设置创建了Table对象，然后构造一个Mapper对象来与之关联。这些幕后的对象我们大多都不需要直接与之打交道。

Table对象是一个更大家庭----我们称之为MetaData----的一部分。当我们使用Declarative时，这个对象也可以在Declarative base class的.metadata属性中看到。

MetaData是我们与数据库打交道的一个接口。对于我们的SQLite数据库而言，此时还没有一个名为users的表的存在，我们需要使用MetaData来发出CREATE TABLE的命令。下面我们使用MetaData.create_all()指令，将我们上面得到的Engine作为参数传入。如果你上面设置了echo为True的话，应该可以看到这一过程中的SQL指令。首先检查了users表的存在性，如果不存在的话会执行表的创建工作。
```python
>>> Base.metadata.create_all(engine)
SELECT ...
PRAGMA table_info("users")
()
CREATE TABLE users (
    id INTEGER NOT NULL, name VARCHAR,
    fullname VARCHAR,
    password VARCHAR,
    PRIMARY KEY (id)
)
()
COMMIT
```
# Create an Instance of the Mapped Class
创建User对象十分简单
```python
>>> ed_user = User(name='ed', fullname='Ed Jones', password='edspassword')
>>> ed_user.name
'ed'
>>> ed_user.password
'edspassword'
>>> str(ed_user.id)
'None'
```
# Create a Session
Session是一个非常重要的概念，类似于iOS中的NSManagedContext的概念，我也在尝试进一步去理解它。

我们现在可以和数据库对话了。ORM对数据库的入口即是Session，当我们构建应用时，和create_engine的同一级别下，我们定义一个Session类来作为生成新的Session的Factory类
```python
>>> from sqlalchemy.orm import sessionmaker
>>> Session = sessionmaker(bind=engine)
```
当你试图在定义Engine之前定义Sesssion的话，这里的bind可以不设置
```python
>>> Session = sessionmaker()
```
后续你定义好Engine后可以通过configure()来将其连接到Session
```python
>>> Session.configure(bind=engine)  # once engine is available
```
这个我们自定义的工厂类就可以拿来我们构造新的Session了。
```python
session = Session()
```
上面的Session已经和我们的SQLite的数据库的Engine关联起来了，但是我们可以发现它还没有打开任何到数据库的连接(connection)。当一个Session被首次使用时，它会从Engine所维护的连接池中取出一个连接来操作数据库。这个连接在我们应用有所更改或者关闭Session时会被释放。

# Adding and Update Objects
为了将User对象存入数据库，我们调用Sesson的add()函数
```python
>>> ed_user = User(name='ed', fullname='Ed Jones', password='edspassword')
>>> session.add(ed_user)
```
当这个操作完成之后，我们成这个User实例的状态为pending。目前实际上还没有执行SQL操作，也就是说数据库中还没有产生和这个User实例对应的行。Session将会在需要的时候执行相应的SQL命令，这个过程我们称之为flush。如果我们试图查询Ed Jones，所有处于pending状态的信息将会首先被flush，然后负责进行查询的SQL语言在此之后立即被执行。

例如，我们创建一个查询来获取刚刚我们创建的用户（涉及查询的部分我们后续会详细介绍）。这个查询会返回一个和我们之前添加的用户相同的用户实例。
```python
>>> our_user = session.query(User).filter_by(name='ed').first() BEGIN (implicit)
INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
('ed', 'Ed Jones', 'edspassword')
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users
WHERE users.name = ?
 LIMIT ? OFFSET ?
('ed', 1, 0)
>>> our_user
<User(name='ed', fullname='Ed Jones', password='edspassword')>
```
事实上这里的Session判断出来了需要返回的行和已经存在内存中的一个映射实例应当是同一个，所以我们会得到一个和之前完全相同的实例
```python
>>> ed_user is our_user
True
```
这里ORM所表现的理念，我们称之为identity map。这个设计理念保证了在一个Session对于一个制定行的操作，作用于同一个内存实例上。当一个拥有特定主键的对象出现在Session中时，所有的查询操作对这个主键都会返回一个相同的Python对象。并且，如果你试图引入重复了主键的新的对象时，系统会产生一个错误来阻止你的操作。

我们可以通过add_all()来一次加入多个对象
```python
>>> session.add_all([
...     User(name='wendy', fullname='Wendy Williams', password='foobar'),
...     User(name='mary', fullname='Mary Contrary', password='xxg527'),
...     User(name='fred', fullname='Fred Flinstone', password='blah')])
并且，如果我们希望改变Ed的密码，可以直接修改之：

>>> ed_user.password = 'f8s7ccs'
这个修改会被Session记录下来

>>> session.dirty
IdentitySet([<User(name='ed', fullname='Ed Jones', password='f8s7ccs')>])
当然，上面的插入操作也被记录了

>>> session.new 
IdentitySet([<User(name='wendy', fullname='Wendy Williams', password='foobar')>,
<User(name='mary', fullname='Mary Contrary', password='xxg527')>,
<User(name='fred', fullname='Fred Flinstone', password='blah')>])
我们可以使用commit()命令来将这些更改flush到数据库中。

>>> session.commit()
```

