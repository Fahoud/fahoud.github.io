---
title: git博客分支管理
date: 2020-07-13 10:47:40
tags:
---


# git hexo部署到github的源码管理


## master用于部署hexo
> 每次写完文档后，不希望把所有数据都保存到服务器中，希望把源码和数据都保存到github中。
> 因为github规定了部署博客必需为master分支，所以用master部署hexo，用dev存储整个hexo文件夹

> 实现hexo d先执行以下命令安装插件
```bash
$ npm install hexo-deployer-git --save 
```


> _config.yml文件中部署配置
```bash
deploy:
  type: git
  repo:  
    gitee: https://gitee.com/Fahoud/Fahoud.git
    github: https://github.com/Fahoud/fahoud.github.io.git
  branch: master
```

> 更改文档内容或者主题设置后部署到github或者码云
```bash
 $ hexo claen
 $ hexo g
 $ hexo d
```

## dev分支存储源码

> 本地只有一条分支，为master

> 把整个hexo文件夹推送到github或者码云
```bash
$ git add .     #add所有更改
$ git commit -m 'Update xxx'   #commit
$ git push github master:dev    #推上github的dev分支
$ 
$ git add . 
$ git commit -m 'Update xxx'
$ git push gitee master:hexo   #推上gitee的hexo分支
```
