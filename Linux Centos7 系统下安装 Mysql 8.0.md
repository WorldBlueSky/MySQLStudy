# Linux Centos 系统下安装Mysql



## (1)官网下载rpm离线安装包



Mysql 官网   mysql.com ,点击 downloads

![1658214220657](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658214220657.png)





滑到最下面，点击下载 mysql 社区版

![1658214257001](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658214257001.png)



点击 Mysql Community Server

![](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658214303330.png)

选择正确的系统版本，下载 rpm 离线安装包



![1658214347677](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658214347677.png)





下载完之后解压得到这些

![1658214407210](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658214407210.png)



我们用不着每个都推送到Linux上安装，只需要 client、client-plugins 、common、libs、server、icu-data即可



## （2）上传至linux系统



利用 xftp 将压缩包上传到 linux上



![1658214716153](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658214716153.png)



## （3）检查权限、安装依赖



给/tmp 目录较大的权限

```bash
chmod -R 777 /tmp
```



如果不存在依赖，那么要安装依赖

```bash
yum install libaio
```

```bash
yum install net-tools
```



## （4）按照顺序进行rpm



![1658216014209](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658216014209.png)



补充在 server 安装之前 rpm icu0data



```bash
rpm -ivh mysql-community-common-8.0.29-1.el7.x86_64.rpm
```



![1658216346659](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658216346659.png)





```bash
rpm -ivh mysql-community-client-plugins-8.0.29-1.el7.x86_64.rpm
```



![1658216414761](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658216414761.png)





```bash
rpm -ivh mysql-community-libs-8.0.29-1.el7.x86_64.rpm
```



这一步很有可能会报错，缺少依赖



![1658216467674](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658216467674.png)



在进行libs的时候很可能提示，缺少依赖,输入以下命令之后再次安装 libs



```bash
yum remove mysql-libs
```



![1658216698287](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658216698287.png)



```bash
rpm -ivh mysql-community-client-8.0.29-1.el7.x86_64.rpm
```



![1658216610138](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658216610138.png)





```bash
rpm -ivh mysql-community-icu-data-files-8.0.29-1.el7.x86_64.rpm
```



![1658216876555](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658216876555.png)



```bash
rpm -ivh mysql-community-server-8.0.29-1.el7.x86_64.rpm 
```



![1658216640826](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658216640826.png)



## （5）如果成功安装了，那么查看当前mysql版本



```bahs
mysql --version
```

![1658215973120](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658215973120.png)



如果显示版本的话，那么说明安装成功





输入以下命令，查看rpm 安装的文件是否成功



```bash
rpm -qa|grep -i mysql
```



![1658217029923](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658217029923.png)



## （6）服务的初始化



```bash
mysqld --initialize --user=mysql
```



--initialize 默认以安全模式来初始化，给root用户生成一个过期的临时密码

登陆后需要重新设置新密码，临时密码需要在日志中查看



## （7）查看临时密码

```bash
cat /var/log/mysqld.log
```



![1658217988264](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658217988264.png)





## （8）开启Mysql 服务



查看mysql服务此时的状态



```bash
systemctl status mysqld
```



![1658218084579](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658218084579.png)





开启mysql服务，之后再次查看状态



```bash
systemctl start mysqld
```



![1658218191059](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658218191059.png)



## （9）设置MySQL服务自启动



查看MySQL服务的启动状态,已经自启动了



```bash
systemctl list-unit-files|grep mysqld.service
```



![1658218370657](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658218370657.png)



如果状态为 disabled,那么需要设置为 enabled



```bash
systemctl enable mysqld.service
```



如果不希望服务自启动的话，那么设置为disabled



```bash
systemctl disable mysqld.service
```



## (10) 登陆mysql，设置新密码



打开mysql



```bash
mysql -uroot -p
```



输入之前查看日志的临时密码即可



![1658218596321](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658218596321.png)



输入之后成功进入到mysql中



![1658218661915](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658218661915.png)



我们第一件事使用以下mysql查看数据库

```mysql
show databases;
```



**提示：你必须重新设置你的密码通过alter user s语句 在执行任何sql命令之前**



![1658218724632](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658218724632.png)



通过下面这个语句设置新密码



```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456'
```



此时再去使用sql命令，完全可以使用了



![1658218941787](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658218941787.png)



同时在以后的登陆中只需要输入自己设置的密码即可登陆



![1658219009315](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1658219009315.png)