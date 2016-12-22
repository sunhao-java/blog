title: 使用history.back(-1)的问题
tags: [frontend]
date: 2013-06-05 10:23
---

> 今天在调一个项目现场的问题，觉得有点记录下来的意义。

首先看看重现问题的步骤：

1. 选择一个事物分类进行查询，搜到一条记录，然后进入申请页面。
	
	![](/imgs/history/1.png)
	
2. 进入申请页面之后点返回，页面立即没有了（只在IE下会出现问题）而且页面上显示的是webpage has expired!页面已经过期。

	ps:返回按钮触发的事件是：`window.history.back(-1);`
	
	![](/imgs/history/2.jpg)
	
3. 解决办法：

	页面过期，我在想了是不是页面上的meta设置no-cache了，即：
		
		<meta http-equiv="Pragma" content="no-cache"/>
	然后我立即查看页面源代码，发现木有这个东西。我就纳闷了，那是怎么了呢。

	后来一想是不是我查询的时候提交了一个表单，这时候已经跳到我查询的目标页面(虽然还是同一个jsp页面)，这时候用history.back(-1)已经找不到参数了，所以会报页面找不到的问题。
	
	然后想到form表单里面的method="post"和method="get"的区别（具体的网上搜索去吧）。
	
	so, 我把搜索的form中method改成get。

然后就OK了，问题解决了。