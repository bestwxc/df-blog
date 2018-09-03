# 使用pfile启动Oracle，解决如ora-27102等启动报错问题

## 系统信息
- 操作系统：Red Hat Enterprise Linux Server release 6.6 (Santiago)
- ORACLE版本：11.2.0.4.0

## 错误出现始末
测试环境oracle所在虚拟机使用生产环境的虚拟机模板，分配了所在物理机的所有内存（64G），调小虚拟机分配的内存后，重启报错，报错信息如下。
```
ORA-27102: out of memory
Linux-x86_64 Error: 28: No space left on device
```

## 原因分析和思路
根据提示和错误出现操作前的操作，可知oracle中配置分配了系统能够分配的最大内存。Oracle的内存设置项有SGA和PGA，将对应配置调小即可。由于ORACLE已经无法启动，所以使用修改pfile并利用pfile启动的方式。

## 操作步骤
- 登陆到oracle所在的用户
    + su - oracle
- 使用本地用户登陆sqlplus
    + sqlplus / as sysdba（sqlplus /nolog;conn / as sysdba）
- 创建pfile
    + create pfile from spfile;
- 打开pfile，pfile一般再$ORACLE_HOME/dbs
    + vim /u01/oracle/product/11.2.0/db/dbs/inittestdb.ora,文件如下
```
testdb.__java_pool_size=134217728
testdb.__large_pool_size=67108864
testdb.__oracle_base='/u01/oracle'#ORACLE_BASE set from environment
testdb.__pga_aggregate_target=2147483648
testdb.__sga_target=2147483648
testdb.__shared_io_pool_size=0
testdb.__shared_pool_size=2818572288
testdb.__streams_pool_size=67108864
*.audit_file_dest='/u01/oracle/admin/testdb/adump'
*.audit_trail='db'
*.compatible='11.2.0.4.0'
*.control_files='/u02/oradata/testdb/testdb/control01.ctl','/u02/oradata/testdb/testdb/control02.ctl'
*.db_block_size=8192
*.db_domain=''
*.db_name='testdb'
*.diagnostic_dest='/u01/oracle'
*.nls_language='AMERICAN'
*.nls_territory='AMERICA'
*.open_cursors=300
*.pga_aggregate_target=2147483648
*.processes=3000
*.remote_login_passwordfile='EXCLUSIVE'
*.sessions=3305
*.sga_target=2147483648
```

- 修改*.sga_target,*.pga_aggregate_target,testdb.__pga_aggregate_target,testdb.__sga_target的值为2147483648

- 编辑后保存
- 重新连接数据库
    + sqlplus / as sysdba
- 使用pfile启动
    + startup pfile='/u01/oracle/product/11.2.0/db/dbs/inittestdb.ora'
- 将pfile中的改动写入spfile
    + create spfile from pfile;
    
