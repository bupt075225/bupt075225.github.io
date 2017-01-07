## Github Pages、Jekyll及poole简介

Github不仅提供了免费源代码托管空间，[github pages](https://pages.github.com/)为我们提供静态页面的托管，允许大家在github上创建自己的博客网站或主面，而且免费，不限流量，还可以绑定自己的域名。

[Jekyll](https://jekyllrb.com/)是一个静态页面模板转换引擎，使用Liquid模板语言，只要上传符合jekyll规范的文件，github会用这种模板引擎为你转化成静态页面和网站。Jekyll是github上的一个开源项目。

[Poole](https://github.com/poole/poole)是github上一个开源项目，它提供Jekyll站点模板，风格简洁，可查看他的一个[DEMO](http://demo.getpoole.com)。

## 搭建blog网站

步骤1：登录github，在github上创建一个新git仓库，并按照"username.github.io"格式对仓库命名，其中username是登录github的账户名。

步骤2：将poole从github上下载到本地git仓库，再将本地git仓库上传到在github上步骤1创建的仓库中，然后打开浏览器访问 http://username.github.io就可以看到blog网站。

项目源码目录下的各个目录是按照[Jekyll的目录结构](https://jekyllrb.com/docs/structure/)组织的,下面介绍几个常用的目录和文件：

_posts文件夹下存放所有需要发布的博客文章，文件格式使用markdown，文件名需要按照“YEAR-MONTH-DAY-title.MARKUP”这样的格式命名，文件可以包含一个[YAML](http://yaml.org/)头部。

_config.yml是网站的配置文件，如下是文件中的部分内容：

    # Setup
    title:Bill Liu
    tagline:  'The personal website of Bill Liu'
    url:  http://bupt075225.github.io/businessblog/
    paginate: 1
    baseurl: /businessblog
    encoding: utf-8
    author:
      name:   Bill Liu
      url:https://twitter.com/mdo
      email:  liuchong.9@gmail.com

index.html、_layouts和_includes文件夹下存放了博客网站的前端页面。

## 定制

直接从poole项目上下载的模板是一个可用的博客网站，但需要做些定制修改让它成为你的专属个人博客。

第一步，增加一个归档页面，罗列出所有的博客文章。方法如下：

创建archive.md文件，用来显示文章动态列表：

    ---
    layout: page
    title: Archive
    ---
    
    ## Blog Posts
    
    {% for post in site.posts %}
    * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]>>({{ post.url }})
    {% endfor %}

文件中使用了Liquid语法。

第二步，增加一个导航条。修改_config.yml，在文件最后加上如下语句，指定导航条中要链接到哪些页面：

    # This is the list of pages to include in the header of the website.
    pages_list:{'About':'/businessblog/about','Archive':'/businessblog/archive'}

修改_layouts/default.html，通过一个循环创建导航条中的链接指向上面定义的pages_list中对应的页面。

    <header class="masthead">
    <h3 class="masthead-title">
      <a href="{{ site.baseurl }}" title="Home">{{ site.title }}</a>
      
      {% for page in site.pages_list %}
      &nbsp;&nbsp;&nbsp;<small><a href="{{ page[1]  }}">{{ page[0] }}</a></small>
      {% endfor %}
    </h3>
      </header>

第三步，使用Google Analytics。在Google Analytics注册博客网站后，Google会提供如下一份JS追踪代码，将它嵌入到网站的所有页面。

    <script>
      (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
      (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
      m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
      })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
    
      ga('create', 'UA-57926242-1', 'auto');
      ga('send', 'pageview');
    
    </script>

将这份代码放在 _includes/google_analytics.html文件中，最后在_layouts/default.html中使用如下命令包含该文件：{% include google_analytics.html %}

第四步，为网站增加评论系统。以前我使用的是[Disqus](https://disqus.com/),但后来被墙了，现在改用[Comm(ent|it)](https://commentit.io/)这个开源项目，它可以将评论提交到Github Pages仓库中。

将Comm(ent|it)集成到Github Pages的方法如下：

第一步，访问[Comm(ent|it)](https://commentit.io/)官网，使用github账号登录并授权。

第二步，Comm(ent|it)会自动生成与你的github账号关联的变量设置、表单源码、评论展示源码。将评论表单和展示源码插入Gighub Pages模板文件comments.html。

注意：Comm(ent|it)提供两种保存评论的方式，一种是将每篇文章的评论放在[YAML](http://yaml.org/)头部，另一种是将所有文章的评论放在[jekyll的数据文件](http://jekyllrb.com/docs/datafiles/)中。Comm(ent|it)生成的默认设置是使用的前一种方式。如果使用后一种方式，就删除commentitPath变量，然后在变量设置中添加commentitDatafile变量，变量取值是数据文件名，默认值是comments，数据文件一定要存在，即使最开始它是一个空文件。Comm(ent|it)会自动在Github Pages仓库中创建一个master_comments分支用来存放评论，每发表一条评论就会向分支发起一个commit。手动将分支合并到master主干后，就会在博客中显示出评论。

## 发表文章到博客网站

网站建好后，使用markdown编辑器书写文章并保存到_posts文件夹下，注意文件名必须为"年-月-日-文章标题.后缀名"的格式，如下所示：

![](http://i.imgur.com/hR96StY.png)

也可以在文章的头部加一个yaml文件头，用来设置一些元数据。它用三根短线“---”标记开始和结束，里面第一行设置一种元数据。

将本地git仓库文章提交到github上使用如下命令。

> git push origin master
