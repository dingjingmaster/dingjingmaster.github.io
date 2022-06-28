# Sql语句整理


## 数据库

- 实现数据持久化
- 使用完整的管理系统统一管理，易于查询

> 补充一句: 使用数据库并不比把数据保存到文件性能好，数据库的优点在于其结构化的数据存储和管理查询引擎，解决数据增、删、改、查中的同步问题以及为表的权限管理等问题。

数据库概念：
1. DB(数据库)：存储数据的仓库，保存一系列有组织的数据
2. DBMS(数据库管理系统)：数据库通过 DBMS 创建和操作的容器
3. SQL(结构化查询语言)：专门用来与数据库通信的语言

常见数据库：MySQL、Oracle、DB2、SqlServer 等

## SQL
SQL (Structural query language) —— 结构查询语言，其具有以下优点：
1. 不是某个特定数据库供应商专用语言，几乎所有 DBMS 都支持 SQL
2. 简单易学、语言灵活

SQL 语句可以分为以下三种类型：
1. DML(Data Manipulation Language) 数据操纵语言
2. DDL (Data Definition Language) 数据定义语言
3. dcl (Data Control Language) 数据控制语言

### DML
用于查询与修改数据记录，包括如下 SQL 语句：
- INSERT: 添加数据到数据库中
- UPDATE: 修改数据库中的数据
- DELETE: 删除数据库中的数据
- SELECT: 选择数据库中的数据

### DDL
DDL 用于定义数据库的结构，比如创建、修改、删除数据库对象，包括以下SQL语句
- CREATE TABLE: 创建数据库表
- ALTER TABLE: 更改表结构、添加、删除、修改列长度
- DROP TABLE: 删除表
- CREATE INDEX: 在表上建立索引
- DROP INDEX: 删除索引

### DCL
DCL 用来控制数据库的访问，包括如下 SQL 语句：
- GRANT: 授予访问权限
- REVOKE: 撤销访问权限
- COMMIT: 提交事务处理
- ROLLBACK: 事务处理回退
- SAVEPOINT: 设置保存点
- LOCK: 对数据库的特定部分进行锁定

