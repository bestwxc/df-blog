# mysql 用户、数据库、权限常用操作
## 系统信息
- 操作系统： macOS High Sierra 10.13.6
- mysql版本： MySQL Community Server 5.7.18

## 登录
- 使用root 用户登录mysql
    + mysql -uroot -p123456

## 用户
- 查看用户
    + 通过查看系统表中的user表查看
    + select t.host,t.user,t.account_locked from mysql.user t;
- 创建用户
    + create user 'dream_flower'@'%' identified by 'dream_flower';
- 修改用户名
    + update mysql.user set authentication_string = password('dream_flower'), password_expired = 'N', password_last_changed = now() where user = 'dream_flower';
- 删除用户
    + drop user 'dream_flower'@'%';

## 数据库
- 查看数据库
    + show databases;
- 切换当前数据库
    + use mysql;
- 创建数据库
    + create database dream_flower DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
    + COLLATE表示校对方式
        * utf8_general_ci 不区分大小写，字符方式保存
        * utf8_general_cs 区分大小写，字符方式保存(该版本不能使用)
        * utf8_bin 区分大小，将字符串每个字符使用二进制保存，且同时可以保存二进制内容
- 删除数据库
    + drop database dream_flower;

## 权限
### 权限级别
- 全局层级权限
- 数据库层级权限
- 表层级别权限
- 列层级别权限
- 子程序层级权限
### 权限类型
mysql提供的权限类型见[官方文档](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html)

### 授予操作
- 查看权限
    + 权限保存在user, db, tables_priv, columns_priv, and procs_priv表中，可以通过查看对应表查看
    + 查看权限 show grants for dream_flower;
    + select * from mysql.user where user='dream_flower'\G;
    + select * from mysql.db where use='dream_flower';
    + select * from mysql.tables_priv where use='dream_flower';
    + select * from mysql.columns_priv where use='dream_flower';
    + select * from mysql.procs_priv where use='dream_flower';
- 授予权限
    + 格式 
        * grant 权限 on 数据库.表 to '用户'@'授予访问IP' identified by 密码;
- 回收权限
    + 格式
        * revoke 权限 on 数据库.表 from '用户'@'授予访问IP';

