title: Oracle_恢复数据
tags: [Oracle, database, 恢复数据]
date: 2016-03-18
---

## 利用闪回恢复数据

	alter table message.t_message_msg enable row movement;
	flashback table message.t_message_msg to timestamp to_timestamp('2013-05-19 08:30:00','yyyy-mm-dd hh24:mi:ss');

`message.t_message_msg` 指的是需要恢复数据的表

<!-- more -->

## 使用undo恢复数据

	create table t_flow_workitem_bak_20131011_2 as select * from t_flow_workitem as of timestamp (sysdate - 100/1440);
	create table t_flow_histworkitem_bak_2 as select * from t_flow_histworkitem as of timestamp (sysdate - 70/1440);
	create table t_flow_startprocess_bak_4 as select * from t_flow_startprocess as of timestamp (sysdate - 200/1440);
	create table urgedhandle_feedback_bak_2 as select * from t_flow_urgedhandle_feedback as of timestamp (sysdate - 70/1440);
	create table urgedhandle_bak_20131011 as select * from t_flow_urgedhandle as of timestamp (sysdate - 70/1440);
	create table t_oa_doc_relay_20131011 as select * from t_oa_doc_relay as of timestamp (sysdate - 70/1440);
	create table t_oa_doc_receiver_20131011 as select * from t_oa_doc_receiver as of timestamp (sysdate - 70/1440);
