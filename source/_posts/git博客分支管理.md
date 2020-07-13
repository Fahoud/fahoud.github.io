---
title: git博客分支管理
date: 2020-07-13 10:47:40
tags:
---


# git hexo部署到github的源码管理


## master用于部署hexo
> 每次写完文档后，不希望把所有数据都保存到服务器中，希望把源码和数据都保存到github中。
> 因为github规定了部署博客必需为master分支，所以用master部署hexo，用dev存储整个hexo文件夹

> 更改文档内容或者主题设置后部署到github或者码云
```bash
 $ hexo claen
 $ hexo g
 $ hexo d
```

## dev分支存储源码

> 先创建本地分支
```bash
$ git switch -c dev  #旧版本git用git checkout -b dev
```
> 把整个hexo文件夹推送到github或者码云
```bash
$ git switch master  #分支dev切换至master（旧版本git checkout master）
$ git add .     #add所有更改
$ git commit -m 'Update'   #commit
$ git push github master    #推上github的master分支
```
