title: Oracle_降级导入导出
tags: [Oracle, database, 降级, 导入, 导出]
date: 2016-03-18
categories: 数据库
---

> 使用oracle的expdp和impdp

<!-- more -->

1. 在服务器上`expdb`一下,服务器是`Linux`,切换到`oracle`用户,进行下面的操作:

		expdp userid=usr_oa_new/usr_oa_new@192.168.102.202:1521/urpdb schemas=usr_oa_new directory=test dumpfile=usr_oa_new_20130826.dmp logfile=log_20130826.log version=10.02.01

	注意:
	- `userid=usr_oa_new/usr_oa_new@192.168.102.202:1521/urpdb` 不用说了,源数据库的信息
	- `schemas=usr_oa_new	` 该方案用于指定执行方案模式导出,默认为当前用户方案.
	- `directory=test` 这里需要用有dba权限的oracle用户登录数据库新建一个目录地址,这里非真正的硬盘物理路径,仅仅相当于一个在oracle中的代号
	- `create directory test as '/home/oracle';`
	- `grant read,write on directory test to usr_oa_new;`
	- `dumpfile=usr_oa_new_20130826.dmp以及logfile=log_20130826.log` 你懂滴
	- `version=10.02.01` 这个很重要,这里指定你希望以哪个版本的oracle的dmp导出,即目标数据库的版本号

	至此,导出部分已经OK了.

2. 在目标数据库所在服务器上使用impdp,通常情况为windows系统

		impdp userid=usr_oa_cumt_new/usr_oa_cumt_new@sunhao remap_schema=usr_oa_new:usr_oa_cumt_new directory=test_2 dumpfile=usr_oa_new_20130826.dmp logfile=impdp.log table_exists_action=replace version=10.02.01

	注意:
	- `userid=usr_oa_cumt_new/usr_oa_cumt_new@sunhao` the same as expdp
	- `remap_schema=usr_oa_new:usr_oa_cumt_new` 如果目标用户与源用户一致,则可以不写,否则,格式为`remap_schema=源:目标`
	- `directory,dumpfile,logfile` the same as expdp
	- `table_exists_action=replace` 表存在的时候怎么办
	- `version=10.02.01` 要导入的数据库版本

如果报错:

	remap_schema需要权限,就用dba权限的用户设置权限去:
	grant imp_full_database to usr_oa_cumt_new;