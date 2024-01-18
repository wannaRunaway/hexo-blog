---
title: Git-git升级
date: 2021-12-07 15:24:27
tags: Git
---

mac上的git升级.

1、挂上vpn, 设置全局模式.

2、安装brew   

/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

3、安装git 

 brew install git

4、可以看到当前新git文件和旧git文件

which -a git

/usr/local/bin/git   /旧git 

/usr/bin/git            /新git

5、切换到之前git文件夹，删除git

sudo rm -rf git*

6、将新下载git overwrite到当前文件夹

 brew link --overwrite git

7、ls检查文件，git --version 查看当前git版本.
