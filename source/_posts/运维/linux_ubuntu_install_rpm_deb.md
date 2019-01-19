title: Linux_Ubuntu使用Deb安装rpm
tags: [Linux, Ubuntu, Deb, rpm]
date: 2016-03-18
categories: 运维
---

> ubuntu安装软件是用deb格式的文件安装，ubuntu对于rpm格式的文件安装软件是：
> 先将rpm格式的文件转换为deb格式的，再进行安装
<!-- more --> 
# Demo  
> ubuntu安装sqldeveloper-3.0.04.34-1.noarch.rpm

1. ubuntu 安装alien转换软件
	
		sudo apt-get install alien
2. 转换
       
		sudo alien --scripts sqldeveloper-3.0.04.34-1.noarch.rpm
       执行完后会生成一个sqldeveloper_3.0.04.34-2_all.deb这样的文件
       
3. 安装
   
   在图像画面上双击deb文件<br/>
   或者<br/>
   在终端里面输入
   
		sudo dpkg -i sqldeveloper_3.0.04.34-2_all.deb