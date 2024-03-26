# 关于 MySQL 如何进行权限管理

## 管理用户

输入：

>`USE mysql;`
>
>`SELECT user FROM user;`

输出：

```shell
+------------------+
| user             |
+------------------+
| mysql.infoschema |
| mysql.session    |
| mysql.sys        |
| root             |
+------------------+
```

`mysql` 数据库有一个名为`user` 的表，它包含所有用户账号。`user` 表有一个名为`user` 的列，它存储用户登录名。新安装的服务器可能只有一个用户（如这里所示），过去建立的服务器可能具有很多用户。

### 创建用户账号

输入：

> `CREATE USER 'purexua'@'localhost' IDENTIFIED BY 'p@$$w0rd';`

`CREATE USER` 创建一个新用户账号。在创建用户账号时不一定需要口令，不过这个例子用`IDENTIFIED BY 'p@$$wOrd'` 给出了一个口令。

### 重命名用户

输入：

> `RENAME USER purexua TO purexu;`

**MySQL 5之前** 仅MySQL 5或之后的版本支持`RENAME USER` 。为了在以前的MySQL中重命名一个用户，可使用`UPDATE` 直接更新`user` 表。

### 删除用户账号

输入：

> `DROP USER purexua`

**自MySQL 5以来，`DROP USER` 删除用户账号和所有相关的账号权限。**在MySQL 5以前，`DROP USER` 只能用来删除用户账号，不能删除相关的权限。因此，如果使用旧版本的MySQL，需要先用`REVOKE` 删除与账号相关的权限，然后再用`DROP USER` 删除账号。

### 设置访问权限

为看到赋予用户账号的权限，使用`SHOW GRANTS FOR` ，如下所示：

输入：

> `SHOW GRANTS FOR 'purexua'@'localhost';`

输出：

```shell
mysql> SHOW GRANTS FOR purexua;
+-------------------------------------+
| Grants for purexua@%                |
+-------------------------------------+
| GRANT USAGE ON *.* TO `purexua`@`%` |
+-------------------------------------+
1 row in set (0.00 sec)
```

输出结果显示用户`purexua` 有一个权限`USAGE ON*.*` 。`USAGE` 表示*根本没有权限* （我知道，这不很直观），所以，此结果表示在*任意数据库* 和*任意表* 上对*任何东西没有权限* 。

输入：

> `GRANT SELECT ON crash_course.* TO purexua;`

再次查看用户权限

```shell
mysql> SHOW GRANTS FOR purexua;
+---------------------------------------------------+
| Grants for purexua@%                              |
+---------------------------------------------------+
| GRANT USAGE ON *.* TO `purexua`@`%`               |
| GRANT SELECT ON `crash_course`.* TO `purexua`@`%` |
+---------------------------------------------------+
2 rows in set (0.00 sec)
```

`GRANT` 的反操作为`REVOKE` ，用它来撤销特定的权限。下面举一个例子：

> REVOKE SELECT ON crash_course.* FROM purexua;

这条`REVOKE` 语句取消刚赋予用户`purexua` 的`SELECT` 访问权限。被撤销的访问权限必须存在，否则会出错。

GRANT 和REVOKE 可在几个层次上控制访问权限：

- 整个服务器，使用GRANT ALL 和REVOKE ALL ；

- 整个数据库，使用ON database.* ；

- 特定的表，使用ON database.table ；

- 特定的列；

- 特定的存储过程。

使用`GRANT` 和`REVOKE` ，再结合下表中列出的权限，你能对用户可以就你的宝贵数据做什么事情和不能做什么事情具有完全的控制。

|           权限            |                             说明                             |
| :-----------------------: | :----------------------------------------------------------: |
|           `ALL`           |                除`GRANT OPTION` 外的所有权限                 |
|          `ALTER`          |                      使用`ALTER TABLE`                       |
|      `ALTER ROUTINE`      |           使用`ALTER PROCEDURE` 和`DROP PROCEDURE`           |
|         `CREATE`          |                      使用`CREATE TABLE`                      |
|     `CREATE ROUTINE`      |                    使用`CREATE PROCEDURE`                    |
| `CREATE TEMPORARY TABLES` |                 使用`CREATE TEMPORARY TABLE`                 |
|       `CREATE USER`       | 使用`CREATE USER` 、`DROP USER` 、`RENAME USER` 和`REVOKE ALL PRIVILEGES` |
|       `CREATE VIEW`       |                      使用`CREATE VIEW`                       |
|         `DELETE`          |                         使用`DELETE`                         |
|          `DROP`           |                       使用`DROP TABLE`                       |
|         `EXECUTE`         |                    使用`CALL` 和存储过程                     |
|          `FILE`           |        使用`SELECT INTO OUTFILE` 和`LOAD DATA INFILE`        |
|      `GRANT OPTION`       |                    使用`GRANT` 和`REVOKE`                    |
|          `INDEX`          |              使用`CREATE INDEX` 和`DROP INDEX`               |
|         `INSERT`          |                         使用`INSERT`                         |
|       `LOCK TABLES`       |                      使用`LOCK TABLES`                       |
|         `PROCESS`         |                 使用`SHOW FULL PROCESSLIST`                  |
|         `RELOAD`          |                         使用`FLUSH`                          |
|   `REPLICATION CLIENT`    |                       服务器位置的访问                       |
|    `REPLICATION SLAVE`    |                        由复制从属使用                        |
|         `SELECT`          |                         使用`SELECT`                         |
|     `SHOW DATABASES`      |                     使用`SHOW DATABASES`                     |
|        `SHOW VIEW`        |                    使用`SHOW CREATE VIEW`                    |
|        `SHUTDOWN`         |         使用`mysqladmin shutdown` （用来关闭MySQL）          |
|          `SUPER`          | 使用`CHANGE MASTER` 、`KILL` 、`LOGS` 、`PURGE` 、`MASTER` 和`SET GLOBAL` 。还允许`mysqladmin` 调试登录 |
|         `UPDATE`          |                         使用`UPDATE`                         |
|          `USAGE`          |                          无访问权限                          |

在使用`GRANT` 和`REVOKE` 时，用户账号必须存在，但对所涉及的对象没有这个要求。这允许管理员在创建数据库和表之前设计和实现安全措施。

这样做的副作用是，当某个数据库或表被删除时（用`DROP` 语句），相关的访问权限仍然存在。而且，如果将来重新创建该数据库或表，**这些权限仍然起作用**。

可通过列出各权限并用逗号分隔，将多条`GRANT` 语句串在一起，如下所示：

> `GRANT SELECT, INSERT ON crash_course.* TO purexua;`

或者 使用所有权限

> `REVOKE | GRANT ALL  ON *.*  FROM | TO 'purexua'@'localhost';`

### 更改口令

验证当前密码的作用是当用户使用 `ALTER USER` **或** `SET PASSWORD` 语句更改密码时。示例使用 `ALTER USER` **，这比** `SET PASSWORD` 更受推荐，但这里描述的原则对于两个语句都是相同的。

输入：

> `ALTER USER USER() IDENTIFIED BY '*auth_string*' REPLACE '*current_auth_string*';`

or **更改指定用户的密码**

> `ALTER USER 'purexua'@'localhost'  IDENTIFIED BY '*auth_string*'  REPLACE '*current_auth_string*';`

or **更改指定用户的身份验证插件和密码**

> `ALTER USER 'purexua'@'localhost'  IDENTIFIED WITH caching_sha2_password BY '*auth_string*'  REPLACE '*current_auth_string*';`