title: Linux_Ubuntu下安装配置Svn+Apache服务器
tags: [Linux, Ubuntu, Svn, Apache]
date: 2016-03-18
categories: 中间件
---

## 安装 Subversion
1. 安装 Subversion

		sudo apt-get install subversion
<!-- more -->
2. 创建仓库
	
		svnadmin create /opt/svn/message
	`/opt/svn/message`为svn仓库所要创建到的目录，如创建目录的位置需要root权限，使用`sudo svnadmin`

3. 修改配置文件 

		sudo vi /opt/svn/message/conf/svnserve.conf
		#去掉#[general]前面的#号  
		[general]  
		#匿名访问的权限，可以是read,write,none,默认为read  
		anon-access = none
		#认证用户的权限，可以是read,write,none,默认为write  
		auth-access = write
		#密码数据库的路径，去掉前面的#  
		password-db = passwd

4. 修改配置文件`passwd`

		[users]  
		admin = admin
	一定要去掉[users]前面的#,否则svn只能以匿名用户登录，客户端不会出现登录窗口，除非你的anon不为none,否则将返回一个错误。这里的密码是没有加密的，后面在apache时，会讲到用htppasswd生成加密的密码。
	
到这里，Svn服务器已经配置好了。一些命令:

	#停止Subversion服务器
	sudo killall svnserve
	#启动Subversion服务器
	svnserve -d -r /opt/svn/message
	其中-d表示在后台运行，-r指定服务器的根目录，这样访问服务器时就可以直接 用svn://服务器ip来访问了。

## 安装apache服务器(第34步骤可以省略)
1. 安装Apache服务器
	
		sudo apt-get install libapache2-svn
		sudo apt-get install apache2

	在`/etc/apache2/mods-available/dav_svn.load`最后一行加上一下一句话
	
		LoadModule authz_svn_module /usr/lib/apache2/modules/mod_authz_svn.so

2. 此时如果出现以下错误：
	
		sudo /etc/init.d/apache2 restart 时失败 
		提示：No apache MPM package installed 
	则还要下载一个文件,执行以下命令:
	
		sudo apt-get install apache2-mpm-worker

3. 添加subversion管理用户及svn组
	
		sudo adduser svn
		sudo addgroup svn svn

4. 版本仓库。
	
	版本仓库我们用第一步中所创建的，这时需要为它加一些用户权限<br/>
	这里要为apache用到的www-data添加权限
		
		sudo chown -R root:svn /opt/svn/message
		sudo chown -R www-data:www-data /opt/svn/message
	赋予组成员对所有新加入文件仓库拥有相应的权限
		
		sudo chmod -R g+rws /opt/svn/message

5. 添加用户并设置权限
	
	这里注意了,通过http访问的账号是Apachehttp验证的。
	通过svn://访问的账号是svn仓库conf目录下passwd指定的。两个是独立的认证方式。
	如果用同一个passwd文件 就会出现http跟svn只能有其中一个能访问
	因为htpasswd创建用户的密码是加密的,相反原始passwd文件里的用户是没加密的

	这里我们用htpasswd创建密码文件，取名pwdfile
	
		htpasswd -c /opt/svn/message/conf/pwdfile admin
	执行上面语句后，创建admin账号，以及会要输入密码等东东，安提示完成就行了。
	如果要加第二个用户，要把 -c 参数去掉，否则会覆盖掉前面文件。

6. 配置httpd.conf文件

		sudo vi /etc/apache2/mods-enabled/dav_svn.conf
	在最后面添加下面内容
		
		<Location /message>
			DAV svn
			SVNPath /opt/svn/message
			AuthType Basic
			AuthName "SVN 认证名称"
			AuthUserFile /opt/svn/message/conf/pwdfile
			#<LimitExcept GET PROPFIND OPTIONS REPORT>
				Require valid-user
			#</LimitExcept>
		</Location>


	此时 AuthUserFile ，要指定pwdfile

7. 重启apache和svn就可以了
	
		sudo /etc/init.d/apache2 restart
		sudo svnserve -d -r /opt/svn/message


8. 打开浏览器 访问: `http://sunhao-java.vicp.cc/message` 输入 admin 密码：admin 就可以进去了！
9. 用svn客户端访问: `svn://sunhao-java.vicp.cc/`  输入用户名和密码就可以进去了！



## dav_svn.conf文件

	#demo for svn
	#<Location /svn>
	#    DAV svn
	#    SVNPath /opt/test/demo 
	#    AuthType Basic
	#    AuthName "SVN 认证名称"
	#    AuthUserFile /opt/test/demo/conf/pwdfile
	#    #<LimitExcept GET PROPFIND OPTIONS REPORT>
	#        Require valid-user
	#    #</LimitExcept>
	#</Location>
	#message
	<Location /message>
	    DAV svn
	    SVNPath /opt/svn/message
	    AuthType Basic
	    AuthName "SVN 认证名称"
	    AuthUserFile /opt/svn/message/conf/pwdfile
	    #<LimitExcept GET PROPFIND OPTIONS REPORT>
	        Require valid-user
	    #</LimitExcept>
	</Location>
	
	#messageboard
	<Location /messageboard>
	    DAV svn
	    SVNPath /opt/svn/messageboard
	    AuthType Basic
	    AuthName "SVN 认证名称"
	    AuthUserFile /opt/svn/messageboard/conf/pwdfile
	    #<LimitExcept GET PROPFIND OPTIONS REPORT>
	        Require valid-user
	    #</LimitExcept>
	</Location>