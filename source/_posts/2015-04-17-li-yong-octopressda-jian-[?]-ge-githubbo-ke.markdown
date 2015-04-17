---
layout: post
title: "利用Octopress搭建一个Github博客"
date: 2014-12-03 22:08:47 +0800
comments: true
categories: 转载

---

###小引
`Octopress`是利用`Jekyll`博客引擎开发的一个博客系统，生成的静态页面能够很好的在github page上展现。号称是hacker专属的一个博客系统(`A blogging framework for hackers.`)
根据大家的反应，本文我就来介绍一下如何在苹果电脑(OS X 10.8.3)利用Octopress搭建一个Github博客。本文需要读者熟悉一些shell命令，并掌握基本的git操作。

<!--more-->

###目录
1. 安装Ruby
2. 安装Octopress
3. 配置Octopress
4. 将博客部署到GitHub上
5. 开始写博客
6. 更多操作
7. 小结

###1.安装Ruby
Octopress需要Ruby环境，RVM(Ruby Version Manager)负责安装和管理Ruby的环境。所以我们先在终端输入如下命令，来安装RVM：

```   
$ curl -L https://get.rvm.io | bash -s stable --ruby
```

接着是安装Ruby 1.9.3，在终端依次运行如下命令： 
     
```
$ rvm install 1.9.3                                                   
$ rvm use 1.9.3                
$ rvm rubygems latest
```  

完成上面的操作之后，运行`ruby --version`应该可以看到ruby 1.9.3环境已经安装好了。   
参考：[Installing Ruby With RVM](http://octopress.org/docs/setup/rvm/)

###2.安装Octopress

在安装Octopress之前，请确保你的电脑上已经安装有git了，在终端输入`git --version`，应该可以看到电脑中的git版本(我电脑上输出:`git version 1.7.12.4 (Apple Git-37)`)，如果没有显示相关内容，请先安装git。
git安装之后，利用git命令将octopress从github上clone到本机，如下命令：

```
$ git clone git://github.com/imathis/octopress.git octopress                
$ cd octopress    # If you use RVM, You'll be asked if you trust           the .rvmrc file (say yes).   
$ ruby --version  # Should report Ruby 1.9.3
```
接着安装相关依赖项：

```
$ gem install bundler
$ rbenv rehash    # If you use rbenv, rehash to be able to run   $ the bundle command
$ bundle install
```
最后安装默认的Octopress 主题。

```
$ rake install
```

参考： [Octopress Setup](http://octopress.org/docs/setup/)

###3.配置Octopress

Octopress的作者已经尽量让配置简化了。大多数情况下只需要配置`_config.yml`和`Rakefile`文件即可。其中Rakefile是跟博客部署相关，一般情况下并不需要修改这个文件，除非使用了rsync。
config.yml是博客重要的一个配置文件，在config.yml文件中有三大配置项：`Main Configs`、`Jekyll & Plugins`和`3rd Party Settings`。

一般，该文件中其中`url`是必须要填写的，这里的url是在github上创建的一个仓库地址，具体请看第四步中创建的地址。另外再修改一下`title`、`subtitle`和`author`，根据需求，在开启一些第三方组件服务。
关于_config.yml文件中的更多内容，请看这里的内容：[Configuring Octopress](http://octopress.org/docs/configuring/)

建议：最好把里面的twitter相关的信息全部删掉，否则由于GFW的原因，将会造成页面load很慢。同理，修改定制文件/source/_includes/custom/head.html 把google的自定义字体去掉。from[唐巧的博文中—配置](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)。

###4.将博客部署到GitHub上

Github的[Page service](https://pages.github.com/)可以免费托管博客，并且还可以自定义域名。
首先需要在GitHub上[创建一个仓库](http://beyondvincent.com/blog/2013/08/03/108-creating-a-github-blog-using-octopress/)，并将仓库名称按照这样的方式进行命名：`username.github.com`或`organization.github.com`。等后面配置完毕之后，我们就可以在浏览器中使用页面地址`http://username.github.com`来访问我们的博客。一般来说，我们希望在将博客的源码放到source分支下，并把生成的内容提交到master分支。

创建好仓库之后，我们需要利用octopress的一个`配置rake任务`来自动配置上面创建的仓库：可以让我们方便的部署GitHub page。在终端输入如下命令：

```
$ rake setup_github_pages
```
上面的命令会做一些事情(详细介绍看下面给出的参考链接)。其中最主要的就是创建一个`_deploy`目录，目录用来存放部署到master分支的内容。期间会要求你输入仓库的url，根据提示，进行输入即可。

完成上面的命令之后，我们就可以生成博客并真正的部署到仓库中了。执行如下命令：

```
$ rake generate
$ rake deploy
```

上面的命令首先生成博客文件，并将生成的博客文件拷贝到`_deploy/`目录下，然后将这些内容添加到git中，并commit和push到仓库的master分支。

现在可以访问`http://username.github.com`了。注意：有时候可能会有延时，要等几分钟才能打开。此处的username必须和你github账号名一致。

至此，我们的博客已经完成基本的部署，不过博客的source需要单独提交，执行如下命令就可以将source提交到仓库的source分支下。

```
$ git add .
$ git commit -m 'Initial source commit'
$ git push origin source
```

如果在部署到仓库之前，需要先预览一下博客，可以在终端输入`rake preview`命令，然后就能在浏览器中进行本地预览访问了：`http://127.0.0.1:4000/`或`http://localhost:4000/`，效果跟仓库中的一样。
参考：[Deploying to Github Pages](http://octopress.org/docs/deploying/)

###5.开始写博客

Octopress为我们提供了一些task来创建博文和页面。博文必须存储在`source/_posts`目录下，并且需要按照Jekyll的命名规范对文章进行命名：`YYYY-MM-DD-post-title.markdown`。文章的名字会被当做url的一部分，而其中的日期用于对博文的区分和排序。

通过Octopress提供的task可以正确的按照命名规范创建一个博文，并且在博文中会附带常用的一些yaml元数据。只需要在终端输入如下命令：

```
$ rake new_post["title"]
```

其中title为博文的文件名，创建出来的文件默认是markdown格式。上面的命令会创建出这样一个文件：`source/_posts/2013-08-03-title.markdown`。打开这个文件，可以看到里面有如下一些内容了(告诉Jekyll博客引擎如何处理博文和页面)：

```
layout: post
title: "title"
date: 2013-08-03 16:36
comments: true
categories: 
```

接着我们就可以在这个文件中写我们的博文啦。完成之后，我们可以预览和部署博文。下面是创建并部署博文的一个完整过程：

```
$ rake new_post["New Post"]
$ rake generate
$ git add .
$ git commit -m "Some comment here." 
$ git push origin source
$ rake deploy
```

参考：[Blogging Basics](http://octopress.org/docs/blogging/)

###6.更多操作

在搭建博客的时候，我们可能会对博客做一些配置，例如添加评论、域名解析、分享等。这些内容我写在另外一篇文章中，会经常更新，请前往观看：[你好！github页面](http://beyondvincent.com/blog/2013/07/27/107-hello-page-of-github/)。

###7.小结

本文介绍了如何利用Octopress搭建一个Github博客。大家在搭建的时候，要是遇到问题，可以回复我。本文转载至：[破船之家](http://beyondvincent.com/2013/08/03/2013-08-03-108-creating-a-github-blog-using-octopress/).

         