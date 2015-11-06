---
layout: post
title:  "Jekyll+github 博客搭建"
date:   2015-11-05 13:44:32
categories: 随笔
---
其实一直有搭建个博客的想法，域名也都买好...
记录下搭建的过程

##系统环境
* MacBook Pro OS X EI Capitan 版本10.11

##工具
* code编辑器[Sublime Text](http://sublimetext.com)
* markdown编辑器[Mou](http://www.macupdate.com/app/mac/40420/mou)
* Github客户端SourceTree
* Terminal(终端)

工具的安装比较简单，省略此过程

####第1步：注册github账号，新建blog仓库
1. 注册github账号
2. 新建Repository(仓库)
![image](https://github.com/hong4cong/hong4cong.github.io/raw/master/images/2015.11.05/0CDCD449-2BA2-4F8C-9523-ED880F7EAC33.png)
3. 修改Repository name输入`username.github.io`，username指的是你注册时的用户名
![image](https://github.com/hong4cong/hong4cong.github.io/raw/master/images/2015.11.05/BF03E8B7-CAE4-4E22-8E41-1ACCBFA98A61.png)
4. 点击项目右下角`Settings`进入，再选择`Automatic page generator`（页面自动生成器）
5. 选择好页面模板后，就可以通过username.github.io直接访问了

其实可以不通过新建Repository创建page，我们可以上[jekyllthemes](http://jekyllthemes.org/) fork一个github上jekyll主题模板，修改下Repository Name就可以了

####第2步：安装jekyll
sudo gem install jekyll

####第3步：添加多说评论
* 注册[多说账号](http://duoshuo.com/)
* 复制多说提供的代码，打开本地仓库_layouts/post.html文件粘贴代码

![image](https://github.com/hong4cong/hong4cong.github.io/raw/master/images/2015.11.05/8753F661-D207-48CE-A442-D145E8D27E74.png)


##结束
* 鼓捣jekyll3.0，导致之前选的theme build不过，各种error！最后选的是[pithy](http://jekyllthemes.org/themes/pithy/) 一路顺畅

