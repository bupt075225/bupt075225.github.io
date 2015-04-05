## Github Pages、Jekyll及poole简介

Github不仅提供了免费源代码托管空间，[github pages](https://pages.github.com/)为我们提供静态页面的托管，允许大家在github上创建自己的博客网站或主面，而且免费，不限流量，还可以绑定自己的域名。

[Jekyll](https://github.com/jekyll/jekyll)是一个静态页面模板转换引擎，使用Liquid模板语言，只要上传符合jekyll规范的文件，github会用这种模板引擎为你转化成静态页面和网站。Jekyll是github上的一个开源项目,这里就不详细介绍。

[Poole](https://github.com/poole/poole)是github上一个开源项目，它提供Jekyll站点模板，风格简洁，可查看他的一个[DEMO](demo.getpoole.com)。

## 搭建blog网站

将poole从github上下载到本地git仓库，再将本地git仓库上传到在github上你创建的仓库中，过一会就可以看到你的博客网站了。

_posts文件夹下存放所有需要发布的博客文章，文件格式使用markdown。

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

直接从poole项目上下载的模板是一个可用的博客网站，但需要做些定制修改让他打上自己的烙印，真正成为属于自己的博客网站。

第一步，增加一个归档页面，罗列出所有的博客文章。方法如下：

创建archive.md文件，用来显示文章动态列表：

    ---
    layout: page
    title: Archive
    ---
    
    ## Blog Posts
    
    {% for post in site.posts %}
      * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
    {% endfor %}

文件中使用了Liquid语法。

第二步，增加一个导航条。修改_config.yml，指定导航条中要链接到哪些页面：

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

将这份代码放在 _includes/google_analytics.html文件中，最后在_layouts/default.html中使用如下命令包含该文件：

    {% include google_analytics.html %}

第四步，为网站增加评论系统。这里我使用的是[Disqus](https://disqus.com/),他是一个第三方评论托管系统。到官网注册一个账号后，在设置选项中有一项是“Add Disqus To Site”是用来设置你将到把评论模块加到哪个网站，设置完成后，点击“Install”，这时根据网站使用的那种建站平台来进行安装，我选的是“Universal Code”。

在_layouts/default.html中包含comments.html

    {% include comments.html %}

创建 _includes/comments.html，将Disqus提供的代码放在这个文件中：

    {% if page.comments %}
    <div id="disqus_thread"></div>
    <script type="text/javascript">
    /* * * CONFIGURATION VARIABLES * * */
    var disqus_shortname = 'username'; //replace example with your forum shortname
    
    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
    var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
    dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
    (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
    </script>
    <noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
    {% endif %}

`{% if page.comments %}` 用来设置页面是否包含评论功能。在index.html头部增加一行“comments: True”，就可以在所有发表的文章页面包含评论功能，而在about或archive页面不开启评论功能。

    ---
    layout: default
    title: Home
    comments: True
    ---

## 发表文章到博客网站

网站建好后，使用markdown编辑器书写文章并保存到_posts文件夹下，注意文件名必须为"年-月-日-文章标题.后缀名"的格式，如下所示：

![](http://i.imgur.com/hR96StY.png)

也可以在文章的头部加一个yaml文件头，用来设置一些元数据。它用三根短线“---”标记开始和结束，里面第一行设置一种元数据。

将本地git仓库文章提交到github上使用如下命令,gh-pages是一个没有父节点的分支名称，github规定只有在这个分支中的页面，才会生成网页文件。

> git push origin gh-pages

## 参考链接

1. [http://joshualande.com/jekyll-github-pages-poole/](http://joshualande.com/jekyll-github-pages-poole/)