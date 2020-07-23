title: git同时push到多个远端仓库
tags: [git]
date: 2020-07-23
categories: 工具
---

> 当你的项目同时放在两个远程仓库(例如：一个开源中国，一个github)上时，需要分别push两次，进行以下修改，即可一次push，推送到两个远程库！

<!-- more -->

## 添加第二个远端地址
```
git remote set-url --add origin git@gitee.com:xxx/xxx.git
```

## 有加，就有删
```
git remote set-url --delete origin git@gitee.com:xxx/xxx.git
```

## 查看远端分支
```
git remote -v

origin  git@github.com:xxx/xxx.git (fetch)
origin  git@github.com:xxx/xxx.git (push)
origin  git@gitee.com:xxx/xxx.git (push)
```

## 推送
```
git add .
git commit -m "commit message"
git push
```