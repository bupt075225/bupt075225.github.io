##git 常用命令

把当前目录变成Git可以管理的仓库

> git init

把文件名为file的文件从工作区添加到仓库暂存区(stage/index)

> git add *file*

将暂存区中的所有内容提交到仓库当前分支

> git commit -m "*your modification instruction*"

查看当前本地仓库中文件的修改处于什么状态，是已添加到仓库暂存区，还是提交到仓库分支。

> git status


查看工作区和仓库里最新版本的差异

> git diff *filename*

将本地仓库与远程仓库github关联起来（前提是已在github上创建了与本地仓库同步更新的远程仓库），这样就可以把本地仓库的内容推送到github，如下示例为通过SSH地址进行关联。

> git remote add origin *git@github.com:bupt075225/businessblog.git*

第一次把本地仓库的所有内容推送到远程仓库，远程仓库的名字是origin，这是git默认的叫法，也可以修改为其它的，master表示本地主分支。

> git push -u origin master

以后就可用下面的命令把本地master分支的最新修改推送到远程仓库github


> git push origin master

 
从远程仓库克隆到本地仓库，创建一个文件夹用来存放从github克隆下来的源代码，在这个文件夹下执行以下命令：

> git clone *git@github.com:appfuse/appfuse.git*

从远程仓库获取最新的版本到本地仓库有两个命令:git fetch和git pull

> git fetch origin master:temp
> 
> git diff temp
> 
> git merge temp

还可以使用

> git pull origin master

首先从远程仓库下载origin的master分支最新版本,然后比较本地master分支和origin的masteer分支差异.最后进行合并.git fetch这个命令和git pull更安全一些,在merge前我们可以查看更新情况,然后再决定是否合并.

##在github上搭建一个blog网站
使用github + jekyll + markdown 搭建个人blog站点。在本地撰写blog并在上传到github前预览真实的发布显示效果。另外还可以增加评论和google analytics等其它功能。

在本地启动jekyll 服务的命令：bundle exec jekyll serve

##参考链接

1. [http://www.pchou.info/web-build/2014/07/04/build-github-blog-page-08.html](http://www.pchou.info/web-build/2014/07/04/build-github-blog-page-08.html "blog on github tutorial") 在windows环境中使用github + jekyll在github上搭建一个blog以及发布blog的本地测试环境
2. [http://joshualande.com/jekyll-github-pages-poole/](http://joshualande.com/jekyll-github-pages-poole/)  基于一个简洁的jekyll模板完善github blog
3. [http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)  一个介绍git的学习教程


