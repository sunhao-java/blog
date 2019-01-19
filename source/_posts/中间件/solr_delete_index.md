title: Solr删除数据的几种方式
tags: [Solr]
date: 2016-03-18
categories: 中间件
---

> 有时候需要删除 Solr 中的数据（特别是不重做索引的系统中，在重做索引期间）。删除一些 Solr无效数据（或不合格数据）。

<!-- more -->

# 删除 solr 中的数据有几种方式：

## 先来看 curl 方式：

	curl http://localhost:8080/solr/update --data-binary "<delete><query>title:abc</query></delete>" -H 'Content-type:text/xml; charset=utf-8'  
  
	#删除完后，要提交  
	curl http://localhost:8080/solr/update --data-binary "<commit/>" -H 'Content-type:text/xml; charset=utf-8'  


## 用自带的 post.jar，在 apache-solr-XXX\example\exampledocs 目录下：

	java -Ddata=args  -jar post.jar "<delete><id>42</id></delete>"  
  
	#怎么使用 post.jar 查看帮助    
	java -jar post.jar -help  

## 直接用 url，使用 stream 相关参数：
	
	比如：http://localhost:80/solr/update/?stream.body=<delete><query>*:*</query></delete>&stream.contentType=text/xml;charset=utf-8&commit=true

	stream 相关参数还有：
	stream.file=（服务器本地文件），
	stream.url 分别指到你的删除文本，这里是直接字符串内容用 stream.body 参数。
	commit 参数是指提交，提交了才能看到删除效果。

## 小结：
> 其实，方式1、2原理一样，直接 POST xml 数据过去。方式3就是直接可以告诉服务器从那些地方取删除的 xml 内容。

删除指令有两种:

1. 用 <id></id> 包装；
2. <query></query> 包装。

指令都很明显，一个是 id 值（是在 schema.xml 的 uniqueKey 所指字段的值，而不是索引内部的 docId）；query 值是查询串，如：title:"solr lucene"。