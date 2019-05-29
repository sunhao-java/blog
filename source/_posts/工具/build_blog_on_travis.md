title: 通过Travis来自动构建博客
tags: [站务]
date: 2018-12-13
categories: 工具
---

> 本文基于熟练搭建基于github+hexo搭建博客的基础上，并且对github简单操作，并且会使用通过ssh的方式提交代码到github上

<!-- more -->

# 准备工作
1. 通过github账号登录Travis
2. 并且启用已经在github上创建好的博客项目，如下图
![](/imgs/build_blog_by_travis/1.png)
3. 点击右侧的`Setting`，按下图配置（红框处，暂时可以不管）
![](/imgs/build_blog_by_travis/2.png)

# 修改原来的博客项目
1. 在根目录增加`.travis.yml`文件和`.travis`文件夹
2. 修改`.travis.yml`文件，内容如下
    ```
    language: node_js
    node_js: stable
    install:
      - npm install
    script:
      - hexo d -g
    ```
3. 我们用`hexo deploy`去发布博客，会发现Travis后台提示说没有权限
4. 这是因为我们用`git@github.com:[Your name]/[Your project].git`的方式去拉取代码的，并且在hexo的配置文件中，我们也是这样配置使用ssh方式拉取、推送代码的
5. 而Travis环境中没有对应的ssh key，所以不能推送代码。

# 为Travis创建SSH
1. 首先使用你已经生成好的ssh key私钥/公钥文件
2. 将公钥上传到github上
3. 将秘钥复制到博客目录下（有.git文件夹的那一层），并改名为`id_rsa_github`[随意]
4. 安装Travis Client
5. 首先确保已经安装了Ruby，并且大于等于2.0.0版本
    ```
    [sunhao@sunhaode-MacBook-Pro blog]$ ruby -v
    ruby 2.3.7p456 (2018-03-28 revision 63024) [universal.x86_64-darwin18]
    ```
6. 安装Travis Client
    ```
    # 如果是Mac或者Linux，需要加sudo
    gem install travis
    ```
7. 简称是否正确安装
    ```
    [sunhao@sunhaode-MacBook-Pro blog]$ travis version
    1.8.9
    ```
8. 开始加密SSH key
    ```
    # 如果是github项目，先用travis login登录，如图
    ```
![](/imgs/build_blog_by_travis/3.png)
    ```
    # 通过命令travis encrypt-file id_rsa_github -add加密私钥
    [sunhao@sunhaode-MacBook-Pro blog_hexo]$ travis encrypt-file id_rsa_github -add
    encrypting id_rsa_github for sunhao-java/blog_hexo
    storing result as id_rsa_github.enc
    storing secure env variables for decryption
    
    Make sure to add id_rsa_github.enc to the git repository.
    Make sure not to add id_rsa_github to the git repository.
    Commit all changes to your .travis.yml.
    ```
9. 这个时候，Travis会自动把一些参数上传到Travis的网站上了，就是第二张图展示的`暂时不需要关注的红框内容`，记住前面的key，下文需要用到
10. 然后复制生成的`enc`文件到`.travis`文件夹下
11. 并且删除原来的`id_rsa_github`文件

# 修改`.travis.yml`
1. 修改前面创建的`.travis.yml`文件
2. 这个文件里有可能有一段话，大概如下，可以删掉，是因为前面使用了--add参数
    ```
    openssl aes-256-cbc -K $encrypted_65cc9ea56965_key -iv $encrypted_65cc9ea56965_iv
    -in .travis/id_rsa_github.enc -out ~/.ssh/id_rsa_github -d
    ```
3. 在这个文件的末尾加上如下的命令：
    ```
    before_install:
      - openssl aes-256-cbc -K $encrypted_65cc9ea56965_key -iv $encrypted_65cc9ea56965_iv
      -in .travis/id_rsa_github.enc -out ~/.ssh/id_rsa_github -d
      - chmod 600 ~/.ssh/id_rsa_github
      - eval $(ssh-agent)
      - ssh-add ~/.ssh/id_rsa_github
      - cp .travis/ssh_config ~/.ssh/config
      - git config --global user.name \'Your Name\'
      - git config --global user.email \'Your Email\'
    ```
4. 附上上面的`.travis/ssh_config`文件内容：
    ```
    Host github.com
        User git
        StrictHostKeyChecking no
        IdentityFile ~/.ssh/id_rsa_github
        IdentitiesOnly yes
    ```

# 结尾
1. 至此，配置以及全部OK了
2. 可以写一篇博客，并push到github上
3. 然后到Travis的控制台观察,一次构建就被触发了
4. 稍等片刻就可以在博客上看到你刚刚写的文章了


# Tks，安利下我写的东西
- 博客：[https://www.crazy-coder.cn/](https://www.crazy-coder.cn/)
- 开源的代码：[https://github.com/lodsve/lodsve-framework](https://github.com/lodsve/lodsve-framework)
- 欢迎提Issue和PR
