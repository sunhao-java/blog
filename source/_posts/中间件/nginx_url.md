title: nginx过滤url实现前台js的配置问题
tags: [frontend, nginx]
date: 2016-03-18
categories: 中间件
---

> 我们在开发的过程中,可能需要一些配置,这些配置可能就是仅仅为了开发的方便,比方说,订单过期时间,生产环境需要半小时失效,但是真正开发时,我不可能等上个半小时,所以这个时间这个失效时间我们会写在配置文件中,这样开发环境和生产环境各一套配置,来回切换很方便的.

<!-- more -->

- 基于摘要里的,在Java后台实现很方便,只需要读取properties配置文件即可
- 但是在前台js,js是在浏览器里执行的,无法读取服务器上的配置,除非请求后台,但是每次的开销也是挺大的,所以这个想法被ps了
- 这时候可以利用nginx,前台静态页面是部署在nginx中,所以我们可以配置nginx过滤某个js的url,然后指向我们需要的文件
- 前台代码
```
index.html
<!-- 即配置文件 -->
<script src="/config.js"></script>
<!-- 动态加载js -->
<script type="application/javascript">
    if (config.devMode == 'dev') {
        loadJs("开发环境的js");
    } else {
        loadJs("开发环境的js");
    }

    function loadJs(url, callback) {
        // 实现
    }
</script>
```
- 配置文件(生产环境配置和开发环境的配置在不同路径下,但是文件名同名)
```
var config = {
    // 或者 prd
    devMode: 'dev',
    // 还可以配置请求后台的url前缀
    serverUrl: 'http://dev.company.com'
    // serverUrl: 'http://api.company.com'
}
```
- nginx的配置
```
server {
    listen       80;
    server_name  www.company.com;

    location / {
        root /Users/sunhao/Documents/company/project;
        index index.html;
        try_files $uri $uri/ /index.html;
        expires -1;
    }
}
server {
    listen       80;
    server_name  debug.company.com;

    location / {
        root /Users/sunhao/Documents/company/project;
        index index.html;
        try_files $uri $uri/ /index.html;
        expires -1;
    }

    location ~ .flower\.js$ {
        root /Users/sunhao/Documents/company/project/js;
    }
}
```
    - 前一个server配置的是生产环境,正常配置
    - 后一个,过滤flower.js,定向到另外一个文件夹下
- 访问www.company.com就是正式环境
- 访问debug.company.com就是开发环境了
- 这样就可以实现配置的功能了