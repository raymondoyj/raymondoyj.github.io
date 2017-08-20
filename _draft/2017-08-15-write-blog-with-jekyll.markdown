---
layout:     post
title:      "搭建博客"
subtitle:   "用jekyll搭建博客"
date:       2017-08-15 12:00:00
author:     "Raymond"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - blog
    - jekyll
    - markdown
    - git
---

> 按照惯例，在开始写博客的时候，都会写一篇如何搭建blog的指导教程:smile:。  
> 本篇文章所搭建的博客是用[git](/2017/08/15/git-flow/)管理，用[markdown](/2015/07/31/Markdown-Syntax-CN/)写blog，用jekyll作为本地调式环境。

## 什么是Jekyll

[Jekyll][jekyll]是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll恰好是GitHub Pages的运行引擎，也就是说，你可以使用[GitHub Pages][github pages]的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。除了Github Pages外，还可以选择[码云Pages][码云Pages]来托管博客。

## 选择模板

jekyll

## 安装Jekyll

Jekyll是Ruby的程序，接下在会在ubuntu系统上一步一步地安装Jekyll。

### 安装RVM

[RVM][rvm]是一个命令行工具，它可以让你轻松的安装，管理和使用多个Ruby环境。

第一步，安装mpapis公钥，用于验证安装包，确保其安全。

```shell
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```

第二步，安装RVM稳定版

```shell
\curl https://raw.githubusercontent.com/rvm/rvm/master/binscripts/rvm-installer | bash -s stable
```

// TODO 配置rvm in zsh

### 安装Ruby

[Ruby][Ruby]是一种编程语言。

列出ruby的所有可安装的版本：

```shell
rvm list known
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-p374]
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p320]
[ruby-]1.9.3[-p545]
[ruby-]2.0.0-p353
[ruby-]2.0.0[-p451]
[ruby-]2.1[.1]
[ruby-]2.1-head
ruby-head
...
```

安装Ruby的最新稳定版本（如：2.1.1）：

```shell
rvm install 2.1.1
Checking requirements for opensuse.
Requirements installation successful.
Installing Ruby from source to: /home/mpapis/.rvm/rubies/ruby-2.1.1, this may take a while depending on your cpu(s)...
...
Install of ruby-2.1.1 - #complete
Using /home/mpapis/.rvm/gems/ruby-2.1.1
```


使用新安装的Ruby：

```shell
rvm use 2.1.1
Using /home/mpapis/.rvm/gems/ruby-2.1.1
```

检查是否运行正常：

```shell
ruby -v
ruby 2.1.1p76 (2014-02-24 revision 45161) [x86_64-linux]

which ruby
/home/mpapis/.rvm/rubies/ruby-2.1.1/bin/ruby
```

### 安装RubyGems  

[RubyGems][RubyGems]（简称 gems）是一个用于对 Ruby组件进行打包的 Ruby 打包系统。 它提供一个分发 Ruby 程序和库的标准格式，还提供一个管理程序包安装的工具。

1. [下载RubyGems](https://rubygems.org/pages/download)
1. 解压并cd进去
1. 用`ruby setup.rb`安装RubyGems
1. 检查是否安装成功，`gem -v`
1. 用国内的RubyGems镜像代替官方版本
    ```shell
    $ gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
    $ gem sources -l
    https://gems.ruby-china.org
    # 确保只有 gems.ruby-china.org
    ```


### 安装Bundler

[Bundler][bundler]是用来保证Ruby项目一致性的工具。Bundler在这里不是必须要的。

执行命令安装：

```shell
$ gem install bundler
```

在项目的根目录编写`Gemfile`文件来指定依赖：

```
source 'https://rubygems.org'
gem "jekyll"
gem "jekyll-paginate"
```

使用国内的镜像：

```shell
$ bundle config mirror.https://rubygems.org https://gems.ruby-china.org
```

这样你不用改你的 Gemfile 的 source。  
执行命令安装依赖：
```shell
$ bundle install
```


## 参考链接

1. [https://rvm.io/][rvm]
1.

[jekyll]: http://jekyllrb.com/ "Jekyll"
[码云Pages]: http://git.mydoc.io/?t=154714  "码云Pages"
[github pages]: https://pages.github.com/ "Github Pages"
[rvm]: https://rvm.io/ "RVM"
[Ruby]: https://www.ruby-lang.org/zh_cn/ "Ruby"
[RubyGems]: https://rubygems.org/ "RubyGems"
[bundler]: https://bundler.io/ "bundler"
