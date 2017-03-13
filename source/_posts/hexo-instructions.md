title: Hexo安装维护说明
author: alpha
tags:
  - hexo
categories:
  - 说明文档
date: 2017-03-13 11:05:00
---
## 一 背景介绍
Hexo是一个开源的静态博客生成器，用node.js开发。原理是将source文件夹下的.md文章生成纯静态页面到public文件夹，并将public文件夹push到github上便可以被Github Pages识别了。

博客地址：[https://alpha-huang.github.io](https://alpha-huang.github.io/)

<!--more-->

## 二 配置环境

1.安装node（略）

2.安装git（略）

3.安装hexo

创建博客文件夹（例如blog，以下所有操作除特别说明外都是在该文件夹下执行）并安装hexo
```javascript
mkdir blog
cd blog
npm install hexo-cli –g
hexo init
```  
生成静态页面
```javascript
hexo g
```  
启动本地服务，进行文章预览调试

```javascript
hexo s
```  
浏览器访问 http://localhost:4000 能打开页面 hexo就搭建完成了

4.安装hexo-admin插件

```javascript
npm install --save hexo-admin
```  
启动本地服务后，浏览器访问 http://localhost:4000/admin 能打开页面 hexo就搭建完成了

5.从github更新文章和主题到本地

```javascript
git init
git remote add origin https://github.com/madeteam/hexo-source
git fetch
git reset --hard origin/master
```  
这里需要删除hexo默认的文章和主题
```javascript
rm source/_posts/hello-world.md
rm -r themes/landscape
```  
重新编译运行之后就能在本地预览最新的文章和主题了




## 三 编辑文章

1 .创建文章

**方法一：手动创建文件**

只需要将符合Markdown语法的.md文件放到 /source/_posts/ 目录下编译即可，需要注意的是

文章内容的头部需要包含 标题、作者、标签、分类、创建时间

例如：
```javascript
title: Hello World
author: alpha
tags:
  - 使用说明
  - 前端
  - 组件
categories:
  - 资源分享
date: 2016-12-15 16:13:00
---
<!--以下是正文-->
```  
在文章中加入下面的代码会让前半部分的内容显示在文章列表页而后半部分内容被隐藏
```javascript
<!--more-->
```  

**方法二：通过hexo-admin管理后台编辑**

可以直接打开hexo-admin管理后台创建和编辑文章，系统会在  /source/_posts/  下新建文件并且可以很方便地编辑作者、标签和分类等内容。但是由于hexo-admin这个插件还不完善，编辑文章功能比较弱，所以建议的方法就是先在km上编辑完成之后直接粘贴到hexo-admin里，点击publish，就可以在本地预览了，后面再修改也是实时自动编译的。

2 .插入图片

在HEXO中插入图片有两种方式

第一种：使用本地路径 将图片放到 /source/img/ 目录下即可
，插入图片时链接为
```javascript
![alt](/img/文件名)
```  

第二种：使用第三方图床 例如七牛和微博图床，使用七牛需要有已备案的域名，使用微博图床比较简单，在应用商店安装“微博图床”chrome插件后拖入图片，即可返回 Markdown 格式的链接。

上传图片这里还需要后续再看看有没有更好的方法。

# 四 发布文章和提交源文件

1 发布文章到github

由于hexo-admin插件还不完善，发布文章功能可能有bug，建议直接通过命令发布

```javascript
hexo d

```  
第一次发布需要安装一个插件
```javascript
npm install hexo-deployer-git --save

```  

然后输入github用户名和密码即可

2 提交文章源文件

为了便于多人维护，需要将源文件提交到github

主要是 Source 和 Themes 文件夹


```javascript
git add -A
git commit -m "hexo-admin"

```  
# 五 多人维护和多端维护

由于HEXO没有用户体系，所有人共用一个账号，当需要在另一台机器上编辑时就需要同步到最新的文章。这里使用github来管理文章源文件，每次编辑前只需要从github更新文章和主题到本地，编辑完成后再提交到github就可以了。

----

<br /><br /><br />


**文件目录介绍**

```javascript
|-- _config.yml	//全局配置文件
|-- package.json
|-- public 		//编译后生成的静态文件
|-- scaffolds 	 //新建文章时的格式模板
|-- source 		//文章源文件
|-- themes 		//主题
```  


**可能遇到的问题**

1 .发布文章到github时出现

```javascript
ERROR Deployer not found: git
```  
> 需要安装一个插件

```javascript
npm install hexo-deployer-git --save

```  
2 .如果出现

```javascript
git config --global user.name
```  

> 需要配置一下github用户名和邮箱

```javascript
git config user.name "madeteam"
git config user.email "madeteam2016@gmail.com"

```  
3 .想屏蔽某一篇文章的评论功能

```javascript
comments: false
```  

> 在这篇文章的.md文件头部加上 comments: false
 参数就可以了

----
<br />
相关链接：

 [hexo官方文档](hexo.io)

 [hexo-admin官方文档](http://jaredforsyth.com/hexo-admin/)
