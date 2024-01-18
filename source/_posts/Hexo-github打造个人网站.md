---
title: Hexo-github打造个人网站
date: 2021-11-21 21:57:38
tags: Hexo
---

使用hexo+github打造个人网站：

1、安装node.js。输入`node -v`和`npm -v`，如果出现版本号，那么就安装成功了。

2、安装git，github账户new repo.

输入自己的项目名字，后面一定要加`.github.io`后缀，README初始化也要勾上。然后项目就建成了，点击`Settings`，向下拉到最后有个`GitHub Pages`，点击`Choose a theme`选择一个主题。然后等一会儿，再回到`GitHub Pages`，点击那个链接，就会出现自己的网页啦.

3、安装hero.

输入`npm i hexo-cli -g`安装Hexo。会有几个报错，无视它就行。

安装完后输入`hexo -v`验证是否安装成功。

然后就要初始化我们的网站，输入`hexo init`初始化文件夹，接着输入`npm install`安装必备的组件。

这样本地的网站配置也弄好啦，输入`hexo g`生成静态网页，然后输入`hexo s`打开本地服务器，然后浏览器打开[http://localhost:4000/](https://link.zhihu.com/?target=http%3A//localhost%3A4000/)，就可以看到我们的博客啦.

4、写文章并发布

首先在博客根目录下右键打开git bash，安装一个扩展`npm i hexo-deployer-git`。

然后输入`hexo new post "article title"`，新建一篇文章。

然后打开`D:\study\program\blog\source\_posts`的目录，可以发现下面多了一个文件夹和一个`.md`文件，一个用来存放你的图片等数据，另一个就是你的文章文件啦。

编写完markdown文件后，根目录下输入`hexo g`生成静态网页，然后输入`hexo s`可以本地预览效果，最后输入`hexo d`上传到github上。这时打开你的github.io主页就能看到发布的文章啦。

5、theme选择。

github上面很多的theme download, 选择 一个自己最喜欢的然后放进去就可以啦。
