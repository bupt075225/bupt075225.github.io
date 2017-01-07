##git 常用命令

1.**初始化git仓库**

把当前目录变成Git可以管理的仓库

> git init

通过SSH或HTTPS从远程git服务器克隆一个远程仓库：

> git clone username@hostname/IP:/path/xxx.git
> 
> git clone https://xxx.com

2.**提交更新**

把文件名为file的文件从工作区添加到仓库暂存区(stage/index)

> git add *file*

将暂存区中的所有内容提交到仓库当前分支

> git commit -m "*your modification instruction*"

查看当前本地仓库中文件的修改处于什么状态，是已添加到仓库暂存区，还是提交到仓库分支。

> git status

将本地仓库中的更新提交到远程仓库：

> git push 远程git服务器名 本地分支名:远程分支名

这条命令表示的意思是，将源地直为本地分支的更新推送到远程git服务器上，目的地址为远程分支。如果本地分支和远程分支建立了跟踪关系，可以省略远程分支名，一个示例：

> git push origin master:master

第一次把本地仓库的所有内容推送到远程仓库，远程仓库所在的服务器在本地的别名是origin，这是git默认的叫法，也可以修改为其它的，master是远程仓库的主分支。-u选项设置本地分支去跟踪远程对应的分支，设置好跟踪的分支后，就可以使用git push命令省去指定推送分支的参数。

> git push -u origin master

以后就可用下面的命令把本地master分支的最新修改推送到远程仓库github

> git push origin master

查看工作区和仓库里最新版本的差异

> git diff *filename*

将本地仓库与远程仓库github关联起来（前提是已在github上创建了与本地仓库同步更新的远程仓库），这样就可以把本地仓库的内容推送到github，如下示例为通过SSH地址进行关联。

> git remote add origin *git@github.com:bupt075225/businessblog.git*
 
3.**拉取更新**

从远程仓库获取最新的版本到本地仓库有两个命令:git fetch和git pull

> git fetch 远程git服务器 远程分支:本地分支

这条命令的意思是，从远程git服务器上抓取本地没有的数据，源地址是远程分支，目的地址是本地分支。

一个示例：
> git fetch origin master:temp
> 
> git diff temp
> 
> git merge temp

还可以使用

> git pull origin master

首先从远程仓库下载origin的master分支最新版本,然后比较本地master分支和origin的master分支差异.最后进行合并.git fetch这个命令比git pull更安全一些,在merge前我们可以查看更新情况,然后再决定是否合并.

## git进阶命令

1.**版本回退**

如果暂存区的修改通过commit命令提交到了仓库，现在又想回退到之前某个已commit的版本，应该使用什么命令呢？

首先用git log命令查看提交历史记录，确定要回退到哪个版本。

> git log

在git中，HEAD表示当前版本，上一个版本是HEAD^，上上一个版本是HEAD^^,继续下去就是从三到万了，往回100个版本可写为HEAD~100。使用git reset命令进行版本回退，通过HEAD指定回退到哪个版本，如下所示回退三个版本：

> git reset --hard HEAD~3

上面是回退到过去的某个版本，还可以再回到未来的某个版本，就像在古今来回穿越。穿越回现代就得知道现在对应的哪个commit id，仍然使用上面这个命令，只是将HEAD换成commit id。

2.**使用分支**

通过clone命令创建主分支后，在主分支文件根目录下执行下面的命令创建分支：

> git checkout -b new-branch-name master

git checkout命令加上-b选项表示创建并切换(相当于git branch new-branch-name和git checkout new-branch-name两条命令)，这个命令检出并创建一个基于master分支，名为new-branch-name的分支。-b选项表示如果分支还不存在就新建分支。

下面是一个创建分支的操作示例：

![](http://i.imgur.com/Rl2dUYD.png)

创建完分支或在分支上提交修改后要push到服务器中央仓库,否则这个分支永远只是隐藏在创建者的本地,其他协作同事看不到:

> git push -u origin new-branch-name

协作开发时,新加入的同事要基于上述新建分支开发,就必须将分支拉到本地,使用如下命令拉取分支到本地:

> git fetch origin
> 
> git checkout -b new-branch-name origin/new-branch-name

首先使用git fetch命令从远程git服务器抓取本地没有的数据，它不会修改工作区域中的内容，只会获取数据让你自己合并。

然后git checkout命令在远程分支origin/new-branch-name基础上创建新的本地分支并切换到新建分支上。

如果新建分支前不抓取更新，git checkout命令可能会报如下的错误:

![](http://i.imgur.com/pQbw6mA.png)

可以尝试使用如下的命令检出分支到本地:

> git fetch && git checkout develop-branch

![](http://i.imgur.com/NiavH40.png)

> git branch

上面这个命令会列出所有分支，当前分支前会标一个*号，如下图所示：

![](http://i.imgur.com/C9Y7dPF.png)

> git branch -vv

-vv参数用于显示更详细的分支信息，包括分支与它跟踪的远程分支对应关系。

> git branch -d <branch-name>

删除分支

> git checkout <branch-name>

切换分支

> git merge <branch-name>

将branch-name分支合并到当前分支

![](http://i.imgur.com/5G7VTZo.png)



##参考链接

3. [http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)  一个介绍git的学习教程


