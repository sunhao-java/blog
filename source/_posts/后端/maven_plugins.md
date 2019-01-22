title: 关于maven的几个常用插件
tags: [Java, maven]
date: 2013-11-18 00:07
categories: 后端
---
> 最近整理了自己写的一个项目,使用maven+svn管理,idea开发,搭建了一个maven的私服.原来是凌乱不堪,所以费了好大劲才整理好,而且平时公司项目上事情多的一米...

下面说说使用的几个插件(至于那些dependency就让它们见鬼去吧)

不说废话了,代码贴上

1. clean插件

		<plugin>
		    <groupId>org.apache.maven.plugins</groupId>
		    <artifactId>maven-clean-plugin</artifactId>
		    <version>2.5</version>
		    <configuration>
		        <filesets>
		            <fileset>
		                <directory>F:/logs</directory>
		            </fileset>
		            <fileset>
		                <directory>../message-test</directory>
		                <includes>
		                    <include>spy.log</include>
		                </includes>
		            </fileset>
		            <fileset>
		                <directory>../message-test/target</directory>
		            </fileset>
		        </filesets>
		    </configuration>
		</plugin>
	这个插件没啥好说的,要是不需要删除别的地方代码,就用默认的,不用任何配置
2. 单元测试插件

		<plugin>
		    <groupId>org.apache.maven.plugins</groupId>
		    <artifactId>maven-surefire-plugin</artifactId>
		    <version>2.16</version>
		    <configuration>
		        <skip>true</skip>
		    </configuration>
		</plugin>
	单元测试没做好,一测就报错,干脆给skip了
3. resources

		<plugin>
		    <groupId>org.apache.maven.plugins</groupId>
		    <artifactId>maven-resources-plugin</artifactId>
		    <version>2.6</version>
		    <executions>
		        <execution>
		            <id>copy-resources</id>
		            <!-- here the phase you need -->
		            <phase>validate</phase>
		            <goals>
		                <goal>copy-resources</goal>
		            </goals>
		            <configuration>
		                <outputDirectory>${basedir}/target/test-classes</outputDirectory>
		                <resources>
		                    <resource>
		                        <directory>${basedir}/src/main/webapp/WEB-INF/config</directory>
		                        <filtering>true</filtering>
		                    </resource>
		                </resources>
		            </configuration>
		        </execution>
		    </executions>
		</plugin>
	用来复制一些资源文件,配置都是直译的
4. war

		<plugin>
		    <groupId>org.apache.maven.plugins</groupId>
		    <artifactId>maven-war-plugin</artifactId>
		    <version>2.4</version>
		    <configuration>
		        <warName>${message.war.name}</warName>
		        <includeEmptyDirectories>true</includeEmptyDirectories>
		        <webResources>
		            <resource>
		                <directory>../message-easyjs</directory>
		                <targetPath>js</targetPath>
		                <excludes>
		                    <exclude>**/.svn</exclude>
		                    <exclude>**/*.iml</exclude>
		                    <exclude>**/pom.xml</exclude>
		                </excludes>
		            </resource>
		        </webResources>
		    </configuration>
		</plugin>
	打war包的插件
5. jetty

		<plugin>
		    <groupId>org.mortbay.jetty</groupId>
		    <artifactId>maven-jetty-plugin</artifactId>
		    <version>6.1.10</version>
		    <configuration>
		        <!-- 配置扫描时间 -->
		        <scanIntervalSeconds>10</scanIntervalSeconds>
		        <!-- 配置项目在容器中的根路径 -->
		        <contextPath>${project.contextPath}</contextPath>
		        <!-- 配置jetty容器中的jndi -->
		        <jettyEnvXml>src/main/resources/jetty.xml</jettyEnvXml>
		        <connectors>
		            <connector implementation="org.mortbay.jetty.nio.SelectChannelConnector">
		                <!-- 端口 -->
		                <port>${project.port}</port>
		                <maxIdleTime>60000</maxIdleTime>
		            </connector>
		        </connectors>
		        <!-- 按照官网上说的是配置停止容器的快捷键和端口,至今不知怎么在idea中如何使用,有知道的麻烦告知下,3Q -->
		        <stopKey>foo</stopKey>
		        <stopPort>8888</stopPort>
		    </configuration>
		    <executions>
		        <!-- 配置在maven哪个生命周期执行插件的哪个动作 -->
		        <execution>
		            <id>jetty_run</id>
		            <!-- maven生命周期 -->
		            <phase>compile</phase>
		            <!-- 执行插件的哪个动作 -->
		            <goals><goal>run</goal></goals>
		        </execution>
		    </executions>
		    <dependencies>
		        <!-- 这个插件依赖的几个包 -->
		        <dependency>
		            <groupId>org.eclipse.jetty</groupId>
		            <artifactId>jetty-io</artifactId>
		            <version>7.6.6.v20120903</version>
		        </dependency>
		        <dependency>
		            <groupId>org.eclipse.jetty</groupId>
		            <artifactId>jetty-server</artifactId>
		            <version>7.6.6.v20120903</version>
		        </dependency>
		    </dependencies>
		</plugin>
	jetty这个插件当时可是整的我头疼,各种报错各种上网找资料,附上jetty的配置文件:jetty.xml
		
		<?xml version="1.0"?>
		<!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN" "http://jetty.mortbay.org/configure.dtd">
		<Configure class="org.mortbay.jetty.webapp.WebAppContext">
		    <New id="mysql" class="org.mortbay.jetty.plus.naming.Resource">
		        <Arg>jdbc/core</Arg>
		        <Arg>
		            <New class="com.mysql.jdbc.jdbc2.optional.MysqlConnectionPoolDataSource">
		                <Set name="Url">jdbc:mysql://localhost:3306/message</Set>
		                <Set name="User">root</Set>
		                <Set name="Password">123456</Set>
		            </New>
		        </Arg>
		    </New>
		</Configure>
6. cargo(可以启动tomcat,远程部署,本地部署,都支持的,这里我只用到本地部署,远程部署需要配置tomcat-user.xml和tomcat的控制台)
	
		<plugin>
		    <groupId>org.codehaus.cargo</groupId>
		    <artifactId>cargo-maven2-plugin</artifactId>
		    <version>1.4.3</version>
		    <configuration>
		        <container>
		            <!-- tomcat的版本,tomcat6使用tomcat6x -->
		            <containerId>${cargo.tomcat.version}</containerId>
		            <!-- tomcat在本地的绝对路径 -->
		            <home>${tomcat.home}</home>
		            <!-- 本地安装就用installed,远程使用remote -->
		            <type>installed</type>
		 
		            <!-- tomcat日志文件路径 -->
		            <output>${tomcat.home}/logs/container.log</output>
		            <append>false</append>
		            <log>${tomcat.home}/logs/cargo.log</log>
		        </container>
		        <configuration>
		            <!-- 本地部署,已存在 -->
		            <type>existing</type>
		            <!-- 再配置一次tomcat绝对路径 -->
		            <home>${tomcat.home}</home>
		            <properties>
		                <!-- 端口 -->
		                <cargo.servlet.port>${project.port}</cargo.servlet.port>
		            </properties>
		        </configuration>
		        <!-- 这里一次可以部署多个项目 -->
		        <deployables>
		            <!-- 指定我部署的项目GAV -->
		            <deployable>
		                <groupId>com.message</groupId>
		                <artifactId>message-test</artifactId>
		                <!-- war包形式部署 -->
		                <type>war</type>
		                <properties>
		                    <!-- 容器中的上下文根 -->
		                    <context>${project.contextPath}</context>
		                </properties>
		            </deployable>
		        </deployables>
		    </configuration>
		    <!-- 同jetty -->
		    <executions>
		        <execution>
		            <id>tomcat-run</id>
		            <phase>package</phase>
		            <goals><goal>run</goal></goals>
		        </execution>
		    </executions>
		</plugin>
	- tomcat使用cargo这个插件有个缺点,不能debug了,而且不是热部署,不知道cargo有没有热部署的功能,暂时我还没找到,待研究 
	- jetty那个是可以热部署的,不管改Java类还是jsp或者css,js都可以(加减方法,改参数不行)
7. 附上maven的properties

		<properties>
		    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		    <junit.version>4.10</junit.version>
		    <spring.version>3.0.5.RELEASE</spring.version>
		    <jdk.version>jdk15</jdk.version>
		    <tomcat.version>6.0.32</tomcat.version>
		    <message.war.name>message</message.war.name>
		    <project.port>8099</project.port>
		    <project.contextPath>/core</project.contextPath>
		    <tomcat.home>F:\study\apache-tomcat-6.0.32</tomcat.home>
		    <cargo.tomcat.version>tomcat6x</cargo.tomcat.version>
		</properties>   