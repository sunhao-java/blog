title: Sonar使用方法
tags: [Sonar]
date: 2014-10-17
categories: 中间件
---

###方法1：直接在服务器分析
- 在项目源代码目录新建sonar-project.properties文件`touch sonar-project.properties`
- 编辑此文件`vim sonar-project.properties`，内容如下：

		# Required metadata
		sonar.projectKey=my:project		#项目的key
		sonar.projectName=My project	#项目显示的名称
		sonar.projectVersion=1.0		#版本

<!-- more -->

		# Path to the parent source code directory.
		# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
		# Since SonarQube 4.2, this property is optional if sonar.modules is set.
		# If not set, SonarQube starts looking for source code from the directory containing
		# the sonar-project.properties file.
		sonar.sources=src				#项目源代码目录

		# Encoding of the source code
		sonar.sourceEncoding=UTF-8		#使用的编码格式

		# Additional parameters
		sonar.my.property=value
- 执行债务分析
	- 在上面文件所在路径执行`sonar-runner`

###方法2：结合maven使用
- 配置maven的setting.xml文件，位置：`%MAVEN_HOME%/conf/settings.xml`

		<profile>
		   <id>sonar</id>
		   <activation>
			  <activeByDefault>true</activeByDefault>
		   </activation>
		   <properties>
			  <!-- 数据库url -->
			  <sonar.jdbc.url>jdbc:mysql://172.16.128.160:3306/sonar</sonar.jdbc.url>
			  <!-- 数据库driver -->
			  <sonar.jdbc.driver>com.mysql.jdbc.Driver</sonar.jdbc.driver>
			  <!-- 数据库username -->
			  <sonar.jdbc.username>sonar</sonar.jdbc.username>
			  <!-- 数据库password -->
			  <sonar.jdbc.password>sonar</sonar.jdbc.password>
			  <!-- sonar地址 -->
			  <sonar.host.url>http://172.16.128.160:19000</sonar.host.url>
		   </properties>
		</profile>
- 执行maven命令
	- mvn sonar:sonar

---

#问题解决:
- 执行债务分析时，若报错，并且sonar后台日志（%SONARQUBE_HOME%/logs/sonar.log）中有如下错误
	`undefined method 'generate' for #<JSON::Ext::Generator::State:0x76afc32d>`

	**问题原因**：Sonar后台有Ruby on Rails工程，会用到Ruby的API，这个是Ruby的异常。由于部署的服务器刚好也装过Ruby版本，导致了调用自己安装的Ruby，而不是Sonar内嵌的Ruby，导致出现错误

	**解决方案**：在sonar的启动脚本soanr.sh的开头加入unset GEM_PATH ，在重新启动即可，通过这个命令可以保证每次启动的时候使用sonar内置的ruby版本
