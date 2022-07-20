# MySQL 高级基础(三)----角色管理



角色是权限的集合，相当于把权限都集中于角色中，或者换个说法

原来是 权限 -->  用户

现在是  权限 --> 角色 --> 用户



![1658329242242](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658329242242.png)



&emsp;&emsp;为了系统的安全性，需要给用户授予权限，但是当用户数量较多时，为了避免给每一个用户创建授予很多权限，我们先把权限放到角色集合中，用户要用的话直接赋予这个角色即可。



# 1、角色与权限



这里会讲到 角色与权限直接的关系设置 ，同时 角色与用户之间的设置，先将与权限的关系。



## 1.1 创建角色



角色其实和用户差不多，都放到 mysql.user表中，只不过角色与用户相比，只具有权限相关的功能



```mysql
creat role 'role_name'@'host_name' [,role@host]... #如果不加主机号，默认为 % 
```



使用上面这个命令创建一个或者多个角色，如果不写主机号默认为 %



创建角色 boss、manage



![1658331348371](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658331348371.png)



## 1.2 查看角色



角色和 用户是在一张表中存的，所以不用担心角色和用户相同的情况



```mysql
select user,host from mysql.user;
```



查看刚才创建的角色信息

![1658331386652](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658331386652.png)



## 1.3 删除角色



```mysql
drop role 'role_name'@'host' [,'role_name'@'host']...
```



删除一个或者多个角色信息



## 1.4 给角色赋予权限



赋予权限、回收权限其实和 给用户权限差不多，或者一样的



赋予权限的命令

```mysql
grant 权限1,权限2,权限3 on 数据库.表明 to 'role_name'@'host';
```



给boss 赋予所有的权限



![1658332404584](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658332404584.png)

给 manage 赋予对 data_DB 所有表增删改查的权限



![1658332420970](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658332420970.png)



## 1.5 查看角色的权限



```mysql
select grants for 'role_name'@'host';
```



查看manage 角色的权限

![1658332514765](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658332514765.png)





## 1.6 回收角色的权限



```mysql
revoke 权限1，权限2，权限3 on 数据库.表 from 'role_name'@'host';
```



回收manage 角色的 update、delete 权限



![1658332653692](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658332653692.png)



再次查看manage 角色的权限



![1658332708241](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658332708241.png)



# 2、角色与用户



然而，给角色赋予了一定的权限之后，用户怎么与角色建立连接呢？



## 2.1 给用户赋予角色



```mysql
grant 角色及主机 to 用户及主机 
```



使用root用户 给 wangwu 赋予角色 manage

![1658333055410](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658333055410.png)



登陆wangwu，查看当前授权信息，发现 manage 角色已经授权给  wangwu 用户了

![](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658333128786.png)

但是会发现一个问题，我们用 wangwu 用户进行查询数据库

![1658333194105](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658333194105.png)

没有查到 data_DB 数据库，说明这个角色的权限还没有生效，那么就需要激活



## 2.2 激活角色



mysql 中创建的角色默认都是未激活的，我们必须激活才能拥有 角色的权限。



### 方式一 set default role...to..



```mysql
 set default role 'role_name' to '用户及主机',[用户及主机] [用户及主机] 
```



激活 manage 角色给 wangwu@localhost 使用，当前用户必须退出重新登陆才有效



![1658333407600](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658333407600.png)



登陆 wangwu 发现已经拥有了 manage 角色的权限了

![1658333566258](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658333566258.png)



### 方式二  修改系统变量



默认情况下

```mysql
show variables like  'active_all_roles_on_login';
```

![1658333754687](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658333754687.png)



需要我们设置成 ON

```mysql
set global active_all_roles_on_login=ON;
```



这条语句的作用是 对所有创建的角色都永久激活，运行这条语句，用户才真正用户有了角色的全部权限



## 2.3 查看当前会话激活的角色



查询当前登陆用户下的激活的角色

```mysql
select current_role();
```



如果角色没有被激活的话，那么显示为null



![1658333921889](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658333921889.png)



wangwu 用户 把 manage 角色 给激活之后，可查询到激活的角色



![1658334001593](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658334001593.png)



## 2.3 撤销用户的角色



```mysql
revoke 角色名及主机 from 用户名及主机
```



root 用户撤销 wangwu用户的角色

![1658334142946](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658334142946.png)

wangwu 用户重新登陆，此时角色又为 none 了

![1658334216282](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658334216282.png)



## 2.4 设置强制的角色



强制角色就是给每个创建的用户都默认授予激活的角色, 这个角色不能删除和撤销



方式一：服务启动前设置

```mysql
[mysqld]
mandatory_roles=角色名及密码
```



方式二：运行时设置

```mysql
set presist mandatory_roles= 角色及主机名 #重启后依然有效，相当于改配置文件中的内容
```







