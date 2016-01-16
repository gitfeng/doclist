# github + hexo

参考：

```html
http://blog.netpi.me/%E5%AE%9E%E7%94%A8/hexo/
http://www.cnblogs.com/zhcncn/p/4097881.html
http://moxfive.xyz/
```
## hexo 
### 1.1 安装
#### 1.1.1 npm 镜像服务器设置
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

#### 1.1.2 hexo 安装

```bash
npm install hexo-cli -g
npm install hexo --save
npm install hexo-deployer-git --save
```
### 2 blog 内容管理

#### 2.1 新建 blog

```bash
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```
此时文件目录为

blog
\_config\_yml // 注配置文件
db.json // 数据
debug.log // 调试日志
\_node\_mudules // nodejs 相关依赖
package.json // 配置依赖
scaffolds // 脚手架 - 也就是一个工具模板
source // 存放blog正文的地方
themes // 存放皮肤的地方

默认访问 http://localhost:4000

#### 2.2 页面打不开

如果你的电脑没有翻-墙 可能会打不开页面。因为页面中默认使用了ajax.google.com 下的js包。因此我们要把包删掉

解决办法：

进入你刚新建好的 blog根目录 ，进入
themes/landscape/layout/_partial
1，找到 after-footer.ejs

把

```js
<script src="//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"> </script>
```

替换成

```js
<script src="http://cdn.bootcss.com/jquery/2.1.1/jquery.min.js“ > </script>
```

2，找到 header.ejs

注释掉或者删掉 下面这句css引用

```html
<link href="//fonts.googleapis.com/css?family=Source+Code+Pro" rel=”stylesheet” type=”text/css”>
```

重新 hexo server 之后。访问 http://localhost:4000 就会看到blog主页了。

#### 2.3 新建文章
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
title: hexo、github、多说、搭建免费博客
date: 2014-10-19 12:56:58
tags:
- tag1
- tag2
- tag3
categories:
- 目录
__

// 你的内容
<!--more--> // 以上为摘要
```

#### 2.4 静态处理

因为我们的blog要部署到github静态服务器上面，所有我们还要将页面进行静态化

hexo 为我们提供了 hexo g 的方法。进入blog根目录 执行

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
[create] Generated: css/style.css (434ms)
[create] Generated: css/fonts/FontAwesome.otf (1ms)
[create] Generated: css/fonts/fontawesome-webfont.eot (2ms)
[create] Generated: css/fonts/fontawesome-webfont.svg (2ms)
[create] Generated: css/fonts/fontawesome-webfont.ttf (4ms)
[create] Generated: css/fonts/fontawesome-webfont.woff (1ms)
[create] Generated: css/images/banner.jpg (3ms)
[create] Generated: fancybox/blank.gif (2ms)
[create] Generated: fancybox/fancybox_loading.gif (1ms)
[create] Generated: fancybox/fancybox_loading@2x.gif (1ms)
[create] Generated: fancybox/fancybox_overlay.png (1ms)
[create] Generated: fancybox/fancybox_sprite.png (0ms)
[create] Generated: fancybox/fancybox_sprite@2x.png (1ms)
[create] Generated: fancybox/jquery.fancybox.css (0ms)
[create] Generated: fancybox/jquery.fancybox.js (1ms)
[create] Generated: fancybox/jquery.fancybox.pack.js (1ms)
[create] Generated: fancybox/helpers/fancybox_buttons.png (1ms)
[create] Generated: fancybox/helpers/jquery.fancybox-buttons.css (1ms)
[create] Generated: fancybox/helpers/jquery.fancybox-buttons.js (0ms)
[create] Generated: fancybox/helpers/jquery.fancybox-media.js (1ms)
[create] Generated: fancybox/helpers/jquery.fancybox-thumbs.css (2ms)
[create] Generated: fancybox/helpers/jquery.fancybox-thumbs.js (1ms)
[info] 28 files generated in 0.565s
```

blog根目录下会生成public文件夹-里面就是刚才生成的静态文件

hexo 的详细实用说明请参看[官方文档](http://hexo.io/docs/)

## github
### 注册设置GitHub
登录GitHub，注册自定义用户名如wsgzao
在主页右下角创建New repository，name必须和用户名一致如wsgzao.github.io
这里我试过如果用户名中有大写字母，短横线等情况，会关联失败。
### 3 将blog部署到 github
部署到github 非常简单。因为hexo已经为你集成好了发布到github的配置。

我们只需要 修改 blog 目录下的 _config.yml 文件

打开 _config.yml 找到如下配置

```bash
deploy:
  type: github
  repo: https://github.com/yourname/blog.git
```

修改 repo : ‘your github repo’

回到 blog 目录 执行

```
hexo deploy
```

你会发现public 目录下的页面已经发布到github gh-pages 分支了

往后我们要做的就是用自己的域名指向 github。

yaml语法要求严格，注意空格。建议把github地址那条语句重新手写一遍，冒号后面要有一个空格。

配置那里的冒号后面是有一个半角的空格的.如果中间有空格用分号引用一下

## 写文章
除了正常的 markdown 语法外，在 github 上基于 hexo 部署的 blog, .md 文件还支持一下字段

```
title: postName #文章页面上的显示名称，可以任意修改，不会出现在URL中
date: 2013-12-02 15:30:16 #文章生成时间，一般不改，当然也可以任意修改
categories: example #分类
tags: [tag1,tag2,tag3] #文章标签，可空，多标签请用格式，注意:后面有个空格
description: 附加一段文章摘要，字数最好在140字以内。
```

