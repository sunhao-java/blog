title: windows7共享网络
tags: [windows7, 共享网络]
date: 2016-03-18
categories: 工具
---

1. 以管理员身份运行命令提示符：
  	
  		快捷键win+R→输入cmd→回车
  		
<!-- more -->
  		
2. 启用并设定虚拟WiFi网卡：
  		
  		运行命令： netsh wlan set hostednetwork mode=allow ssid=你的wifi名字 key=你的密码
3. 设置Internet连接共享：
  		
  		在“网络连接”窗口中，右键单击已连接到Internet的网络连接，选择“属性”→“共享” 勾上“允许其他........连接(N)” 并选择刚才建立的wifi(直接选本地连接即可)
4. 开启无线网络：
		
		继续在命令提示符中运行：netsh wlan start hostednetwork