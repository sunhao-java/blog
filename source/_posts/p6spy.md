title: 关于p6spy的介绍
tags: [Java, p6spy]
date: 2013-07-11 16:44
---

最近公司项目里需要使用到将所有执行的sql打出日志，放在一个文件下。公司一大神级人物给解决了，刚刚花了点时间给研究了一下。

他用的就是p6spy，其实也没做多少工作。主要的工作p6spy都已经做过了。

p6spy我的理解就是：p6spy将应用的数据源给劫持了，应用操作数据库其实在调用p6spy的数据源，p6spy劫持到需要执行的sql或者hql之类的语句之后，他自己去调用一个realDatasource，再去操作数据库，只要劫持到那些sql之后，能干的事情就很多了。

p6spy 可以输出日志到文件中、控制台、或者传递给 Log4j，而且还能配搭 SQL Profiler 或 IronTrackSQL 图形化监控 SQL 语句，监测到哪些语句的执行是耗时的，逐个优化。关于与 SQL Profiler 或 IronTrackSQL 的配合使用可参数文件的链接。 

p6spy在sourceforge上下载：[p6spy][]
[p6spy]: http://sourceforge.net/projects/p6spy/?source=dlp


p6spy的配置：

1. p6spy.jar放入应用的classpath下
2. 修改连接池或者连接配置的jdbc的驱动为p6spy所提供的驱动，`com.p6spy.engine.spy.P6SpyDriver`

	在单独的Hibernate的应用中，数据库驱动配置在hibernate.cfg.xml里面,所以我需要配置文件中的connection.driver_class属性从oracle.jdbc.driver.OracleDriver改为com.p6spy.engine.spy.P6SpyDriver其他的用户名密码等等配置信息全部不用修改.在web程序中，配置的连接池部分，也只需要修改jdbc-driver的配置即可。
	
	Hibernate.cfg.xml:
	
		<session-factory>
		    <!-- 在这里改成p6spy提供的数据源 -->
		    <property name="connection.driver_class">com.p6spy.engine.spy.P6SpyDriver</property>       
		    <property name="connection.url">jdbc:oracle:thin:@localhost:1521:orcl</property>
		    <property name="connection.username">scott</property>
		    <property name="connection.password">tiger</property>
		    <property name="connection.pool_size">1</property>
		    <property name="dialect">org.hibernate.dialect.Oracle9Dialect</property>
		    <property name="current_session_context_class">thread</property>
		    <property name="cache.provider_class">org.hibernate.cache.NoCacheProvider</property>
		    <property name="show_sql">true</property>
		    <property name="hbm2ddl.auto">false</property>
		    <property name="hibernate.jdbc.batch_size">0</property>
		</session-factory>
3. 修改 `spy.properties` 并将其放到classpath下：

		#################################################################
		# MODULES                                                       #
		#                                                               #
		# Modules provide the P6Spy functionality.  If a module, such   #
		# as module_log is commented out, that functionality will not   #
		# be available.  If it is not commented out (if it is active),  #
		# the functionality will be active.                             #
		#                                                               #
		# Values set in Modules cannot be reloaded using the            #
		# reloadproperties variable.  Once they are loaded, they remain #
		# in memory until the application is restarted.                 #
		#                                                               #
		#################################################################
		#第一：module.log的属性必须配置，如果不配置，P6SPY将不起任何作用，典型配置：
		module.log=com.p6spy.engine.logging.P6LogFactory
		#module.outage=com.p6spy.engine.outage.P6OutageFactory
		 
		#################################################################
		# REALDRIVER(s)                                                 #
		#                                                               #
		# In your application server configuration file you replace the #
		# "real driver" name with com.p6spy.engine.P6SpyDriver. This is #
		# where you put the name of your real driver P6Spy can find and #
		# register your real driver to do the database work.            #
		#                                                               #
		# If your application uses several drivers specify them in      #
		# realdriver2, realdriver3.  See the documentation for more     #
		# details.                                                      #
		#                                                               #
		# Values set in REALDRIVER(s) cannot be reloaded using the      #
		# reloadproperties variable.  Once they are loaded, they remain #
		# in memory until the application is restarted.                 #
		#                                                               #
		#################################################################
		 
		#第二：数据库驱动配置，你懂的，不多说了
		# oracle driver
		# realdriver=oracle.jdbc.driver.OracleDriver
		 
		# mysql Connector/J driver
		# realdriver=com.mysql.jdbc.Driver
		 
		# informix driver
		# realdriver=com.informix.jdbc.IfxDriver
		 
		# ibm db2 driver
		# realdriver=COM.ibm.db2.jdbc.net.DB2Driver
		 
		# the mysql open source driver
		realdriver=org.gjt.mm.mysql.Driver
		 
		#specifies another driver to use
		realdriver2=
		#specifies a third driver to use
		realdriver3=
		 
		#第三：appender配置，一般分为三种
		#specifies the appender to use for logging
		#appender=com.p6spy.engine.logging.appender.Log4jLogger
		#控制台
		#appender=com.p6spy.engine.logging.appender.StdoutLogger
		appender=com.p6spy.engine.logging.appender.FileLogger
		 
		# name of logfile to use, note Windows users should make sure to use forward slashes in their pathname (e:/test/spy.log) (used for file logger only)
		#日志文件存放路径及文件名
		logfile     = spy.log
		 
		# append to  the p6spy log file.  if this is set to false the
		# log file is truncated every time.  (file logger only)
		append=true
		 
		#The following are for log4j logging only
		log4j.appender.STDOUT=org.apache.log4j.ConsoleAppender
		log4j.appender.STDOUT.layout=org.apache.log4j.PatternLayout
		log4j.appender.STDOUT.layout.ConversionPattern=p6spy - %m%n
		 
		log4j.logger.p6spy=INFO,STDOUT