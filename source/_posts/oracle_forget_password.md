title: Oracle_忘记密码解决方案
tags: [Oracle, database, 忘记密码]
date: 2016-03-18
---

> 如果忘记oracle的sys、system、scott等用户名的密码：

<!-- more -->

1. 配置环境变量

		ORACLE_HOME=F:\study\oracle\product\10.1.0\db_1
		path=(原path);ORACLE_HOME/BIN
2. 打开命令提示符窗口，输入`sqlplus /nolog`

		conn /as sysdba
		alter user XXX identified by "password";
   然后就OK了。


---

命令行内容：

	Microsoft Windows XP [版本 5.1.2600](C) 版权所有 1985-2001 Microsoft Corp.
	C:\Documents and Settings\Administrator>sqlplus /nolog
	SQL*Plus: Release 10.1.0.2.0 - Production on 星期四 11月 3 18:48:48 2011
	Copyright (c) 1982, 2004, Oracle.  All rights reserved.
	SQL> conn /as sysdba
	已连接。
	SQL> alter user scott identified by "123456";
	用户已更改。
	SQL>