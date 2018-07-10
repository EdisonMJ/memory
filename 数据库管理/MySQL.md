## Pymysql

### 1. 连接数据库

在这里我们首先尝试连接一下数据库，假设当前的 MySQL运行在本地，用户名为 root，密码为 123456，运行端口为 3306，在这里我们利用 PyMySQL 先连接一下 MySQL 然后创建一个新的数据库，名字叫做 spiders，代码如下：

```python
import pymysql

db = pymysql.connect(host='localhost',user='root', password='123456', port=3306)
cursor = db.cursor()
cursor.execute('SELECT VERSION()')
data = cursor.fetchone()
print('Database version:', data)
cursor.execute("CREATE DATABASE spiders DEFAULT CHARACTER SET utf8")
db.close()
```

运行结果：

```python
Database version: ('5.6.22',)
```

在这里我们通过 PyMySQL 的 connect() 方法声明了一个 MySQL 连接对象，需要传入 MySQL 运行的 host 即 IP，此处由于 MySQL 在本地运行，所以传入的是 localhost，如果 MySQL 在远程运行，则传入其公网 IP 地址，然后后续的参数 user 即用户名，password 即密码，port 即端口默认 3306。

连接成功之后，我们需要再调用 cursor() 方法获得 MySQL 的操作游标，利用游标来执行 SQL 语句，例如在这里我们执行了两句 SQL，用 execute() 方法执行相应的 SQL 语句即可，第一句 SQL 是获得 MySQL 当前版本，然后调用fetchone() 方法来获得第一条数据，也就得到了版本号，另外我们还执行了创建数据库的操作，数据库名称叫做 spiders，默认编码为 utf-8，由于该语句不是查询语句，所以直接执行后我们就成功创建了一个数据库 spiders，接着我们再利用这个数据库进行后续的操作。

### 2. 创建表

一般来说上面的创建数据库操作我们只需要执行一次就好了，当然我们也可以手动来创建数据库，以后我们的操作都是在此数据库上操作的，所以后文介绍的 MySQL 连接会直接指定当前数据库 spiders，所有操作都是在 spiders 数据库内执行的。

所以这里MySQL的连接就需要额外指定一个参数 db。

然后接下来我们新创建一个数据表，执行创建表的 SQL 语句即可，创建一个用户表 students，在这里指定三个字段，结构如下：

| 字段名 | 含义 | 类型    |
| ------ | ---- | ------- |
| id     | 学号 | varchar |
| name   | 姓名 | varchar |
| age    | 年龄 | int     |

创建表的示例代码如下：

```python
import pymysql

db = pymysql.connect(host='localhost', user='root', password='123456', port=3306, db='spiders')
cursor = db.cursor()
sql = 'CREATE TABLE IF NOT EXISTS students (id VARCHAR(255) NOT NULL, name VARCHAR(255) NOT NULL, age INT NOT NULL, PRIMARY KEY (id))'
cursor.execute(sql)
db.close()
```

运行之后我们便创建了一个名为 students 的数据表，字段即为上文列举的三个字段。

当然在这里作为演示我们指定了最简单的几个字段，实际在爬虫过程中我们会根据爬取结果设计特定的字段。

### 3. 插入数据

我们将数据解析出来后的下一步就是向数据库中插入数据了，例如在这里我们爬取了一个的学生信息，学号为 20120001，名字为 Bob，年龄为 20，那么如何将该条数据插入数据库呢，实例代码如下：

```python
import pymysql

id = '20120001'
user = 'Bob'
age = 20

db = pymysql.connect(host='localhost', user='root', password='123456', port=3306, db='spiders')
cursor = db.cursor()
sql = 'INSERT INTO students(id, name, age) values(%s, %s, %s)'
try:
    cursor.execute(sql, (id, user, age))
    db.commit()
except:
    db.rollback()
db.close()
```

在这里我们首先构造了一个 SQL 语句，其 Value 值我们没有用字符串拼接的方式来构造，如：

```python
sql = 'INSERT INTO students(id, name, age) values(' + id + ', ' + name + ', ' + age + ')'
```

这样的写法繁琐而且不直观，所以我们选择直接用格式化符 %s 来实现，有几个 Value 写几个 %s，我们只需要在 execute() 方法的第一个参数传入该 SQL 语句，Value 值用统一的元组传过来就好了。

这样的写法有既可以避免字符串拼接的麻烦，又可以避免引号冲突的问题。

之后值得注意的是，需要执行 db 对象的 commit() 方法才可实现数据插入，这个方法才是真正将语句提交到数据库执行的方法，对于数据插入、更新、删除操作都需要调用该方法才能生效。

接下来我们加了一层异常处理，如果执行失败，则调用rollback() 执行数据回滚，相当于什么都没有发生过一样。

在这里就涉及一个事务的问题，事务机制可以确保数据的一致性，也就是这件事要么发生了，要么没有发生，比如插入一条数据，不会存在插入一半的情况，要么全部插入，要么整个一条都不插入，这就是事务的原子性，另外事务还有另外三个属性，一致性、隔离性、持久性，通常成为 ACID 特性。

归纳如下：

| 属性                  | 解释                                                         |
| --------------------- | ------------------------------------------------------------ |
| 原子性（atomicity）   | 一个事务是一个不可分割的工作单位，事务中包括的诸操作要么都做，要么都不做。 |
| 一致性（consistency） | 事务必须是使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的。 |
| 隔离性（isolation）   | 一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。 |
| 持久性（durability）  | 持续性也称永久性（permanence），指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响。 |

插入、更新、删除操作都是对数据库进行更改的操作，更改操作都必须为一个事务，所以对于这些操作的标准写法就是：

```python
try:
    cursor.execute(sql)
    db.commit()
except:
    db.rollback()
```

这样我们便可以保证数据的一致性，在这里的 commit() 和 rollback() 方法就是为事务的实现提供了支持。

好，在上面我们了解了数据插入的操作，是通过构造一个 SQL 语句来实现的，但是很明显，这里有一个及其不方便的地方，比如又加了一个性别 gender，假如突然增加了一个字段，那么我们构造的 SQL 语句就需要改成：

```mysql
INSERT INTO students(id, name, age, gender) values(%s, %s, %s, %s)
```

相应的元组参数则需要改成：

```
(id, name, age, gender)
```

这显然不是我们想要的，在很多情况下，我们要达到的效果是插入方法无需改动，做成一个通用方法，只需要传入一个动态变化的字典给就好了。比如我们构造这样一个字典：

```python
{
    'id': '20120001',
    'name': 'Bob',
    'age': 20
}
```

然后 SQL 语句会根据字典动态构造，元组也动态构造，这样才能实现通用的插入方法。所以在这里我们需要将插入方法改写一下：

```python
data = {
    'id': '20120001',
    'name': 'Bob',
    'age': 20
}
table = 'students'
keys = ', '.join(data.keys())
values = ', '.join(['%s'] * len(data))
sql = 'INSERT INTO {table}({keys}) VALUES ({values})'.format(table=table, keys=keys, values=values)
try:
   if cursor.execute(sql, tuple(data.values())):
       print('Successful')
       db.commit()
except:
    print('Failed')
    db.rollback()
db.close()
```

在这里我们传入的数据是字典的形式，定义为 data 变量，表名也定义成变量 table。接下来我们就需要构造一个动态的 SQL 语句了。

首先我们需要构造插入的字段，id、name 和 age，在这里只需要将data的键名拿过来，然后用逗号分隔即可。所以 ', '.join(data.keys()) 的结果就是 id, name, age，然后我们需要构造多个 %s 当作占位符，有几个字段构造几个，比如在这里有两个字段，就需要构造 %s, %s, %s ，所以在这里首先定义了长度为 1 的数组 ['%s'] ，然后用乘法将其扩充为 ['%s', '%s', '%s']，再调用 join() 方法，最终变成 %s, %s, %s。所以我们再利用字符串的 format() 方法将表名，字段名，占位符构造出来，最终sql语句就被动态构造成了：

```mysql
INSERT INTO students(id, name, age) VALUES (%s, %s, %s)
```

最后再 execute() 方法的第一个参数传入 sql 变量，第二个参数传入 data 的键值构造的元组，就可以成功插入数据了。

如此以来，我们便实现了传入一个字典来插入数据的方法，不需要再去修改 SQL 语句和插入操作了。

### 4. 更新数据

数据更新操作实际上也是执行 SQL 语句，最简单的方式就是构造一个 SQL 语句然后执行：

```python
sql = 'UPDATE students SET age = %s WHERE name = %s'
try:
   cursor.execute(sql, (25, 'Bob'))
   db.commit()
except:
   db.rollback()
db.close()
```

在这里同样是用占位符的方式构造 SQL，然后执行 excute() 方法，传入元组形式的参数，同样执行 commit() 方法执行操作。

如果要做简单的数据更新的话，使用此方法是完全可以的。

但是在实际数据抓取过程中，在大部分情况下是需要插入数据的，但是我们关心的是会不会出现重复数据，如果出现了重复数据，我们更希望的做法一般是更新数据而不是重复保存一次，另外就是像上文所说的动态构造 SQL 的问题，所以在这里我们在这里重新实现一种可以做到去重的做法，如果重复则更新数据，如果数据不存在则插入数据，另外支持灵活的字典传值。

```python
data = {
    'id': '20120001',
    'name': 'Bob',
    'age': 21
}

table = 'students'
keys = ', '.join(data.keys())
values = ', '.join(['%s'] * len(data))

sql = 'INSERT INTO {table}({keys}) VALUES ({values}) ON DUPLICATE KEY UPDATE'.format(table=table, keys=keys, values=values)
update = ','.join([" {key} = %s".format(key=key) for key in data])
sql += update
try:
    if cursor.execute(sql, tuple(data.values())*2):
        print('Successful')
        db.commit()
except:
    print('Failed')
    db.rollback()
db.close()
```

在这里构造的 SQL 语句其实是插入语句，但是在后面加了 ON DUPLICATE KEY UPDATE，这个的意思是如果主键已经存在了，那就执行更新操作，比如在这里我们传入的数据 id 仍然为 20120001，但是年龄有所变化，由 20 变成了 21，但在这条数据不会被插入，而是将 id 为 20120001 的数据更新。

在这里完整的 SQL 构造出来是这样的：

```mysql
INSERT INTO students(id, name, age) VALUES (%s, %s, %s) ON DUPLICATE KEY UPDATE id = %s, name = %s, age = %s
```

相比上面介绍的插入操作的 SQL，后面多了一部分内容，那就是更新的字段，ON DUPLICATE KEY UPDATE 使得主键已存在的数据进行更新，后面跟的是更新的字段内容。所以这里就变成了 6 个 %s。所以在后面的 execute() 方法的第二个参数元组就需要乘以 2 变成原来的 2 倍。

如此一来，我们就可以实现主键不存在便插入数据，存在则更新数据的功能了。

### 5. 删除数据

删除操作相对简单，使用 DELETE 语句即可，需要指定要删除的目标表名和删除条件，而且仍然需要使用 db 的 commit() 方法才能生效，实例如下：

```python
table = 'students'
condition = 'age > 20'

sql = 'DELETE FROM  {table} WHERE {condition}'.format(table=table, condition=condition)
try:
    cursor.execute(sql)
    db.commit()
except:
    db.rollback()

db.close()
```

在这里我们指定了表的名称，删除条件。因为删除条件可能会有多种多样，运算符比如有大于、小于、等于、LIKE等等，条件连接符比如有 AND、OR 等等，所以不再继续构造复杂的判断条件，在这里直接将条件当作字符串来传递，以实现删除操作。

### 6. 查询数据

说完插入、修改、删除等操作，还剩下非常重要的一个操作，那就是查询。

在这里查询用到 SELECT 语句，我们先用一个实例来感受一下：

```python
sql = 'SELECT * FROM students WHERE age >= 20'

try:
    cursor.execute(sql)
    print('Count:', cursor.rowcount)
    one = cursor.fetchone()
    print('One:', one)
    results = cursor.fetchall()
    print('Results:', results)
    print('Results Type:', type(results))
    for row in results:
        print(row)
except:
    print('Error')
```

运行结果：

```python
Count: 4
One: ('20120001', 'Bob', 25)
Results: (('20120011', 'Mary', 21), ('20120012', 'Mike', 20), ('20120013', 'James', 22))
Results Type: <class 'tuple'>
('20120011', 'Mary', 21)
('20120012', 'Mike', 20)
('20120013', 'James', 22)
```

在这里我们构造了一条 SQL 语句，将年龄 20 岁及以上的学生查询出来，然后将其传给 execute() 方法即可，注意在这里不再需要 db 的 commit() 方法。然后我们可以调用 cursor 的 rowcount 属性获取查询结果的条数，当前示例中获取的结果条数是 4 条。

然后我们调用了 fetchone() 方法，这个方法可以获取结果的第一条数据，返回结果是元组形式，元组的元素顺序跟字段一一对应，也就是第一个元素就是第一个字段 id，第二个元素就是第二个字段 name，以此类推。随后我们又调用了fetchall() 方法，它可以得到结果的所有数据，然后将其结果和类型打印出来，它是二重元组，每个元素都是一条记录。我们将其遍历输出，将其逐个输出出来。

但是这里注意到一个问题，显示的是4条数据，fetall() 方法不是获取所有数据吗？为什么只有3条？这是因为它的内部实现是有一个偏移指针来指向查询结果的，最开始偏移指针指向第一条数据，取一次之后，指针偏移到下一条数据，这样再取的话就会取到下一条数据了。所以我们最初调用了一次 fetchone() 方法，这样结果的偏移指针就指向了下一条数据，fetchall() 方法返回的是偏移指针指向的数据一直到结束的所有数据，所以 fetchall() 方法获取的结果就只剩 3 个了，所以在这里要理解偏移指针的概念。

所以我们还可以用 while 循环加 fetchone() 的方法来获取所有数据，而不是用 fetchall() 全部一起获取出来，fetchall() 会将结果以元组形式全部返回，如果数据量很大，那么占用的开销会非常高。所以推荐使用如下的方法来逐条取数据：

```python
sql = 'SELECT * FROM students WHERE age >= 20'
try:
    cursor.execute(sql)
    print('Count:', cursor.rowcount)
    row = cursor.fetchone()
    while row:
        print('Row:', row)
        row = cursor.fetchone()
except:
    print('Error')
```

这样每循环一次，指针就会偏移一条数据，随用随取，简单高效。

## MySQL

### 1. 初始化

```bash
# 初始化数据库
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

# 设置密码
mysqladmin -u root password "newpass"

# 修改密码, 登录mysql后, 输入
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
```

### 2. 运行服务

```bash
/usr/bin/mysqld_safe --datadir='/var/lib/mysql'
```

### 3. 导入数据

```bash
# 1.创建数据库
create database <数据库名>;
# 2.切换数据库
use <数据库名>;
# 3.设置数据库编码
set names utf8;
# 4.导入
mysql>source /home/abc/abc.sql;
```

### 4. 命令

```mysql
# 创建数据库
create database <数据库名>;
# 显示可用数据库
show database; 
# 切换数据库
use <数据库名>;
# 设置数据库编码
set names utf8;
# 创建表
create table <表名>;
# 显示可用表
show tables; 
# 显示指定表信息
describe <表名>; 
```

