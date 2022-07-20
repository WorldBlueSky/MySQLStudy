

# Mysql 基础（二）---- 权限管理



权限管理就是 限制你操作数据库的权力，不可以越界，防止你误操作一些数据从删库到跑路。 

比如说只允许你有select 操作某个数据库中所有表的权限，那么你就不能delete、insert 等操作。



## 1、权限认识 show privileges



可以通过 mysql查看所有的权限

```mysql
show privileges
```



查看所有权限



![1658304247456](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658304247456.png)



简单的来说，可以概括成以下几类

（1）create、drop 权限，可以创建新的数据库或表，可移除已有的数据库或表  

（2）select、insert、delete、update 权限，允许在一个数据库中的表上进行操作

（3）alter 可以使用 alter table 来个别更改表的结构以及重新命名

（4）index 权限，允许创建或者删除索引，index适用于已存在的表

（5）create routine 创建可保存的程序，alter routine 更改或删除保存的程序，excute 用来执行保存的程序

（6）grant 权限用来 允许授权给其他用户

（7）file 权限使用户可使用sql语句读取或者写入 服务器上的文件（可以读任何数据库目录下的文件，因为服务器可访问这些文件）



## 2、授予权限的原则 



处于数据库的安全考虑，遵循以下几个经验原则



> 1、只授予满足用户使用的最小权限，防止用户干坏事，从删库到跑路



> 2、创建用户的时候限制用户登录的主机，可限制成内网ip或指定ip，防止外网ip破解用户密码进行登录



> 3、为用户创建满足一定复杂度的密码，防止被密码破解拿到权限删除数据



> 4、定期清理不需要的用户，清理离职员工不用的用户账号，回收权限或者删除用户





## 3、查看权限 show grant



查看当前用户的所有权限

```mysql
show grants;
```



查看某个用户的所有权限

```mysql
show grants for 'user'@'host';
```



![1658319380221](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658319380221.png)





## 4、授予权限 grant



&emsp;&emsp;授予权限有两种方式，一种是角色赋予用户进行授权，一种是直接给用户授予权限，这里角色还没说到，先介绍直接在数据库中对用户授予权限。



```mysql
grant  权限1，权限2，权限3 on 指定数据库.指定表 to '用户名'@'主机地址'
```



授予权限的时候如果发现没有这个用户，那么会直接新创建一个用户



在root登陆的用户中，授予 'wangwu'@'localhost' 用于 data_DB 中所有表 select 的权限

![1658307734989](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658307734989.png)



使用 wangwu 用户登录，select成功查询

![1658307877376](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658307877376.png)



给 user表中插入一条数据，insert 命令被拒绝，只有select 的权限



![1658307921094](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658307921094.png)



查看当前用户的权限，只有select 的权限

![1658308119557](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658308119557.png)



### 权限追加



&emsp;&emsp;如果我们还想给 用户授予权限的话，那么还是使用grant 命令授予权限，此时的授予不是覆盖，是在原来的基础上继续进行追加权限。



root 用户 给wangwu 继续追加 insert 的权限

![1658308273413](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658308273413.png)



wangwu 用户使用 insert 插入数据，此时插入成功



![1658308443825](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658308443825.png)



查看当前wanguw 的权限



![1658308482780](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658308482780.png)



### 授予所有权限



```mysql
grant all privileges on *.* to '用户名'@'主机';
```



登陆 root 用户 授予 wangwu@localhost 所有的权限

![1658318405304](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658318405304.png)

 

登陆 wangwu 用户，查看权限

![1658318539062](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658318539062.png)



那么此时的 wangwu ,拿到了root 授予的全部权限，那么此时wangwu 等同于 root吗？

我们再来查看root的权限，进行对比，发现root的权限还是比wangwu多一项， with grant option,root 拥有给其他用户授权的能力，但是wangwu不能授权。

![1658318573783](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658318573783.png)



如果给 指定用户 授予 可以给其他用户授予权限的 功能呢？



- 给指定用户授权的时候参数加上 with grant option 即可，表示该用户可以将自己拥有的权限授权给他人。



 在开发的时候要根据用户需求的不同，对数据进行横向和纵向进行分组



- 横向分组，指用户可以接触到数据的范围，具体就是可以查看那个数据库的哪几张表
- 纵向分组，指用户对可接触到的数据能访问到什么程度，比如能select查找，能改，还是能删除



## 5、收回权限 revoke



收回权限就是取消已经赋予的用户的权限，收回用户不必要的权限保证系统的安全性



```mysql
revoke 权限1，权限2，权限3 on 数据库名.表 from user@host
```



> 注意： 在从user表中删除用户之前，应该收回对应用户的所有权限



收回 wangwu 的所有权限

![1658319196711](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658319196711.png)



登陆 wangwu 查看当前权限，所有权限被收回

![1658319224699](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658319224699.png)





## 6、权限表



MySQL 服务器通过权限表 来控制用户对数据的访问，权限表也放在mysql 数据库中。

mysql数据库系统会根据 这些权限表来赋予每个用户对应的权限

最重要的有user表、db表，还有 table_priv表、column_priv表 和 proc_priv表等

mysql 启动的时候，服务器会将这些表中的权限信息给读入到内存中



**mysql数据库中与权限相关的表**



![1658325862351](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658325862351.png)



**user表中与权限相关的字段内容，字段中包括与用户操作相关的一些权限设置，用枚举类型，y\n 表示**



![1658325977843](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658325977843.png)





**db 表中host、db、user作为联合主键，粒度增大，同时包含一些数据库权限相关的字段**



![1658326557022](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658326557022.png)



**tables_priv 表使用host、user、db、tables_name 作为主键，包含表的一些权限字段**



![1658326730989](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658326730989.png)





**column_priv 使用user、host、db、table_name、column_name作为联合主键，包含列相关的权限信息**



![1658326846398](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658326846398.png)





## 7、访问控制



&emsp;&emsp;在mysq中一个用户执行操作时，首先核实mysql服务器发送的来接请求，确认该请求是否被允许，这个过程称为访问控制过程，mysql的访问控制分为两个阶段，**连接核实与 请求核实阶段**



### 7.1、连接核实阶段



&emsp;&emsp;其实就是查看你与mysql数据库服务是否建立了连接，mysql服务器基于用户的身份以及提供的密码来验证身份来接受或者拒绝连接。



&emsp;&emsp;客户端在连接请求中提供 用户名、主机地址、密码，mysql服务器接收之后，与mysql.user表中的 user、host、authentication_string 匹配客户端提供的信息。



&emsp;&emsp;&emsp;只有在匹配成功时才能接收连接，等待用户请求。如果连接核实没有通过，服务器拒绝访问。





### 7.2 、请求核实阶段



当成功建立连接以后，服务器就会对mysql 客户端提供的每个请求，进行核实。怎么核实呢？



服务器检查要执行什么操作，是否有权限去执行。



&emsp;&emsp;mysql 先检查 user表，如果指定的权限在user表中没有授予，那么会继续检查 db表，db表的权限限定于数据库层次，如果在db表中没有被授予，那么mysql将继续检查 table_priv 表 以及 columns_priv 表，如果所有的权限表都检查完毕，还是没有找到允许的权限操作，那么返回错误信息，命令被拒绝。



![1658320872056](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658320872056.png)