title: Linux上给新建的用户赋权限
tags: [Linux, 权限]
date: 2016-03-14
---

> ubuntu的超级管理员root用户默认是不启用的，root默认是空密码，所以root是不能使用的。

 1. 为root设置密码，在终端执行`sudo passwd root`指令后，系统会提示你设置一个root账号的密码。在你没有经过当前用户密码验证的时候，还会要求你先输入当前用户的密码，然后才能设置root帐号的密码。设置完成后，就可以在终端切换到root用户了。

		切换用户su root或者su sunhao
		
<!-- more -->


# linux下如何添加一个用户并且让用户获得root权限

### 添加用户，首先用adduser命令添加一个普通用户，命令如下：

		#adduser tommy  //添加一个名为tommy的用户
		#passwd tommy   //修改密码
		Changing password for user tommy.
		New UNIX password:     //在这里输入新密码
		Retype new UNIX password:  //再次输入新密码
		passwd: all authentication tokens updated successfully.

### 赋予root权限

1. 修改 /etc/sudoers 文件，找到下面一行，把前面的注释（#）去掉

		## Allows people in group wheel to run all commands
		%wheel    ALL=(ALL)    ALL

	然后修改用户，使其属于root组（wheel），命令如下：

		#usermod -g root tommy

	修改完毕，现在可以用tommy帐号登录，然后用命令 su - ，即可获得root权限进行操作。

2. 修改 `/etc/sudoers` 文件，找到下面一行，在root下面添加一行，如下所示：

		## Allow root to run any commands anywhere
		root    ALL=(ALL)     ALL
		tommy   ALL=(ALL)     ALL

	修改完毕，现在可以用tommy帐号登录，然后用命令 su - ，即可获得root权限进行操作。

3. 修改 `/etc/passwd` 文件，找到如下行，把用户ID修改为 0 ，如下所示：

		tommy:x:500:500:tommy:/home/tommy:/bin/bash

	修改后如下

		tommy:x:0:500:tommy:/home/tommy:/bin/bash

	保存，用tommy账户登录后，直接获取的就是root帐号的权限。

> 友情提醒：虽然方法三看上去简单方便，但一般不推荐使用，推荐使用方法二。