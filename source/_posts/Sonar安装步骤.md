title: Sonar安装步骤
tags: [Sonar]
date: 2014-10-13
---

1. 下载**sonarqube-4.5.zip**和**sonar-runner-dist-2.4.zip**
	- 下载地址：
		- [http://www.sonarqube.org/downloads/](http://www.sonarqube.org/downloads/ "sonarqube")

<!-- more -->

2. 上传到服务器，解压
	- `unzip sonarqube-4.5.zip`
	- `unzip sonar-runner-dist-2.4.zip`

3. 设置环境变量

		export SONAR_RUNNER_HOME=/opt/sonar/sonar-runner-2.4
		export PATH=$SONAR_RUNNER_HOME/bin:$PATH

4. 修改配置
	- 修改sonarqube的配置`%SONARQUBE_HOME%/conf/sonar.properties`

			# User credentials.
			# Permissions to create tables, indices and triggers must be granted to JDBC user.
			# The schema must be created first.
	        sonar.jdbc.username=sonar
	        sonar.jdbc.password=sonar
			#----- MySQL 5.x
			sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance

	- 修改sonar-runner的配置`%SONAR_RUNNER_HOME%/conf/sonar-runner.properties`
			#----- Default SonarQube server
			sonar.host.url=http://localhost:9000

			#----- PostgreSQL
			#sonar.jdbc.url=jdbc:postgresql://localhost/sonar

			#----- MySQL
			sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&amp;characterEncoding=utf8
			#----- Global database settings
			sonar.jdbc.username=sonar
			sonar.jdbc.password=sonar

			#----- Default source code encoding
			sonar.sourceEncoding=UTF-8

			#----- Security (when 'sonar.forceAuthentication' is set to 'true')
			sonar.login=admin
			sonar.password=admin

5. 创建数据库，脚本如下：

		create database if not exists sonar character set utf8;
		CREATE USER 'sonar'@'%' IDENTIFIED BY 'sonar';
		CREATE USER 'sonar'@'localhost' IDENTIFIED BY 'sonar';
		grant all privileges on sonar.* to 'sonar'@'%' identified by 'sonar';
		grant all privileges on sonar.* to 'sonar'@'localhost' identified by 'sonar';
		flush privileges;

6. 启动sonar:
	- `%SONARQUBE_HOME%/bin/linux-x86-64/sonar.sh`
	- 请将%SONARQUBE_HOME%替换成sonar安装目录
	- linux-x86-64视具体操作系统而定

7. 汉化步骤
	- 下载汉化包
		[http://repository.codehaus.org/org/codehaus/sonar-plugins/l10n/sonar-l10n-zh-plugin/](http://repository.codehaus.org/org/codehaus/sonar-plugins/l10n/sonar-l10n-zh-plugin/ "sonar-l10n-zh-plugin")
	- 放入%SONARQUBE_HOME%/extensions/plugins/下
	- 重启server即可
