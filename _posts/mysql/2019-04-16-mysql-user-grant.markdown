---
layout: post
title:  "mysql 创建用户及授权"
date:   2019-04-16
categories: mysql
tag: mysql
---

* content
{:toc}

# MySQL 创建用户 #

语法：

    CREATE USER <用户名> [ IDENTIFIED ] BY [ PASSWORD ] <口令>

实例：

	CREATE USER 'test'@'%' IDENTIFIED BY 'test';

> 语法说明：
> 
> 1. <用户名>： 指定创建用户账号，格式为 'user_name'@'host_name'。这里user_name是用户名，host_name为主机名，即用户连接 MySQL 时所在主机的名字。若在创建的过程中，只给出了账户的用户名，而没指定主机名，则主机名默认为“%”，表示一组主机。
> 
> 2. PASSWORD：可选项，用于指定散列口令，即若使用明文设置口令，则需忽略PASSWORD关键字；若不想以明文设置口令，且知道 PASSWORD() 函数返回给密码的散列值，则可以在口令设置语句中指定此散列值，但需要加上关键字PASSWORD。
> 
> 3. IDENTIFIED BY子句：用于指定用户账号对应的口令，若该用户账号无口令，则可省略此子句。
> 
> 4. <口令>：指定用户账号的口令，在IDENTIFIED BY关键字或PASSWOED关键字之后。给定的口令值可以是只由字母和数字组成的明文，也可以是通过 PASSWORD() 函数得到的散列值。
> 
> 使用 CREATE USER 语句应该注意以下几点：
> 
> - 如果使用 CREATE USER 语句时没有为用户指定口令，那么 MySQL 允许该用户可以不使用口令登录系统，然而从安全的角度而言，不推荐这种做法。
> - 使用 CREATE USER 语句必须拥有 MySQL 中 MySQL 数据库的 INSERT 权限或全局 CREATE USER 权限。
> - 使用 CREATE USER 语句创建一个用户账号后，会在系统自身的 MySQL 数据库的 user 表中添加一条新记录。若创建的账户已经存在，则语句执行时会出现错误。
> - 新创建的用户拥有的权限很少。他们可以登录 MySQL，只允许进行不需要权限的操作，如使用 SHOW 语句查询所有存储引擎和字符集的列表等。
> 
> 如果两个用户具有相同的用户名和不同的主机名，MySQL 会将他们视为不同的用户，并允许为这两个用户分配不同的权限集合。

# 修改用户 #

## 修改账号口令 ##

语法：

    RENAME USER <旧用户> TO <新用户>

实例：

    RENAME USER 'test'@'%' TO 'test1'@'localhost';

> 语法说明：
> 
> 1. <旧用户>：系统中已经存在的 MySQL 用户账号。
>  
> 2. <新用户>：新的 MySQL 用户账号。
> 
> 使用 RENAME USER 语句时应该注意以下几点：
> 
> - RENAME USER 语句用于对原有的 MySQL 账户进行重命名。若系统中旧账户不存在或者新账户已存在，则该语句执行时会出现错误。
> - 要使用 RENAME USER 语句，必须拥有 MySQL 中的 MySQL 数据库的 UPDATE 权限或全局 CREATE USER 权限。
> 

## 修改密码口令 ##

可以使用 SET PASSWORD 语句修改一个用户的登录口令。

语法格式：

    SET PASSWORD [ FOR <用户名> ] =
    {
    	PASSWORD('新明文口令')
    	| OLD_PASSWORD('旧明文口令')
    	| '加密口令值'
    }

实例：

    SET PASSWORD FOR 'test2'@'localhost'= PASSWORD('test');

> 语法说明如下:
> 
> - FOR 子句：可选项。指定欲修改口令的用户。
> - PASSWORD('新明文口令')：表示使用函数 PASSWORD() 设置新口令，即新口令必须传递到函数 PASSWORD() 中进行加密。
> - 加密口令值：表示已被函数 PASSWORD() 加密的口令值。
> 
> 使用 SET PASSWORD 语句应注意以下几点：
> 
> - 在 SET PASSWORD 语句中，若不加上 FOR 子句，表示修改当前用户的口令。若加上 FOR 子句，表示修改账户为 user 的用户口令。
> - user 必须以 'user_name'@'host_name' 的格式给定，user_name 为账户的用户名，host_name 为账户的主机名。
> - 该账户必须在系统中存在，否则语句执行时会出现错误。
> - 在 SET PASSWORD 语句中，只能使用选项 PASSWORD('新明文口令') 和加密口令值中的一项，且必须使用其中的一项。


# MySQL用户授权（GRANT） #

对于新建的 MySQL 用户，必须给它授权，可以用 GRANT 语句来实现对新建用户的授权。

语法格式：

    GRANT
    <权限类型> [ ( <列名> ) ] [ , <权限类型> [ ( <列名> ) ] ]
    ON <对象> <权限级别> TO <用户>
    其中<用户>的格式：
    <用户名> [ IDENTIFIED ] BY [ PASSWORD ] <口令>
    [ WITH GRANT OPTION]
    | MAX_QUERIES_PER_HOUR <次数>
    | MAX_UPDATES_PER_HOUR <次数>
    | MAX_CONNECTIONS_PER_HOUR <次数>
    | MAX_USER_CONNECTIONS <次数>

实例：

	GRANT SELECT,INSERT ON *.* TO 'test2'@'localhost';

	GRANT all privileges ON *.* TO 'test2'@'localhost';
    
    GRANT all privileges ON test.test TO 'test2'@'localhost';
    
   	flush privileges;
    

> 语法说明如下：
> 
> 1. <列名> 可选项:用于指定权限要授予给表中哪些具体的列。
> 
> 2. ON 子句:用于指定权限授予的对象和级别，如在 ON 关键字后面给出要授予权限的数据库名或表名等。
> 
> 3. <权限级别>:用于指定权限的级别。可以授予的权限有如下几组：
>  - 列权限，和表中的一个具体列相关。例如，可以使用 UPDATE 语句更新表 students 中 student_name 列的值的权限。
>  - 表权限，和一个具体表中的所有数据相关。例如，可以使用 SELECT 语句查询表 students 的所有数据的权限。
>  - 数据库权限，和一个具体的数据库中的所有表相关。例如，可以在已有的数据库 mytest 中创建新表的权限。
>  - 用户权限，和 MySQL 中所有的数据库相关。例如，可以删除已有的数据库或者创建一个新的数据库的权限。

> 对应地，在 GRANT 语句中可用于指定权限级别的值有以下几类格式：
> 
> - \*：表示当前数据库中的所有表。
> - \*.\*：表示所有数据库中的所有表。
> - db_name.\*：表示某个数据库中的所有表，db_name 指定数据库名。
> - db_name.tbl_name：表示某个数据库中的某个表或视图，db_name 指定数据库名，tbl_name 指定表名或视图名。
> - tbl_name：表示某个表或视图，tbl_name 指定表名或视图名。
> - db_name.routine_name：表示某个数据库中的某个存储过程或函数，routine_name 指定存储过程名或函数名。
> - TO 子句：用来设定用户口令，以及指定被赋予权限的用户 user。若在 TO 子句中给系统中存在的用户指定口令，则新密码会将原密码覆盖；如果权限被授予给一个不存在的用户，MySQL 会自动执行一条 CREATE USER 语句来创建这个用户，但同时必须为该用户指定口令。
> 
> GRANT语句中的\<权限类型\>的使用说明如下：
> 
> **1. 授予数据库权限时，\<权限类型\>可以指定为以下值：**
> 
> - SELECT：表示授予用户可以使用 SELECT 语句访问特定数据库中所有表和视图的权限。
> - INSERT：表示授予用户可以使用 INSERT 语句向特定数据库中所有表添加数据行的权限。
> - DELETE：表示授予用户可以使用 DELETE 语句删除特定数据库中所有表的数据行的权限。
> - UPDATE：表示授予用户可以使用 UPDATE 语句更新特定数据库中所有数据表的值的权限。
> - REFERENCES：表示授予用户可以创建指向特定的数据库中的表外键的权限。
> - CREATE：表示授权用户可以使用 CREATE TABLE 语句在特定数据库中创建新表的权限。
> - ALTER：表示授予用户可以使用 ALTER TABLE 语句修改特定数据库中所有数据表的权限。
> - SHOW VIEW：表示授予用户可以查看特定数据库中已有视图的视图定义的权限。
> - CREATE ROUTINE：表示授予用户可以为特定的数据库创建存储过程和存储函数的权限。
> - ALTER ROUTINE：表示授予用户可以更新和删除数据库中已有的存储过程和存储函数的权限。
> - INDEX：表示授予用户可以在特定数据库中的所有数据表上定义和删除索引的权限。
> - DROP：表示授予用户可以删除特定数据库中所有表和视图的权限。
> - CREATE TEMPORARY TABLES：表示授予用户可以在特定数据库中创建临时表的权限。
> - CREATE VIEW：表示授予用户可以在特定数据库中创建新的视图的权限。
> - EXECUTE ROUTINE：表示授予用户可以调用特定数据库的存储过程和存储函数的权限。
> - LOCK TABLES：表示授予用户可以锁定特定数据库的已有数据表的权限。
> - ALL 或 ALL PRIVILEGES：表示以上所有权限。
> 
> **2. 授予表权限时，\<权限类型\>可以指定为以下值：**
> 
> - SELECT：授予用户可以使用 SELECT 语句进行访问特定表的权限。
> - INSERT：授予用户可以使用 INSERT 语句向一个特定表中添加数据行的权限。
> - DELETE：授予用户可以使用 DELETE 语句从一个特定表中删除数据行的权限。
> - DROP：授予用户可以删除数据表的权限。
> - UPDATE：授予用户可以使用 UPDATE 语句更新特定数据表的权限。
> - ALTER：授予用户可以使用 ALTER TABLE 语句修改数据表的权限。
> - REFERENCES：授予用户可以创建一个外键来参照特定数据表的权限。
> - CREATE：授予用户可以使用特定的名字创建一个数据表的权限。
> - INDEX：授予用户可以在表上定义索引的权限。
> - ALL 或 ALL PRIVILEGES：所有的权限名。
> 
> **3. 授予列权限时，\<权限类型\>的值只能指定为 SELECT、INSERT 和 UPDATE，同时权限的后面需要加上列名列表 column-list。**
> 
> **4. 最有效率的权限是用户权限。**
> 
> 授予用户权限时，<权限类型>除了可以指定为授予数据库权限时的所有值之外，还可以是下面这些值：
> 
> - CREATE USER：表示授予用户可以创建和删除新用户的权限。
> - SHOW DATABASES：表示授予用户可以使用 SHOW DATABASES 语句查看所有已有的数据库的定义的权限。



[http://c.biancheng.net/view/2615.html](http://c.biancheng.net/view/2615.html)

