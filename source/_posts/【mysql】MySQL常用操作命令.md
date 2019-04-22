---
title: 【mysql】MySQL常用操作命令
date: 2019-4-22 14:48:12
tags: 

- mysql
---

### 1. 权限解释

#### 全局管理权限：
FILE: 在MySQL服务器上读写文件。
PROCESS: 显示或杀死属于其它用户的服务线程。
RELOAD: 重载访问控制表，刷新日志等。
SHUTDOWN: 关闭MySQL服务。

#### 数据库/数据表/数据列权限：
ALTER: 修改已存在的数据表(例如增加/删除列)和索引。
CREATE: 建立新的数据库或数据表。
DELETE: 删除表的记录。
DROP: 删除数据表或数据库。
INDEX: 建立或删除索引。
INSERT: 增加表的记录。
SELECT: 显示/搜索表的记录。
UPDATE: 修改表中已存在的记录。

#### 特别的权限：
ALL: 允许做任何事(和root一样)。
USAGE: 只允许登录–其它什么也不允许做。

### 2. 备份、恢复（mysqldump）

