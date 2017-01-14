---
title: SQLAlchemy ORM教程之三：Relationship
date: 2016-10-30 18:44:07
tags: 'SQLAlchemy'
---
『关系』是关系型数据库的一大特色，也是我们在建模过程中的一个重要的抽象过程。在前面的两个教程中，我们分别回顾了使用`SQLAlchemy`在数据库中进行创建和简单查询的方法，今天我们来深入到更为复杂和抽象部分。
<!-- more -->

# 建立关系
之前我们已经建立了一个用户(User)表，现在我们来考虑增加一个与用户关联的新的表。在我们的系统里面，用户可以存储多个与之相关的email地址。这是一种基本的一对多的关系。我们把这个新增加的存储email地址的表称为addresses。应用Declarative，我们按照如下方式定义这个新表：
```Python
>>> from SQLAlchemy import ForeignKey
>>> from SQLAlchemy.orm import relationship

>>> class Address(Base):
...     __tablename__ = 'addresses'
...     id = Column(Integer, primary_key=True)
...     email_address = Column(String, nullable=False)
...     user_id = Column(Integer, ForeignKey('users.id'))
...
...     user = relationship("User", back_populates="addresses")
...
...     def __repr__(self):
...         return "<Address(email_address='%s')>" % self.email_address

>>> User.addresses = relationship(
...     "Address", order_by=Address.id, back_populates="user")
```
上面的代码中我们使用了一个新的名为ForeignKey的构造。其含义为，其所在的列的值域应当被限制在另一个表的指定列的取值范围之类。这一特性是关系型数据库的核心特性之一。就上例而言，addresses.user_id这一列的取值范围，应当包含在users.id的取值范围之内。

除了ForeignKey之外，我们还引入了一个relationship，来告诉ORM，Address类需要被连接到User类。relationship和ForeignKey这个两个属性决定了表之间关系的属性，决定了这个关系是多对一的。

在完成对Address类的声明之后，我们还定义另一个relationship，将其赋值给了User.addresses。在两个relationship中，我们都有传入了一个relationship.back_populates的属性来为反向关系所对应的属性进行命名。（作者：到这里为止，看来`SQLAlchemy`中定义关系要比Django的ORM要麻烦许多。Django中只需要一行就可以了。而且这里的两个relationship的定义明显是冗余的）

多对一的关系的反向永远都是一对多的关系。关于更多的relationship()的配置方法，可以参见这个链接Basic Relationship Patterns。

上述我们定义的两个互补的关系Address.user和User.addresses被称为双向关系(bidirectional relationship)，这是`SQLAlchemy`的核心特性这一。

relationship()的参数配置中指向被连接的类的字符串，可以指向工程中任何位置所定义的，基于declarative base的类，而无先后之分。Declarative会在完成所有的映射以后的将这些字符串转换为适当的、实际使用的参数形式。

# 使用关联对象
现在，当我们创建一个User实例的时候，会同时创建一个空的addresses的collection。这个collection可能是多种类型，如list, set, 或是dictionary。默认情况下，其应当为一个Python列表。
```Python
>>> jack = User(name='jack', fullname='Jack Bean', password='gjffdd')
>>> jack.addresses
[]
此时你可以自由的向这个列表里面插入User对象。
```Python
>>> jack.addresses = [
...                 Address(email_address='jack@google.com'),
...                 Address(email_address='j25@yahoo.com')]
```
当使用bidirectional relationship时，通过其中一个方向的关系（如上例）会自动出现在另一个方向的关系上。

```Python
>>> jack.addresses[1]
<Address(email_address='j25@yahoo.com')>

>>> jack.addresses[1].user
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
```

让我们把jack添加进入Session。
```Python
>>> session.add(jack)
>>> session.commit()
INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
('jack', 'Jack Bean', 'gjffdd')
INSERT INTO addresses (email_address, user_id) VALUES (?, ?)
('jack@google.com', 5)
INSERT INTO addresses (email_address, user_id) VALUES (?, ?)
('j25@yahoo.com', 5)
COMMIT
```

可以发现上面执行了三个INSERT命令，也就是说与jack关联的两个Address对象也被提交了。现在我们通过查询来取出jack。
```Python
>>> jack = session.query(User).\
... filter_by(name='jack').one()
BEGIN (implicit)
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users
WHERE users.name = ?
('jack',)

>>> jack
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
```

可以发现目前只有针对User表的查询，而没有对Address表的查询。此时访问addresses属性，相关的SQL才会执行
```Python
>>> jack.addresses
SELECT addresses.id AS addresses_id,
        addresses.email_address AS
        addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM addresses
WHERE ? = addresses.user_id ORDER BY addresses.id
(5,)
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

上面这种方式我们称之为lazy loading。

# 使用join进行查询
现在我们有了两会在那个彼此关联的数据表了，相比与上一篇教程中的简单查询情况，此时试图对这两张表进行联合查询就更加复杂一些了。关于join技术，读者可以自行阅读我的前一篇文章。

为了在User和Address之间构造一个简单的join，我们可以通过Query.filter()来连接其相关列（本质是隐式写法的JOIN）。下面是一个简单的例子：
```Python
>>> for u, a in session.query(User, Address).\
...                     filter(User.id==Address.user_id).\
...                     filter(Address.email_address=='jack@google.com').\
...                     all():
...     print(u)
...     print(a)
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
<Address(email_address='jack@google.com')>
```

而实际的SQL JOIN语法，可以通过Query.join()来想实现
```Python
>>> session.query(User).join(Address).\
...         filter(Address.email_address=='jack@google.com').\
...         all()
users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users JOIN addresses ON users.id = addresses.user_id
WHERE addresses.email_address = ?
('jack@google.com',)
[<User(name='jack', fullname='Jack Bean', password='gjffdd')>]
```

在上面的例子中由于只存在一个ForeignKey，Query.join知道如何选取合适的列进行JOIN。如果没有定义ForeignKey，或者存在多个，此时你需要手动指明你参与JOIN的列。Query.join()以如下方式进行：
```Python
query.join(Address, User.id==Address.user_id)    # explicit condition
query.join(User.addresses)                       # specify relationship from left to right
query.join(Address, User.addresses)              # same, with explicit target
query.join('addresses')
对于OUTER JOIN，只需要使用Query.outerjoin()就可以了。

query.outerjoin(User.addresses)   # LEFT OUTER JOIN
```

关于join()更为详细的用法，还是请参考官方的文档[join](http://docs.sqlalchemy.org/en/rel_1_0/orm/query.html#sqlalchemy.orm.query.Query.join)

# 使用Aliases
当你的查询涉及多个表，而其中同一个表出现了多次时，你需要的为重复的表aliase一个新的名字来避免冲突。这个功能其实我们在上一篇文章里面也提到过，下面是关于aliased的一个例子：
```Python
>>> from SQLAlchemy.orm import aliased
>>> adalias1 = aliased(Address)
>>> adalias2 = aliased(Address)
>>> for username, email1, email2 in \
...     session.query(User.name, adalias1.email_address, adalias2.email_address).\
...     join(adalias1, User.addresses).\
...     join(adalias2, User.addresses).\
...     filter(adalias1.email_address=='jack@google.com').\
...     filter(adalias2.email_address=='j25@yahoo.com'):
...     print(username, email1, email2)
SELECT users.name AS users_name,
        addresses_1.email_address AS addresses_1_email_address,
        addresses_2.email_address AS addresses_2_email_address
FROM users JOIN addresses AS addresses_1
        ON users.id = addresses_1.user_id
JOIN addresses AS addresses_2
        ON users.id = addresses_2.user_id
WHERE addresses_1.email_address = ?
        AND addresses_2.email_address = ?
('jack@google.com', 'j25@yahoo.com')
jack jack@google.com j25@yahoo.com
```

# 使用子查询(Subqueries)
Query适合于用来构造子查询。假如我们想要取出User记录，并且同时计算各个用户的Address的数量。产生这种功能的SQL指令最好的办法是按照user的id分组统计地址的数量，然后join到外层查询。此时我们需要LEFT JOIN，这样可以使得没有地址的用户也会出现在查询结果中（地址数量为0）。 我们期望的SQL命令是这样的：
```Python
SELECT users.*, adr_count.address_count FROM users LEFT OUTER JOIN
    (SELECT user_id, count(*) AS address_count
        FROM addresses GROUP BY user_id) AS adr_count
    ON users.id=adr_count.user_id
使用Query，我们可以从内到外来构造上面的语句。

>>> from SQLAlchemy.sql import func
>>> stmt = session.query(Address.user_id, func.count('*').\
...         label('address_count')).\
...         group_by(Address.user_id).subquery()
```

func我们已经在之前的教程中认识过了。subquery()可以产生一个内嵌了alias（是一个query.statement.alias()）的查询(SELECT)语句的表达。

当我们生成了statement之后，其完全可以视为一个Table来使用。你可以通过c来访问它的属性。
```Python
>>> for u, count in session.query(User, stmt.c.address_count).\
...     outerjoin(stmt, User.id==stmt.c.user_id).order_by(User.id):
...     print(u, count)
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password,
        anon_1.address_count AS anon_1_address_count
FROM users LEFT OUTER JOIN
    (SELECT addresses.user_id AS user_id, count(?) AS address_count
    FROM addresses GROUP BY addresses.user_id) AS anon_1
    ON users.id = anon_1.user_id
ORDER BY users.id
('*',)
<User(name='ed', fullname='Ed Jones', password='f8s7ccs')> None
<User(name='wendy', fullname='Wendy Williams', password='foobar')> None
<User(name='mary', fullname='Mary Contrary', password='xxg527')> None
<User(name='fred', fullname='Fred Flinstone', password='blah')> None
<User(name='jack', fullname='Jack Bean', password='gjffdd')> 2
```

## 从子查询中取出Entity
在前一个例子中，我们从子查询活着的是一个临时性的JOIN后的表，但是这个表并未定义我们在ORM中定义的Entity。如果我们想将这个临时表映射到ORM中的类呢？此时我们可以使用aliased这个函数来完成这个映射。
```Python
>>> stmt = session.query(Address).\
...                 filter(Address.email_address != 'j25@yahoo.com').\
...                 subquery()
>>> adalias = aliased(Address, stmt)
>>> for user, address in session.query(User, adalias).\
...         join(adalias, User.addresses):
...     print(user)
...     print(address)
SELECT users.id AS users_id,
            users.name AS users_name,
            users.fullname AS users_fullname,
            users.password AS users_password,
            anon_1.id AS anon_1_id,
            anon_1.email_address AS anon_1_email_address,
            anon_1.user_id AS anon_1_user_id
FROM users JOIN
    (SELECT addresses.id AS id,
            addresses.email_address AS email_address,
            addresses.user_id AS user_id
    FROM addresses
    WHERE addresses.email_address != ?) AS anon_1
    ON users.id = anon_1.user_id
('j25@yahoo.com',)
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
<Address(email_address='jack@google.com')>
```

# 使用EXISTS
EXISTS关键字是一个BOOL型操作符。当查询结果存在至少一行时返回True。EXISTS可以常常和JOIN搭配使用。

下面是一个显式的EXISTS构造方法：
```Python
>>> from SQLAlchemy.sql import exists
>>> stmt = exists().where(Address.user_id==User.id)
>>> for name, in session.query(User.name).filter(stmt):
...     print(name)
SELECT users.name AS users_name
FROM users
WHERE EXISTS (SELECT *
FROM addresses
WHERE addresses.user_id = users.id)
()
jack
Query还定义了若干个自动使用了EXISTS的操作。上面的例子可以用any()来完成：
```Python
>>> for name, in session.query(User.name).\
...         filter(User.addresses.any()):
...     print(name)
SELECT users.name AS users_name
FROM users
WHERE EXISTS (SELECT 1
FROM addresses
WHERE users.id = addresses.user_id)
()
jack
```

any()也接受筛选条件来限制匹配的行：
```Python
>>> for name, in session.query(User.name).\
...     filter(User.addresses.any(Address.email_address.like('%google%'))):
...     print(name)
jack
```

has()对于的many-to-one的关系，起到的是和any()同样的作用（注意这里~表示NOT）：
```Python
>>> session.query(Address).\
...         filter(~Address.user.has(User.name=='jack')).all()
[]
```

# 常用的关系操作
下面只是简单的列出了一些常用的操作。想要更为详细的了解这些功能，还是推荐去官网的相关文档。
```Python
eq() (many-to-one “equals” comparison):
query.filter(Address.user == someuser)
ne() (many-to-one “not equals” comparison):
query.filter(Address.user != someuser)
IS NULL (many-to-one comparison, also uses eq()):
query.filter(Address.user == None)
contains() (used for one-to-many collections):
query.filter(User.addresses.contains(someaddress))
any() (used for collections):
query.filter(User.addresses.any(Address.email_address == 'bar'))

# also takes keyword arguments:
query.filter(User.addresses.any(email_address='bar'))
has() (used for scalar references):
query.filter(Address.user.has(name='ed'))
Query.with_parent() (used for any relationship):
session.query(Address).with_parent(someuser, 'addresses')
```

# Eager Loading
前面的教程中我们有提及到lazing loading的机制。当我们通过查询取出用户时，与之关联的地址并没有取出来。当我们试图获取User.addresses时，相关的针对地址的SQL查询才起作用。如果你想要减少query的次数的话，就需要使用Eager Loading了。`SQLAlchemy`提供了三种Eager Loading的方式，其中两种是自动的，而第三种涉及到自定义的筛选条件。所有的这三种Eager Loading方式都会通过调用Query.options()来影响查询的过程，促使Query生成需要的额外配置来取出期望的内容。

## Subquery Loading
在上面的例子中，我们希望在 取出用户的时候就同步取出对应的地址。此时你们可以此采用orm.subqueryload()。这个函数可以发起第二个SELECT查询来取出与结果相关的另一个表的信息。这里取名为"subquery"的原因是，此处的Query在发起第二个查询时作为子查询而被复用了。详细过程参加下面的程序：
```Python
>>> from SQLAlchemy.orm import subqueryload
>>> jack = session.query(User).\
...                 options(subqueryload(User.addresses)).\
...                 filter_by(name='jack').one()
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users
WHERE users.name = ?
('jack',)
SELECT addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address,
        addresses.user_id AS addresses_user_id,
        anon_1.users_id AS anon_1_users_id
FROM (SELECT users.id AS users_id
    FROM users WHERE users.name = ?) AS anon_1
JOIN addresses ON anon_1.users_id = addresses.user_id
ORDER BY anon_1.users_id, addresses.id
('jack',)
>>> jack
<User(name='jack', fullname='Jack Bean', password='gjffdd')>

>>> jack.addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

注意：当subqueryload()和涉及limiting的函数一起使用的时候（如Query.first(), Query.limit(), Query.offset()等），应当加上一个以Unique的行作为参数的Query.order_by()来确保结果的正确性。详情参见The importance of Ordering

## Joined Load
这种自动Eager Loading的方式要更为常用一些。Joined Loading发起了一个JOIN（默认是LEFT OUTER JOIN），故而查询结果和制定的与之关联的行可以被同时取出。我们这里以和上面的Subquery Loading中同样的查询目的为例。
```Python
>>> from SQLAlchemy.orm import joinedload

>>> jack = session.query(User).\
...                        options(joinedload(User.addresses)).\
...                        filter_by(name='jack').one()
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password,
        addresses_1.id AS addresses_1_id,
        addresses_1.email_address AS addresses_1_email_address,
        addresses_1.user_id AS addresses_1_user_id
FROM users
    LEFT OUTER JOIN addresses AS addresses_1 ON users.id = addresses_1.user_id
WHERE users.name = ? ORDER BY addresses_1.id
('jack',)

>>> jack
<User(name='jack', fullname='Jack Bean', password='gjffdd')>

>>> jack.addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]
```

注意到，如果你是在命令行运行了前一个Subquery Loading的例子的话，在这里jack的addresses实际上已经填充了的，但是这里的Joined Load仍然是会发起JOIN。另外，LEFT OUTER JOIN指令实际上有可能导致重复的User出现，但是在结果中实际得到的User却不会重复。这是因为Query实际上是基于Object Identity采用了一种"uniquing"的策略。

历史上来看joinedload()出现的更早一些。joinedloading()更加适合于处理Many-to-one的关系。

## 显式的Join + EagerLoad
第三种方式我们是我们自己显式的调用join来定位JOIN连接主键，并接着关联表的信息填充到查询结果中对应对象或者列表中。这个特性需要使用到orm.contains_eager()函数。这个机制最典型的用途是pre-loading many-to-one关系，同时添加对这个关系的筛选。我们用下面的这个例子来阐述说明上面这些比较绕的话。假设我们需要筛选出用户的名字为jack的邮件地址，进行这个查询的方法如下：
```Python
>>> from SQLAlchemy.orm import contains_eager
>>> jacks_addresses = session.query(Address).\
...                             join(Address.user).\
...                             filter(User.name=='jack').\
...                             options(contains_eager(Address.user)).\
...                             all()
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password,
        addresses.id AS addresses_id,
        addresses.email_address AS addresses_email_address,
        addresses.user_id AS addresses_user_id
FROM addresses JOIN users ON users.id = addresses.user_id
WHERE users.name = ?
('jack',)

>>> jacks_addresses
[<Address(email_address='jack@google.com')>, <Address(email_address='j25@yahoo.com')>]

>>> jacks_addresses[0].user
<User(name='jack', fullname='Jack Bean', password='gjffdd')>

# 关系中的删除问题
沃恩尝试删除jack，来看结果：
```Python
>>> session.delete(jack)
>>> session.query(User).filter_by(name='jack').count()
UPDATE addresses SET user_id=? WHERE addresses.id = ?
((None, 1), (None, 2))
DELETE FROM users WHERE users.id = ?
(5,)
SELECT count(*) AS count_1
FROM (SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.password AS users_password
FROM users
WHERE users.name = ?) AS anon_1
('jack',)
0
那么与jack关联的地址呢？
```Python
>>> session.query(Address).filter(
...     Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
...  ).count()
2
```

地址记录仍然在这里。如果我们commit的话，我们可以从上面的SQL语句中发现，相关的Address的user_id属性被设置成了NULL。这不符合我们的要求。那么我们需要自己来设置关系的删除规则。

## 配置delete/delete-orphan Cascade
我们通过配置User.addresses关系的cascade*选项来控制删除行为。尽管`SQLAlchemy`允许你在任何时候给ORM添加属性或者关系。此时我们还是需要移除现存的关系并且重新开始（作者：django的ORM包含）。让我们首先关闭当前的session
```Python
>>> session.close()
```

并且使用一个新的declarative_base():
```Python
>>> Base = declarative_base()
```

下面我们重新声明User类，注意addresses中的配置：
```Python
>>> class User(Base):
...     __tablename__ = 'users'
...
...     id = Column(Integer, primary_key=True)
...     name = Column(String)
...     fullname = Column(String)
...     password = Column(String)
...
...     addresses = relationship("Address", back_populates='user',
...                     cascade="all, delete, delete-orphan")
...
...     def __repr__(self):
...        return "<User(name='%s', fullname='%s', password='%s')>" % (
...                                self.name, self.fullname, self.password)
接下来重新声明Address。
```Python
>>> class Address(Base):
...     __tablename__ = 'addresses'
...     id = Column(Integer, primary_key=True)
...     email_address = Column(String, nullable=False)
...     user_id = Column(Integer, ForeignKey('users.id'))
...     user = relationship("User", back_populates="addresses")
...
...     def __repr__(self):
...         return "<Address(email_address='%s')>" % self.email_address
```

现在让我们取出jack(下面我们使用了一个之前没有提到的函数get()，其参数为查询目标的主键)，现在从addresses中删除一个地址的话，会导致这个Address被删除。
```Python
# load Jack by primary key
SQL>>> jack = session.query(User).get(5)

# remove one Address (lazy load fires off)
SQL>>> del jack.addresses[1]

# only one address remains
SQL>>> session.query(Address).filter(
...     Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
... ).count()
1
删除jack也会导致剩下jack以及其所有的Address都会被删除:
```Python
>> session.delete(jack)

SQL>>> session.query(User).filter_by(name='jack').count()
0

SQL>>> session.query(Address).filter(
...    Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
... ).count()
0
```

关于更多的Cascade配置请参见官方文档。

# 建立多对多关系ManyToMany Relationship
现在我们需要引入一个新的模型来阐述多对多的关系了。假设我们需要完成一个博客应用。在这个应用里面我们可以书写BlogPost，每个博客都有若干Keyword。

对于一个多对多的关系，我们需要建立一个未映射的（也就是没有一个Python类与之对应的）表Table来作为中间联系的表。
```Python
>>> from SQLAlchemy import Table, Text
>>> # association table
>>> post_keywords = Table('post_keywords', Base.metadata,
...     Column('post_id', ForeignKey('posts.id'), primary_key=True),
...     Column('keyword_id', ForeignKey('keywords.id'), primary_key=True)
... )
```

不同于我们之前的典型的ORM方法，在上面的代码中我们直接声明了一个Table，而没有制定与之对应的Python类。Table是一个构造函数，其参数中的每个Colomn以逗号分隔。

下面我们来定义BlogPost和Keyword。我们这里需要使用relationship()在这两个类中定义一对互补的关系，其中每个关系的都指向post_keyword这个表。
```Python
>>> class BlogPost(Base):
...     __tablename__ = 'posts'
...
...     id = Column(Integer, primary_key=True)
...     user_id = Column(Integer, ForeignKey('users.id'))
...     headline = Column(String(255), nullable=False)
...     body = Column(Text)
...
...     # many to many BlogPost<->Keyword
...     keywords = relationship('Keyword',
...                             secondary=post_keywords,
...                             back_populates='posts')
...
...     def __init__(self, headline, body, author):
...         self.author = author
...         self.headline = headline
...         self.body = body
...
...     def __repr__(self):
...         return "BlogPost(%r, %r, %r)" % (self.headline, self.body, self.author)


>>> class Keyword(Base):
...     __tablename__ = 'keywords'
...
...     id = Column(Integer, primary_key=True)
...     keyword = Column(String(50), nullable=False, unique=True)
...     posts = relationship('BlogPost',
...                          secondary=post_keywords,
...                          back_populates='keywords')
...
...     def __init__(self, keyword):
...         self.keyword = keyword
```

在上面的定义中，我们可以发现和OneToMany关系不同，relationship()中多了一个secondary的参数，这个参数指向了中间表(原文为associated table)。这个中间表只包含了指向多对多关系两侧的表的主键的列。如果这个表包含了其他属性，甚至是自身的主键，`SQLAlchemy`需要你使用另一种，称为association object的机制来处理。

我们还希望我们的BlogPost能够拥有一个author属性，这个属性指向我们先前定义的User。此时我们需要再定义一个双向关系。由于一个作者可能拥有很多文章，我们希望访问User.posts的时候可以加以筛选而不是载入全部的相关文章。为此我们在定义User.posts中的时候，设置lazy='dynamic'，来控制载入策略。
```Python
>>> BlogPost.author = relationship(User, back_populates="posts")
>>> User.posts = relationship(BlogPost, back_populates="author", lazy="dynamic")
然后让我们来创建数据库中对应的表
```Python
>>> Base.metadata.create_all(engine)
PRAGMA...
CREATE TABLE keywords (
    id INTEGER NOT NULL,
    keyword VARCHAR(50) NOT NULL,
    PRIMARY KEY (id),
    UNIQUE (keyword)
)
()
COMMIT
CREATE TABLE posts (
    id INTEGER NOT NULL,
    user_id INTEGER,
    headline VARCHAR(255) NOT NULL,
    body TEXT,
    PRIMARY KEY (id),
    FOREIGN KEY(user_id) REFERENCES users (id)
)
()
COMMIT
CREATE TABLE post_keywords (
    post_id INTEGER NOT NULL,
    keyword_id INTEGER NOT NULL,
    PRIMARY KEY (post_id, keyword_id),
    FOREIGN KEY(post_id) REFERENCES posts (id),
    FOREIGN KEY(keyword_id) REFERENCES keywords (id)
)
()
COMMIT
多对多关系的使用方法道也没有太大的不同之处。让我们先来给windy添加博文。
```Python
>>> wendy = session.query(User).\
...                 filter_by(name='wendy').\
...                 one()
>>> post = BlogPost("Wendy's Blog Post", "This is a test", wendy)
>>> session.add(post)
```

给博文添加一些关键字。目前数据库里面还没有关键字存在，我们创建一些：
```Python
>>> post.keywords.append(Keyword('wendy'))
>>> post.keywords.append(Keyword('firstpost'))
我们可以开始查询了。先以'firstpost'为关键字来检索所有的博文。我们使用any来查询拥有关键词'firstpost'的博文：
```Python
>>> session.query(BlogPost).\
...             filter(BlogPost.keywords.any(keyword='firstpost')).\
...             all()
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', password='foobar')>)]
如果我们希望将查询范围限制在wendy用户所拥有的博文之内，
```Python
>>> session.query(BlogPost).\
...             filter(BlogPost.author==wendy).\
...             filter(BlogPost.keywords.any(keyword='firstpost')).\
...             all()
SELECT posts.id AS posts_id,
        posts.user_id AS posts_user_id,
        posts.headline AS posts_headline,
        posts.body AS posts_body
FROM posts
WHERE ? = posts.user_id AND (EXISTS (SELECT 1
    FROM post_keywords, keywords
    WHERE posts.id = post_keywords.post_id
        AND keywords.id = post_keywords.keyword_id
        AND keywords.keyword = ?))
(2, 'firstpost')
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', password='foobar')>)]
或者我们可以直接在wendy的posts属性上进行查询：
```Python
>>> wendy.posts.\
...         filter(BlogPost.keywords.any(keyword='firstpost')).\
...         all()
[BlogPost("Wendy's Blog Post", 'This is a test', <User(name='wendy', fullname='Wendy Williams', password='foobar')>)]
```
`SQLAlchemy`系列的教程到此告了一个小段落了，但是我觉得还是有很多说的不详细的地方。并且，教程中给出的用法和我在一些较为成熟的工程中所看到的`SQLAlchemy`的用法不是很一样，之后我会采用专题的形式来分话题详细阐述这些内容，喜欢的话like一下吧。
