#Python 数据库实战


#SQLite
SQLite是一种嵌入式数据库，它的数据库就是一个文件。由于SQLite本身是C写的，而且体积很小，所以，经常被集成到各种应用程序中，甚至在iOS和Android的App中都可以集成。

Python就内置了SQLite3，所以，在Python中使用SQLite，不需要安装任何东西，直接使用。

##SQLite 创建数据库并添加数据
```
# -*- coding:utf-8 -*-

import os,sqlite3

db_file = os.path.join(os.path.dirname(__file__),'test.db')

if os.path.isfile(db_file):
    os.remove(db_file)

# 初始数据

# 连接到SQLite数据库
# 数据库文件是test.db
# 如果文件不存在，会自动在当前目录创建:
conn = sqlite3.connect('test.db')
# 创建一个Cursor:
cursor = conn.cursor()
# 执行一条SQL语句，创建user表:
cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')
# 继续执行一条SQL语句，插入一条记录:
cursor.execute('insert into user (id, name) values (\'1\', \'Michael\')')
cursor.execute('insert into user (id, name) values (\'2\', \'Kenvin\')')
cursor.execute('insert into user (id, name) values (\'3\', \'Snake\')')

# 通过rowcount获得插入的行数:
print('rowcount =', cursor.rowcount)
# 关闭Cursor:
cursor.close()
# 提交事务:
conn.commit()
# 关闭Connection:
conn.close()
```

##SQLite  查询记录

```
# 查询记录：
conn = sqlite3.connect('test.db')
cursor = conn.cursor()
# 执行查询语句:
cursor.execute('select * from user where id=?', '1')
# 获得查询结果集:
values = cursor.fetchall()
print(values)
cursor.close()
conn.close()
```
使用```Python```的```DB-API```时，只要搞清楚```Connection```和```Cursor```对象，打开后一定记得关闭，就可以放心地使用。

使用```Cursor```对象执行```insert```，```update```，```delete```语句时，执行结果由```rowcount```返回影响的行数，就可以拿到执行结果。

使用```Cursor```对象执行```select```语句时，通过```featchall()```可以拿到结果集。结果集是一个```list```，每个元素都是一个```tuple```，对应一行记录。

##SQLite  小结

在```Python```中操作数据库时，要先导入数据库对应的驱动，然后，通过```Connection```对象和```Cursor```对象操作数据。

要确保打开的```Connection```对象和```Cursor```对象都正确地被关闭，否则，资源就会泄露。

如何才能确保出错的情况下也关闭掉```Connection```对象和```Cursor```对象呢？请回忆try:...except:...finally:...的用法。



##MySQL 


##安装mysql
按照安装步骤安装即可,安装时会出现如下弹框,一定要记住,5.7之后的版本默认有个临时的密码:如下图 
![](http://img.blog.csdn.net/20170801113905727?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3VhbmdkYWNhaWt1YW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
e8Awkbiyqf)# 就是临时密码 

##启动MySQL服务器:偏好设置,点击mysql 

![](http://img.blog.csdn.net/20170801114042077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3VhbmdkYWNhaWt1YW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##首次点击start MySQL server 会出现输入电脑密码,输入自己的电脑密码即可 

![](http://img.blog.csdn.net/20170801114219885?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3VhbmdkYWNhaWt1YW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##服务器启动 
![](http://img.blog.csdn.net/20170801114333280?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3VhbmdkYWNhaWt1YW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##修改临时密码:

```
/usr/local/mysql/bin/mysqladmin -u root -p password

yaoyajiedeMacBook-Pro:~ Kenvin$  /usr/local/mysql/bin/mysqladmin -u root -p password
Enter password: 输入刚才弹出的临时密码
New password: 新密码
Confirm new password: 确认新密码
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.
yaoyajiedeMacBook-Pro:~ Kenvin$  
```

##navicat和mysql关联

![屏幕快照 2017-09-06 下午3.27.14](http://ovsiiuil2.bkt.clouddn.com/2017-09-06-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-09-06%20%E4%B8%8B%E5%8D%883.27.14.png)

接下来就是创建表和字段，最后结果如下


![数据库](http://ovsiiuil2.bkt.clouddn.com/2017-09-06-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-09-06%20%E4%B8%8B%E5%8D%883.11.33.png)


```

# -*- coding: utf-8 -*-
import pymysql

# 创建连接
conn = pymysql.connect(host='127.0.0.1', port=3306, user='root', passwd='123456', db='shop', charset='utf8')

# 创建游标, 查询数据默认为元组类型
cursor = conn.cursor()

# 执行SQL，并返回受影响行数（使用pymysql的参数化语句防止SQL注入）
row1 = cursor.executemany("insert into shop( shopid,shopname, shopaddress,shoptel)values( %s,%s, %s,%s)", [( 1021,'锦江乐园', '古美路132楼13单元','13651981343'), (1022,'迪士尼乐园', '静安区132楼13单元','13651981343')])
print(row1)

# 执行SQL，并返回受影响行数,执行多次
effect_row = cursor.executemany("insert into shop(shopid,shopname,shopaddress,shoptel)values(%s,%s,%s,%s)", [(1024,"卢一路232号旺铺","大雨路家岔口","13285073566"),(1025,"卢一路232号旺铺","进贤路232号13弄","13285073588")])
print(effect_row)


# 提交，不然无法保存新建或者修改的数据
conn.commit()
# 关闭游标
cursor.close()
# 关闭连接
conn.close()
```

