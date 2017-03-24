title: git常用命令
author: alpha
tags:
  - git
  - github
categories:
  - 笔记
date: 2016-1-11 16:23:24
---
配置参数：

``` powershell
git config --global user.name "alpha"
git config --global user.email "huangdong2015@outlook.com"
```
<!--more-->

创建版本库

``` powershell
cd d:
cd git
mkdir test
cd test
pwd (显示当前目录)
git init （设置当前目录为git仓库）
```


提交文件

``` powershell
git add readme.txt （将某个文件添加到暂存区）
git add .（将工作区文件添加到暂存区）
git commit -m "注释"（将暂存区文件提交到分支）
git status （查看是否有文件未提交）
git diff readme.txt （查看文件修改）
git log （查看提交日志）
git log --pretty=oneline （查看简要提交日志）
git reset  --hard HEAD^ （恢复到上一次提交）
git reset  --hard HEAD^^ （恢复到上上次提交）
git reset  --hard HEAD~100 （恢复到100次前的提交）
cat readme.txt （查看文件内容）
git reflog （查看版本号）
git reset --hard 版本号 （恢复到某次提交）
git checkout --  readme.txt （撤销提交前的所有修改）
rm test.txt （从文件夹删除某个文件）
```


从GitHub更新文件到本地

``` powershell
$ git clone https://github.com/alpha-huang/alpha.git （下载代码到本地）
$ git remote -v （查看远程仓库）
$ git fetch origin master （从远程获取最新版本到本地）
$ git merge origin/master （远程和本地合并）
$ cd alpha （进入相应目录）
```


修改文件

``` powershell
git add . （添加到暂存区）
git commit -m "注释" （提交到本地代码库）
git push -u origin master （提交到GitHub）
输入github账号密码
```

放弃本地修改 强制更新
``` powershell
git fetch --all
git reset --hard origin/master
```

hexo源文章维护常用命令
``` powershell
cd alpha-huang-blog
npm install hexo-cli
hexo init
npm install --save hexo-admin

git init
git remote add origin https://github.com/alpha-huang/hexo-source
git fetch
git reset --hard origin/master
rm source/_posts/hello-world.md
rm -r themes/landscape
hexo g
hexo s
npm install hexo-deployer-git

git add .
git commit -m "hexo config and source"
git status
git push -u origin master
```

git使用简易指南 http://www.bootcss.com/p/git-guide/
