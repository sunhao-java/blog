title: 使用PowerMockito+MockServer测试外部服务
tags: [单元测试,mock]
date: 2019-10-10
categories: 后端
---

> 前言
>
>&nbsp;&nbsp;&nbsp;&nbsp;我们在项目开发中，很有可能会与第三方服务进行通讯，比如发短信、导入账单等等，这些服务写好后，如何能保证是OK的？这就需要进行**`单元测试`**，但是每次跑**`持续集成`**时，不可能真实的去调用这些发送短信、导入账单的真实接口，所以这里就需要用到强大的mock来模拟这些接口，以实现这些接口。
>&nbsp;&nbsp;&nbsp;&nbsp;所以这里我们用到了powermock和mockserver两个强大的工具。
>
> powermockito和mockito的区别：
>
1. mockito用代理实现mock，而powermockito是直接去修改字节码实现更强大的mock。
2. 所以说powermockito是mockito的一个更加强大的扩展。
3. mockito不能mock静态、final、私有方法等。
4. powermockito号称是**无所不能的PowerMock**,上面mockito的缺陷都可以很好的弥补掉。

<!-- more -->
##调用外部服务分两种情况
1. 第三方给我们client包
2. 直接向第三方提供的url发送post/get请求

##两种情况的具体测试方式
1. client
	- 第三方提供给我们client包，其实他们是将具体请求的url封装在这个jar包中，这时候我们没办法去修改请求的url，那我们应该如何去做呢？
	- 我们可以认为第三方给我们的client是经过他们自己的测试，完全可以保证其的正确性，所以那我们可以直接把这个client给mock掉，那就从client的入口处开始mock吧
	- 这里需要用到powermock（**无所不能的PowerMock**）去mock私有方法或者静态方法，关于powermock的介绍请参考[powermock](http://blog.csdn.net/jackiehff/article/details/14000779 "powermock")。
	- pom.xml的配置
		
			<!-- mock -->
			<dependency>
				<groupId>org.powermock</groupId>
				<artifactId>powermock-api-mockito</artifactId>
				<version>1.5.5</version>
				<scope>test</scope>
			</dependency>
			<dependency>
				<groupId>org.powermock</groupId>
				<artifactId>powermock-module-junit4</artifactId>
				<version>1.5.5</version>
				<scope>test</scope>
			</dependency>
			<dependency>
	            <groupId>org.powermock</groupId>
	            <artifactId>powermock-module-junit4-rule-agent</artifactId>
	            <version>1.5.5</version>
	            <scope>test</scope>
	        </dependency>
	- unit-test代码
		- 具体请参考`sharing\labs\powermock`和`coral.mashup.push`

2. url
	- 直接用url，我们可以将url换成我们自己的url，然后自己创建一个对应的虚拟服务，这个服务将按照指定的请求地址以及参数返回指定的数据格式。如果去创建这个服务？这里我们就用到了MockServer这个工具。
	- 如何去使用：
		1. pom.xml的配置
			
				<!-- mock server -->
		        <dependency>
		            <groupId>org.mock-server</groupId>
		            <artifactId>mockserver-netty</artifactId>
		            <version>3.4</version>
		            <scope>test</scope>
		        </dependency>
		        <dependency>
		            <groupId>org.mock-server</groupId>
		            <artifactId>mockserver-client-java</artifactId>
		            <version>3.4</version>
		            <scope>test</scope>
		        </dependency>
		        <dependency>
		            <groupId>org.mock-server</groupId>
		            <artifactId>mockserver-core</artifactId>
		            <version>3.4</version>
		            <scope>test</scope>
		        </dependency>
		2. unit-test代码
			- 具体请参考`coral.mashup.sms`
			- 这里需要注意一下，如何去替换url。我们url一般都会写在properties中，然后通过SystemConfig这个工具类去获取，所以这里想替换url，只需将SystemConfig工具类相应的方法给mock掉，调用方法的条件是传入对应的key，返回值就是你想要的value即可！
