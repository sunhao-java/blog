title: Linux_Ubuntu_允许root登录
tags: [Linux, Ubuntu, root]
date: 2016-03-15
---

> ubuntu12.04默认是不允许root登录的，在登录窗口只能看到普通用户和访客登录。

1. 以普通身份登陆Ubuntu后我们需要做一些修改,
	
	普通用户登录后，修改系统配置文件需要切换到超级用户模式,在终端窗口里面输入: `sudo -s`.然后输入普通用户登陆的密码，回车即可进入 root用户权限模式.

	然后执行: `vi /etc/lightdm/lightdm.conf`
<!-- more -->
	增加 
		
		greeter-show-manual-login=true  
		allow-guest=false  

	修改完的整个配置文件是
	
		[SeatDefaults]
		greeter-session=unity-greeter
		user-session=ubuntu
		greeter-show-manual-login=true		#手工输入登陆系统的用户名和密码
		allow-guest=false					#不允许guest登录
2. 然后我们启动root帐号：
	
		sudo passwd root
	根据提示输入root帐号密码。
3. 重启ubuntu，登录窗口会有“登录”选项，点击“登录”选项，就会提示让输入用户名了。这时候我们就可以通过root登录了。


## 通过登陆界面进入ubuntu 14.04 的 root。
1. 打开终端。
2. 输入`sudo gedit /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf`
3. 在弹出的编辑框里输入：`greeter-show-manual-login=true` 保存关闭。
4. 再在中端中输入：`sudo passwd root`
5. 输入你想要的密码，关机重启在多出的登录框里输入root 还有你的密码就好了！