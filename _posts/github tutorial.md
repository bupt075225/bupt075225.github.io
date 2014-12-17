##git 常用命令


[http://www.pchou.info/web-build/2014/07/04/build-github-blog-page-08.html](http://www.pchou.info/web-build/2014/07/04/build-github-blog-page-08.html "blog on github tutorial")

把当前目录变成Git可以管理的仓库

> git init

把文件添加到仓库,实际是添加到暂存区(stage/index)

> git add *file*

将文件提交到仓库，把暂存区中的所有内容提交到当前分支

> git commit -m "*your modification instruction*"

查看当前本地仓库中文件的修改处理什么状态，是已添加到仓库暂存区，还是提交到仓库分支。

> git status

将本地仓库与远程仓库github关联起来，这样就可以把本地仓库的内容推送到github

> git remote add origin *git@github.com:bupt075225/businessblog.git*

第一次把本地仓库的所有内容推送到远程仓库，远程仓库的名字是origin，这是git默认的叫法，也可以修改为其它的。

> git push -u origin master

以后就可用下面的命令把本地master分支的最新修改推送到远程仓库github


> git push origin master

 
从远程仓库克隆到本地仓库，创建一个文件夹用来存入从github克隆下来的源代码，在这个文件夹下执行以下命令：

> git clone *git@github.com:appfuse/appfuse.git*



