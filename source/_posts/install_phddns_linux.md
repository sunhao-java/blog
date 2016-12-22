title: Linux下安装花生壳
tags: [Linux,花生壳]
date: 2016-03-14
---

1. 去花生壳官网下载花生壳for Linux（phddns-2.0.5.19225.tar.gz）

		D:\Software\OS\Linux\花生壳

2. 解压压缩包	
		
		tar zxvf phddns-2.0.5.19225.tar.gz

<!-- more -->

3. 转到解压包里面	cd phddns-2.0.2.16556/
4. ./configure 

    	PS：
	    如果出现下面的错误：
		checking for C++ compiler default output file name... configure: error: C++ compiler cannot create executables See `config.log' for more details.
		则是G++没有安装好，你需要安装好G++，这个可以参照：http://www.cnblogs.com/umasuo/archive/2012/06/12/ubuntu_install_gplusplus.html
		安装好后重新运行configure一下
5. 编译：make
6. 跳转到src目录,查看一下文件列表：cd src  ll
　  若列表中有：-rwxr-xr-x 1 root root 47736 Jun 12 11:46 phddns*
    说明软件编译好了
7. 运行：./phddns

PS：
> 由于是第一次运行，所以需要配置一下：

	Enter server address(press ENTER use phlinux3.oray.net):
	在这里输入服务器地址，这里直接回车就行了。
	
	Enter your Oray account:
	在这里输入花生壳的账号
	
	Password:
	然后是password

	Network interface(s):
	[wlan0] = [IP:192.168.1.100][MAC:fd2e028a:fd2e028b:fd2e028c:fd2e028d:fd2e028e:fd2e028f]
	[lo] = [IP:127.0.0.1][MAC:fd2e0262:fd2e0263:fd2e0264:fd2e0265:fd2e0266:fd2e0267]
	然后选择需要绑定的网卡，要是没有特殊的话，默认就可以了，我这里用的是无线，所以选择了wlan0

	Log to use(default /var/log/phddns.log):
	选择日志的保存地点

	Save to configuration file (/etc/phlinux.conf)?(yes/no/other):
	选择配置文件的保存地点，选择yes则直接保存到/etc/phlinux.conf，输入other可以指定文件，这里默认就可以了。

> 接下来程序开始运行，会出现以下东西：

	192.168.1.100
	NIC bind success
	defOnStatusChanged okConnecting
	defOnStatusChanged okRedirecting
	defOnStatusChanged okConnecting
	defOnStatusChanged okDomainListed
	defOnDomainRegistered umasuo.eicp.net
	defOnDomainRegistered umasuo.com
	defOnDomainRegistered www.umasuo.com
	defOnUserInfo <userInfo account='umasuo' login='umasuo'><ID>7554606</ID><Account>umasuo</Account><Password></Password><Email>liuquan89@gmail.com</Email><RegDate>1339458707</RegDate><Credit>0.0</Credit><Largess>0.0</Largess><IsEnable></IsEnable><PHServer>phcnc.oray.net</PHServer><IsEnterprise>0</IsEnterprise><Contactor>umasuo</Contactor><IsMale>1</IsMale><ServiceType>0</ServiceType><ClientIP>2105538930</ClientIP></userInfo>
	defOnAccountDomainInfo <domainInfo account='umasuo' login='umasuo'><roots><root><RootName>umasuo.com</RootName><RegDate>1339458812</RegDate><ExpireDate>0</ExpireDate><StatusCode>17</StatusCode><IsCnRoot>0</IsCnRoot></root></roots><domains><domain><DomainName>umasuo.eicp.net</DomainName><RegDate>1339458712</RegDate><Account>umasuo</Account><StatusCode>153</StatusCode><RootName>eicp.net</RootName><IsFree>1</IsFree></domain><domain><DomainName>umasuo.com</DomainName><RegDate>1339458820</RegDate><Account>umasuo</Account><StatusCode>25</StatusCode><RootName>umasuo.com</RootName><IsFree>0</IsFree></domain><domain><DomainName>www.umasuo.com</DomainName><RegDate>1339458820</RegDate><Account>umasuo</Account><StatusCode>25</StatusCode><RootName>umasuo.com</RootName><IsFree>0</IsFree></domain></domains><domainInfo>
	defOnStatusChanged okDomainsRegistered, UserType: 0
	看到上面这些就表示登录成功，这时候你可以ping一下你所绑定的域名，发现能够ping通了。

> 这个时候可以按ctrl+c先退出程序，将phddns拷贝到你希望的位置，例如：

	cp phddns /opt/
> 这种东西一般可以采用后台模式运行：

	/opt/phddns -c /etc/phlinux.conf -d
> 这样基本就可以了，如果有兴趣还可以将其配置自动启动。


	PS:
	设置开机启动：
	vi /etc/rc.local
	最后一行加上/opt/phddns -c /etc/phlinux.conf -d

	ps:
	刚刚想在ubuntu上安装一个花生壳，但是把花生壳下载下来以后想进行编译的时候出现了
	
	checking for C++ compiler default output file name... configure: error: C++ compiler cannot create executables
	See `config.log' for more details.
	大仙是G++没有装好，于是就装了一次G++
	
	ubuntu 不知道从哪个版本开始就自带了GCC，但是G++貌似没有，于是用
	
	apt-get install g++
	安装，出现下面错误：
	
	The following packages have unmet dependencies:
	g++ : Depends: g++-4.6 (>= 4.6.3-1~) but it is not going to be installed
	google-chrome-stable : Depends: libxss1 but it is not going to be installed
	E: Unmet dependencies. Try 'apt-get -f install' with no packages (or specify a solution).
	这个错误是因为有些需要的包没有
	尝试使用
	
	apt-get -f install
	把欠缺的包安装完毕。

	再次重试
	
	apt-get install g++
	安装成功
	运行
	
	g++ --version
	显示以下信息：
	
	g++ (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3
	Copyright (C) 2011 Free Software Foundation, Inc.
	This is free software; see the source for copying conditions.  There is NO
	warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
	说明G++安装成功了。