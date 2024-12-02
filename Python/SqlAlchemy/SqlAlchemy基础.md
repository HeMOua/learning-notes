# SqlAlchemy 基础
## 安装依赖
```shell
pip install sqlalchemy
```
## 入门

### 初始化 DBSession
1. 创建基类 Base：对于同一个数据库中的对象，比如继承同一个 Base，如果需要管理不同数据库，需要使用 `declarative_base()` 创建多个 Base 实例。
2. 声明实体类：创建实体类，需要继承 Base，编写实体对应的字段
3. 初始化数据库连接：通过 `create_engine()` 初始化数据库连接
4. 创建DBSession类型：用于创建 Session 对象
```python
# 导入:
from sqlalchemy import Column, String, create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.orm import declarative_base

# 创建对象的基类:
Base = declarative_base()

# 定义User对象:
class User(Base):
    # 表的名字:
    __tablename__ = 'user'

    # 表的结构:
    id = Column(String(20), primary_key=True)
    name = Column(String(20))

# 初始化数据库连接:
engine = create_engine('mysql+mysqlconnector://root:password@localhost:3306/test')
# 创建DBSession类型:
DBSession = sessionmaker(bind=engine)
```

### 数据库操作
1. 创建 Session
2. 创建数据对象
3. 通过 Session 执行相关操作（增删改查）
4. 通过 commit 提交操作
5. 关闭 Session

**（1）添加**
```python
# 创建session对象:
session = DBSession()
# 创建新User对象:
new_user = User(id='5', name='Bob')
# 添加到session:
session.add(new_user)
# 提交即保存到数据库:
session.commit()
# 关闭session:
session.close()
```
**（2）查询**
```python
# 创建Session:
session = DBSession()
# 创建Query查询，filter是where条件，最后调用one()返回唯一行，如果调用all()则返回所有行:
user = session.query(User).filter(User.id=='5').one()
# 打印类型和对象的name属性:
print('type:', type(user))
print('name:', user.name)
# 关闭Session:
session.close()
```
### 表关联
```python
class User(Base):
    __tablename__ = 'user'

    id = Column(String(20), primary_key=True)
    name = Column(String(20))
    # 一对多:
    books = relationship('Book')

class Book(Base):
    __tablename__ = 'book'

    id = Column(String(20), primary_key=True)
    name = Column(String(20))
    # “多”的一方的book表是通过外键关联到user表的:
    user_id = Column(String(20), ForeignKey('user.id'))
```
当我们查询一个User对象时，该对象的books属性将返回一个包含若干个Book对象的list。
