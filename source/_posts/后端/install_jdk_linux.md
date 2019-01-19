title: Linux上安装JDK
tags: [Java, Linux, JDK]
date: 2016-03-14
categories: 后端
---

1. 上传`jdk1.6.0_26.zip`

2. 解压后设置环境变量

	- /etc/rc.local

			export JAVA_HOME=/opt/jdk1.6.0_26
			export JRE_HOME=${JAVA_HOME}/jre
			export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
			export PATH=${JAVA_HOME}/bin:$PATH

	- ~/.bashrc

			export JAVA_HOME=/opt/jdk1.6.0_26
			export JRE_HOME=${JAVA_HOME}/jre
			export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib

			export PATH=${JAVA_HOME}/bin:$PATH