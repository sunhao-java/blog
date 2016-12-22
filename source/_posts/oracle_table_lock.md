title: Oracle_查看锁表信息及解锁
tags: [Oracle,database, 锁表]
date: 2016-03-18
---

<!-- more -->

1. 查看被锁的表的信息

		select s.username,
		   decode(l.type, 'tm', 'table lock', 'tx', 'row lock', null) lock_level,
		   o.owner,
		   o.object_name,
		   o.object_type,
		   s.sid,
		   s.serial#,
		   s.terminal,
		   s.machine,
		   s.program,
		   s.osuser
		from v$session s, v$lock l, dba_objects o
		where l.sid = s.sid
		and l.id1 = o.object_id(+)
		and s.username is not null;

2. 表解锁

		--kill session语句
		alter system kill session '50,492'; ---(SID,SERIAL#)