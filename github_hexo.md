---
title: GitHub+hexo 搭建个人博客
date: 2016-01-16 19:09:01
categories: 应用
tags: [hexo,git]
description: GitHub+Hexo 搭建免费的个人博客，这里介绍下基本操作包括 github、hexo配置，但不包括 hexo主题设置

---

参考：
```html
搭建流程 http://blog.netpi.me/%E5%AE%9E%E7%94%A8/hexo/
        http://www.cnblogs.com/zhcncn/p/4097881.html
        http://moxfive.xyz/
        http://ibruce.info/2013/11/22/hexo-your-blog/
```

## hexo
### 安装
#### npm 镜像服务器设置
参考链接：https://cnodejs.org/topic/4f9904f9407edba21468f31e
镜像使用方法（三种办法任意一种都能解决问题，建议使用第三种，将配置写死，下次用的时候配置还在）:
- 通过config命令

```bash
npm config set registry https://registry.npm.taobao.org 
npm info underscore （如果上面配置正确这个命令会有字符串response）
```
- 命令行指定

```
npm --registry https://registry.npm.taobao.org info underscore 
```
- 编辑 ~/.npmrc 加入下面内容

```
registry = https://registry.npm.taobao.org
```
搜索镜像: https://npm.taobao.org

建立或使用镜像,参考: https://github.com/cnpm/cnpmjs.org

#### hexo 安装

```bash
npm install hexo-cli -g
npm install hexo --save
# 这步骤是用于发布到 git
npm install hexo-deployer-git --save
```
## blog 内容管理

### 新建 blog

```bash
#这个 blog 是文件名
$ hexo init blog
$ cd blog
#安装node_modules，使代码支持 本机server浏览
$ npm install
$ hexo server
```

此时文件目录为

```
blog
\_config\_yml // 注配置文件
db.json // 数据
debug.log // 调试日志
\_node\_mudules // nodejs 相关依赖
package.json // 配置依赖
scaffolds // 脚手架 - 也就是一个工具模板
source // 存放blog正文的地方
themes // 存放皮肤的地方
```

默认访问 http://localhost:4000


### 页面打不开

如果你的电脑没有翻-墙 可能会打不开页面。因为页面中默认使用了ajax.google.com 下的js包。因此我们要把包删掉

解决办法：

进入你刚新建好的 blog根目录 ，进入 themes/landscape/layout/_partial
- 找到 after-footer.ejs ,把

```js
<script src="//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"> </script>
```

替换成

```js
<script src="http://cdn.bootcss.com/jquery/2.1.1/jquery.min.js" > </script>
```
- 找到 header.ejs

注释掉或者删掉 下面这句css引用

```html
<link href="//fonts.googleapis.com/css?family=Source+Code+Pro" rel="styleshee" type"”text/cs">
```

重新 hexo server 之后。访问 http://localhost:4000 就会看到blog主页了。

### 新建文章
当我们想写一篇blog时 在blog根目录下我们可以通过 

```bash
hexo new <title> 
```

指令来实现

例如我们想新建一篇主题为hello的blog

```bash
hexo new hello
```

输出信息如下

```bash
[info] File created at /Users/Night/Web/projects/java/temp/blog-test/source/_posts/hello.md
```

hexo会在 source/_posts/ 下新建hello.md 文件。

编辑 hello.md 就是编辑你的blog内容了 — markdown语法

hello.md 的文档和目录可以这样添加

```markdown
title: postName #文章页面上的显示名称，可以任意修改，不会出现在URL中
date: 2013-12-02 15:30:16 #文章生成时间，一般不改，当然也可以任意修改
categories: example #分类
tags: [tag1,tag2,tag3] #文章标签，可空，多标签请用格式，注意:后面有个空格
description: 附加一段文章摘要，字数最好在140字以内。

// 你的内容
<!--more--> // 以上为摘要
```

### 内容静态化：md文件->html

因为我们的blog要部署到github静态服务器上面，所有我们还要将页面进行静态化

hexo 为我们提供了 hexo g 的方法。进入blog根目录，执行

```bash
$ hexo g
```

会有如下提示信息

```bash
[info] Files loaded in 0.081s
[create] Generated: archives/index.html (48ms)
[create] Generated: archives/2014/index.html (10ms)
[create] Generated: archives/2014/10/index.html (7ms)
[create] Generated: index.html (9ms)
[create] Generated: 2014/10/19/hello-world/index.html (19ms)
[create] Generated: js/script.js (5ms)
.....
[create] Generated: fancybox/helpers/jquery.fancybox-buttons.js (0ms)
[create] Generated: fancybox/helpers/jquery.fancybox-media.js (1ms)
[create] Generated: fancybox/helpers/jquery.fancybox-thumbs.css (2ms)
[create] Generated: fancybox/helpers/jquery.fancybox-thumbs.js (1ms)
[info] 28 files generated in 0.565s
```

blog根目录下会生成public文件夹-里面就是刚才生成的静态文件

hexo 的详细实用说明请参看[官方文档](http://hexo.io/docs/)

以上我们就完成了本地的博客系统环境搭建，在 【github】-【将blog部署到 github】中我们会介绍如何将博客系统部署到 github 中。

### hexo常用命令

- hexo new "postName" #新建文章
- hexo new page "pageName" #新建页面
- hexo generate #生成静态页面至public目录
- hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
- hexo deploy #将.deploy目录部署到GitHub

```
关于hexo server：

启动本地服务器，用于预览主题。默认地址： http://localhost:4000/
预览的同时可以修改文章内容或主题代码，保存后刷新页面即可；
对 Hexo 根目录 _config.yml 的修改，需要重启本地服务器后才能预览效果。
```

常用复合命令：

- hexo d -g #生成加部署
- hexo s -g #预览加部署

简写：

- hexo n == hexo new
- hexo g == hexo generate
- hexo s == hexo server
- hexo d == hexo deploy

## github

### 注册GitHub
登录GitHub，注册自定义用户名如gitfeng，不建议用户名中有大写字母，下划线、短横线等非数字、小写字母字符。

### 申请代码 repository

在主页右下角创建New repository，name必须和用户名一致如gitfeng.github.io。

这里试过如果用户名中有大写字母，短横线等情况，会导致访问 http://{name}.github.io失败。

### 将blog部署到 github

部署到github 非常简单。因为hexo已经为你集成好了发布到github的配置。

我们只需要修改 blog 目录下的 _config.yml 文件。打开 _config.yml 找到如下配置

```bash
deploy:
  type: github
  repo: https://github.com/yourname/blog.git
```

修改 repo : 'your github repo'。回到 blog 目录 执行

```
hexo deploy
```

即可将本地 hexo 博客内容发布到 github平台上。
往后我们要做的就是用自己的域名指向 github。

```
yaml语法要求严格，注意空格。建议把github地址那条语句重新手写一遍，冒号后面要有一个空格。
配置那里的冒号后面是有一个半角的空格的.如果中间有空格用分号引用一下
```

## blog 系统更多功能

### 绑定域名
这里介绍万网购买域名的绑定规则, 有两种，
- 配一级域名 如 www.baidu.com , 这个验证通过
- 配二级域名 如 zhidao.baidu.com ，尚未验证

#### 配置步骤
1. 在万网搜索喜欢的域名购买
2. 如果配一级域名，则直接配域名与192.30.252.153，192.30.252.154 ip的绑定，如果执行二级域名，执行命令
    ping {name}.github.io 
    获取 ip 地址
3. 购买完成后，在万网提供的域名解析配置页面配置域名解析到 第2步中获得的 ip 地址
4. 可通过ping {域名} 判断域名解析是否生效，此过程需要等10分钟~2小时
5. 在 GitHub 的代码仓库根目录下加 CNAME 文件，文件中只写自己申请的域名，域名不需要包括 http://www.的前缀

可参考：http://jingyan.baidu.com/article/3c343ff70fb6e60d3779632f.html

#### hexo 下保持 CNAME 不被清理

在 source目录下添加 CNAME 文件，下次发布即可直接带上。REAMDME.md 等文件也可类似处理

### rss 安装
rss的安装非常简单 ，已经有人做好了插件，安装只需一步。

```
npm install hexo-generator-feed
```
启动服务器，用浏览器打开 http://localhost:4000/atom.xml， 就可以看到RSS已经生效了。

### sitemap

同样是一条命令，就可以完成。

```
npm install hexo-generator-sitemap
```
启动服务器，用浏览器打开 用浏览器打开 http://localhost:4000/sitemap.xml 发现site已经生效

### 评论-duoshuo
[Hexo使用多说教程](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9)


### 图片存储服务

[七牛](http://www.qiniu.com/pricing) 免费体验用户每月1G流量

## gitcafe

ref: http://ppting.me/2015/02/08/gitcafe/#通过DNSPOD分流

dns解析：207.226.141.135
