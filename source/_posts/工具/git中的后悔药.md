title: git中的后悔药
tags: [git]
date: 2019-01-27
categories: 工具
---

> 当git提交上去一些错误代码或者一些敏感信息时，这时候你需要回滚，但是还是能在git的日志找到蛛丝马迹，这时候我们就需要后悔药了！

<!-- more -->

1. 首先，我们需要找到我们需要回滚到的提交点的hash，即`commit id`
    - 在git shell中进入我们的项目目录
    - 使用git log命令获取提交的历史找到需要回滚到的提交点(#是我添加的)

            [sunhao@sunhaode-MacBook-Pro docker-templates]$ git log
            # 这次提交包含了敏感信息，需要吃点后悔药
            commit 17449b6201cdf2429217dc056d1d66dbeb11f488 (HEAD -> master, origin/master, origin/HEAD)
            Author: 孙昊 <sunhao.java@gmail.com>
            Date:   Sun Jan 27 01:11:06 2019 +0800

                yapi
            
            # 需要回滚到这次提交
            commit bcb9f25c09b6cac68bff3f95ccbb2d8df9192414
            Author: 孙昊 <sunhao.java@gmail.com>
            Date:   Fri Jan 25 18:24:28 2019 +0800

                jenkins

            commit c9098ba3ec2745ad1107b64bea6e0a5f417d131b
            Author: 孙昊 <sunhao.java@gmail.com>
            Date:   Fri Jan 25 12:27:09 2019 +0800

                owncloud

2. 复制你需要回滚的commit id，输入复制commit id值，使用`git reset --hard commit_id`

        [sunhao@sunhaode-MacBook-Pro docker-templates]$ git reset --hard bcb9f25c09b6cac68bff3f95ccbb2d8df9192414
        HEAD is now at bcb9f25 jenkins

3. 然后只要强制提交就行了

        # --force 慎用，会使用你本地的代码仓库直接覆盖远端仓库
        git push origin HEAD --force

4. 将事先备份好的文件中的敏感信息去掉，再正常操作即可

        git add .
        git commit -m "yapi"
        git push

5. 这样远端仓库中的敏感信息就去掉了，并且在commit log中也看不见了，达到了“毁尸灭迹”的效果了
