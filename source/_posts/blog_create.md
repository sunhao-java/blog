title: 博客搭建过程
tags: [站务]
date: 2016-03-21
---

# 建站过程
## 选型
1. 本人积累了4年左右的技术文档
2. 苦于无处管理
3. 原先想在[开源中国][]上创建博客写写文章,并且也付之行动
4. 后来发现将这么多文档转移到开源中国上得工作量实在太大了
5. N久之前想学习Node.JS
6. 在同事的推荐下,知道了Hexo这个东西
7. 而且有很多theme可以使用,[有哪些好看的 Hexo 主题? - GitHub - 知乎][]
8. 而且所有的文档都是用markdown去写的
9. markdown的语法很简单,很容易入门上手.[Markdown 语法说明(简体中文版)][]
10. 遂,选之.

	PS: 文本文档转markdown的代价也大,但是值得!
	
<!-- more -->
	
## 环境搭建过程
1. 首先在本地搭建Node.JS开发环境
	- 去Node.JS[官网][]下载最新版本Node.JS,[下载地址][]
	- 这个是免安装版本,解压后配置环境变量即可

			vim ~/.bash_profile
			export NODE_JS_HOME=/Users/sunhao/Documents/tools/node-v5.6.0
			export PATH=$NODE_JS_HOME:$PATH
	- 然后执行:`source ~/.bash_profile`
	- 测试是否安装成功: `node -v`

			sunhaodeMacBook-Pro:~ sunhao$ node -v
			v5.6.0
			sunhaodeMacBook-Pro:~ sunhao$
2. npm相关
	- 很久以前的Node.JS安装后需要单独安装npm的
	- 现在不需要了,可以直接使用了
	- 由于我大天朝强大的GFW,npm的源在天朝局域网访问很不理想
	- 所以,阿里的大神们给大伙提供了一个npm registry镜像
	- 配置阿里npm镜像

			npm config set registry https://registry.npm.taobao.org
			npm config set disturl https://npm.taobao.org/dist
	- 这下npm的速度妥妥的
	- 参考文档:
		- [淘宝 NPM 镜像][]
		- [npm介绍][]
3. hexo相关
	- 前戏都做好了,我们就进入正事吧
	- 由于hexo是经常使用的,所以需要全局安装

			npm install hexo -g
	- 测试是否安装成功
		![](/imgs/hexo/1.png)

## 开始搭建博客
1. 选择一个目录
2. 进入之后执行`hexo init`
3. 等待步骤2下载一堆依赖后,即创建好基于Hexo的博客基本目录结构
	![](/imgs/hexo/2.png)
4. 简单应用
	- 直接在本机运行博客
	
			sunhaodeMacBook-Pro:test sunhao$ hexo serve
			INFO  Start processing
			INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
		这样在本机浏览器上直接访问http://localhost:4000即可以访问最简单的博客
	- hexo构建静态文件,可以放入nginx中直接访问(本人就是这种方式)
	
			hexo generate
		生成的文件放在`public`文件夹下
5. 自定义一些配置
	- 在`_config.yml`中进行配置
	- 参考[官方文档][]
6. 自定义主题
	- 主题请参考[有哪些好看的 Hexo 主题? - GitHub - 知乎][]
	- 主题放在`themes`文件夹下
	- 选择主题,在`_config.yml`中设置
	
			 # Extensions
			 ## Plugins: https://hexo.io/plugins/
			 ## Themes: https://hexo.io/themes/
			 theme: landscape
		即将`theme`设置为主题文件夹名
	- 主题配置:在每个主题文件夹下的`_config.yml`文件中
7. 我的主题是基于`yelee`添加了一些修改,后面会把这个修改后的主题放到git上
	- 原生的`yelee`: [yelee][]
	- 我修改后的`yelee`: `敬请期待`

## 如何进行更新博客
1. 将博客整个项目文件托管到Github或者Git@OSC中
2. 利用git的push钩子进行更新
3. 利用Node.JS在服务器的后台启用一个服务,去接收git钩子的请求,从而进行更新
4. 我就是采用这个方法的,所以我买了一个域名和一个阿里云服务器并且将域名备案了
5. 搭建过程
	- 将项目托管到git上,我选择的是Git@OSC,毕竟是天朝局域网直接互相访问,速度快
		![](/imgs/hexo/3.png)
	- 配置git的push钩子
		![](/imgs/hexo/4.png)
	- 在服务器上将Node.JS+Npm+Hexo的环境搭建好
	- 在服务器上将git上得博客pull下来,路径如:`/opt/blog`
	- 在服务器上利用Node.JS起一个服务

			var express = require('express');
			var app = express();
			var exec = require('child_process').exec;
			
			app.post('/hexo', function(req, res){
				// 打开目录,先执行git pull
				// 再执行hexo clean
				// 再重新生成public文件
				exec('cd /opt/blog/ && git pull && hexo clean && hexo generate', function (error, stdout, stderr) {
					console.log('stdout: ' + stdout);
					console.log('stderr: ' + stderr);
					if (error !== null) {
						console.log('exec error: ' + error);
					}
				});
			
				res.send('ok');
			});
			
			// 在3000端口启动监听服务
			var server = app.listen(3000, function() {
			    console.log('Listening on port %d', server.address().port);
			});
	- 注意,要写一个`package.json`文件,写明此服务所以来的组件

			{
			  "name": "gitlab-hexo-webhook",
			  "version": "1.0.0",
			  "description": "gitlab钩子",
			  "main": "hexo-webhook.js",
			  "dependencies": {
			    "debug": "^2.0.0",
			    "express": "^3.0.6"
			  },
			  "devDependencies": {},
			  "scripts": {
			    "test": "echo \"Error: no test specified\" && exit 1"
			  },
			  "author": "sunhao",
			  "license": "ISC"
			}

	- 利用Node.JS的`forever`在后台启动此服务

			forever -a start /opt/shell/gitlab-hexo-webhook/hexo-webhook.js
	- 这样每次在本地用markdown写好博客后push到git上,push钩子就会执行,去请求服务器上的这个服务,这个服务就会将博客更新后,重新generate新的`public`

## 部署到nginx上
1. 在服务器上安装nginx
2. 进行配置

		server {
			listen       80;
			# 您的域名
			server_name  blog.izufang.me;
			
			location / {
				// 您的博客public文件夹绝对路径
				root   /opt/blog/public;
				index  index.html;
				try_files $uri $uri/ /index.html;
				expires -1;
			}
		}
3. 在您的域名提供商设置域名解析即可


[开源中国]: http://www.oschina.net
[有哪些好看的 Hexo 主题? - GitHub - 知乎]: http://www.zhihu.com/question/24422335
[Markdown 语法说明(简体中文版)]: http://www.appinn.com/markdown/
[官网]: http://nodejs.org/
[下载地址]: https://nodejs.org/dist/v4.4.0/node-v4.4.0-darwin-x64.tar.gz
[淘宝 NPM 镜像]: http://npm.taobao.org/
[npm介绍]: http://www.jb51.net/article/50671.htm
[官方文档]: https://hexo.io/docs/configuration.html
[yelee]: https://github.com/MOxFIVE/hexo-theme-yelee