---
title: Oracle连接：ORA-28001 the password has expired
date: 2019-01-31 14:05:30
categories: 
- 数据库
tags: Oracle异常
---

Oracle连接出现的异常：

​       ![](Oracle连接：ORA-28001-the-password-has-expired\111.png)    

原因是密码过期。

此时我们应该登陆oracle.以DBA的身份登陆。打开cmd。然后输入：sqlplus / as sysdba；之后查看密码时间：

select * from dba_profiles where profile='DEFAULT' and resource_name='PASSWORD_LIFE_TIME';

可以看到是180天，此时我们设置为永久：alter profile default limit password_life_time unlimited;​           

然后需要修改要登陆数据库的用户名： select username from dba_users;查看所有的数据库连接；

修改密码：alter user 用户名 identified by 密码;

之后登陆正常。