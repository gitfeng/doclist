---
title: hexo 配置及主题
date: 2015-01-17 11:20:00
categories: 应用
tags: [hexo,git]
description: Hexo 主题及相关功能配置

---

## 配置
rel：http://zipperary.com/2013/05/29/hexo-guide-3/

## 主题

### 哪找主题？

知乎 https://www.zhihu.com/question/24422335/answer/46357100


### 哪些主题？ 

- NexT [gitUrl](http://github.com/iissnan/hexo-theme-next) [deamonUrl](http://notes.iissnan.com)
- yelee [gitUrl](https://github.com/MOxFIVE/hexo-theme-yelee) [deamonUrl](http://moxfive.xyz/)


### 主题安装

1. 执行以下命令，安装主题到主题目录： ~/blog/themes

 | git clone https://github.com/MOxFIVE/hexo-theme-yelee.git themes/yelee

2. 修改 Hexo 根目录对应配置文件 _config.yml

 theme: yelee
 
3. 配置完成后执行 hexo c && hexo g重新发布新主题的内容
