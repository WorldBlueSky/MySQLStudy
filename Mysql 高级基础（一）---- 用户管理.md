#  一、用户管理



##  0、登陆mysql服务器



登陆的命令

```bash
mysql -h localhost -P 3306 -p dbname -e "select * from dbname"
```



> -h 代表连接mysql的主机ip地址，如果没有写，默认是 localhost 127.0.0.1



> -P 大写的P,代表开启的端口号，默认是3306



> -p 小写的p,在下一行输入密码进入mysql



> dataname , 参数直接写一个具体的数据库，输入密码验证之后选中该数据库



> -e "statement"  跟在数据库名字的后面，输入密码之后，立刻执行e后面的 statement 语句，然后退出mysql



![1658237613266](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658237613266.png)



我们通常怎么登陆呢？

```bash
mysql -uroot -p
```

只需要输入 -uroot,-p在下一行输入即可，其他为默认选项



## 1、创建用户



增加一个数据库用户，此时创建的用户只能登陆，没有任何的权限，暂时不提权限

```mysql
create user 'username' identified by 'password'
```



![1658238225587](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658238225587.png)



默认 host 为 %，支持任意ip地址连接

如果需要进行指定的话需要在 用户名后面指定 @ ''可连接的主机号''

![1658238398556](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658238398556.png)



&emsp;&emsp;发现 'rain7666' 这个user 还是创建成功了，说明user不是主键，其实 user、host 是联合主键，只有user、host 的信息都相同这时插入不成功。



**建议在创建 用户的时候 把 @"连接ip" 指定清楚**



## 2、登陆其他用户



```bash
mysql -u'username' -p
```



**在命令行上写密码非常不建议，有风险，所以建议直接-p,回车在下一行输入密码，所有字符均被隐藏**



![1658238687085](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658238687085.png)



查看当前用户所有的数据库

![1658238795491](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658238795491.png)

发现与root用户相比，myql、sys、performance_schema 数据库根本就查看不到了，权限受到限制。



 新创建的用户，默认情况下是没有任何权限的。

 

![1658239127795](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658239127795.png)



没有权限创建数据库等一系列操作



查看用户权限



```bash
show grants # 查看当前用户授予的权限
```



![1658239188515](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658239188515.png)



没有任何权限，只能用于登陆而已



## 3、修改用户



暂时不提权限的问题



修改用户名, 就是修改 mysql 库中的 user 表中对应的信息

```mysql
update mysql.user set user='user' where user='原用户名' and host='原host';
```



修改完之后，必须刷新 权限，否则修改无效

```mysql
flush privileges;
```



修改 zhangsan 的用户名为 wangwu



![1658242833303](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658242833303.png)



退出mysql，用 wangwu 的信息登陆,登陆成功



![1658242879438](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658242879438.png)





## 4、删除用户



### 方式一（推荐）



drop 命令 删除用户，不需要flush



```mysql
drop user 'username'@'host'; #  如果不加 @ 后面的host ,那么默认删除host=% 的用户
```



![1658241896696](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658241896696.png)



### 方式二，在user表中删除用户



```mysql
delete from user where use='' and host=''
```

&emsp;&emsp;

&emsp;&emsp;如果只是在user表中删除了用户信息的话，系统会有残留信息保留，而drop 命令会删除用户以及其对应的权限，mysql.user 表中的记录和 mysql.db 中的记录都删除了。



## 5、修改密码



## （1）修改当前用户的密码  



### 第一种方式 set



首先以修改密码的用户进行登陆，登陆之后输入以下命令

```mysql
set password = 'yourpassword';
```



### 第二种方式 alter



修改当前user 的密码

```mysql 
alter user user() identified by 'yourpassword';
```



&emsp;&emsp;我们修改过的密码对应的其实是 mysql.user表中的 authentication_string 字段，但是如果只是 用 update 改这个字段的话，那么肯定登陆会不成功的，也会修改密码的操作会在数据库中的其他文件中有记录，不只是修改了一个user表中的字段



我们输入的密码其实在数据库中又经过加密的

![1658299262689](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658299262689.png)





## （2）修改其他用户的密码



&emsp;&emsp;有的用户权限比较大可以修改其他用户的密码，比如说root用户，那么提供了以下两种方式来修改其他用户的密码。



### **方式一: alter**



```mysql 
alter user 'username'@'host' identified by 'yourpassword'
```



root 用户修改 wangwu 用户的密码为 abc



![1658299447396](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658299447396.png)



wangwu 使用 密码abc 登陆成功

![1658299682250](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658299682250.png)



### **方式二 :set password**



```mysql 
set password for 'username'@'host'= 'yourpassword'
```



这个命令也很好理解，从字义上来看就是 设置 'username'@'host' 用户的密码为 'yourpassword' 



root 用户设置 wangwu@localhost 用户的密码为123456

![1658299861811](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658299861811.png)



wangwu 用户使用密码123456登陆mysql成功

![1658299903201](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658299903201.png)





