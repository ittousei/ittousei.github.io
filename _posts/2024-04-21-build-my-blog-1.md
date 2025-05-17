---
title: Jekyll博客搭建教程（上篇）
description: 关于Jekyll环境的安装与使用
author: MsEspeon
date: 2024-04-21 12:10:00 +0800
categories: [Tutorial, Jekyll]
pin: false
math: true
mermaid: true
---

## 前言

[Jekyll](https://github.com/jekyll/jekyll)是一个轻量级的静态网站生成器，它采用Liquid作为模板语言，支持用户自定义网页的外观布局、使用Markdown创作发布内容。本文介绍如何使用Jekyll搭建个人网站，内容参考自Jekyll的[说明文档](https://jekyllrb.com/docs/)。

## 环境安装

Jekyll要求安装以下环境：

- Ruby
- RubyGems
- Jekyll

我的操作系统是Ventura(macOS 13)，因此主要基于mac的安装流程进行说明，其他系统的安装步骤类似，这里仅给出文档链接不做具体说明。

### 安装Ruby和RubyGems

Ruby的安装详见[Jekyll Installation](https://jekyllrb.com/docs/installation/)。根据需求选择合适的版本，Jekyll要求Ruby版本高于**2.5.0**，本文使用的版本为**3.1.3**。RubyGems捆绑在Ruby的安装包中，无需额外下载。

对于Windows/Linux等系统，请按照Jekyll文档的说明进行安装，并跳过本章节阅读下一章[使用Jekyll搭建博客](#使用jekyll搭建博客)。

在macOS下，可使用`ruby-install`安装Ruby，和使用`chruby`配置环境：

- 安装[Homebrew](https://brew.sh/)

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- 使用Homebrew安装`chruby`和`ruby-install`

```zsh
brew install chruby ruby-install xz
```

- 使用`ruby-install`安装Ruby

```zsh
ruby-install ruby 3.1.3
```

- 使用`chruby`配置环境

```zsh
echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
echo "chruby ruby-3.1.3" >> ~/.zshrc # run 'chruby' to see actual version
```

- 检查安装是否成功

退出并**重启**（注意要重开一个窗口）Terminal终端，运行以下命令：

```zsh
ruby -v
```

运行结果显示`ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e)`（或是更高版本）即安装成功。

### 安装Jekyll

使用Ruby的包管理器RubyGems可直接下载Jekyll。

```zsh
gem install jekyll
```

## 使用Jekyll搭建博客

在上一节中我们安装了前置环境，现在我们创建一个Jekyll项目来生成博客网页。

我们选择[Bundle](https://bundler.io/)来管理Jekyll的项目环境。对于不同的Jekyll项目，可以用Bundle分别搭建独立的依赖环境，而不相互干扰。使用Bundle创建Jekyll项目的流程如下：

- 使用RubyGems下载`bundle`

```zsh
gem install bundler
```

- 在`./myblog`路径下创建Jekyll项目

```zsh
jekyll new myblog
cd myblog
```

- 运行项目生成博客网页

```zsh
bundle exec jekyll serve
```

可以在[http://localhost:4000](http://localhost:4000)本地浏览网页。


## 自定义博客外观

通过创建Jekyll项目，我们生成了一个简易的网站。当然这个网站十分简陋，和一个功能完备的现代博客相去甚远，下一步就是美化网页的外观以及添加更多的功能了。

一个选择是自己写一套前端，可以参考这篇教程[Step by Step Tutorial](https://jekyllrb.com/docs/step-by-step/01-setup/)，其中的工作量无疑是巨大的。另一个选择是使用别人写好的模版，Jekyll拥有成熟的主题开发生态，用户可以从海量的主题模版中挑选出适合自己的那一款。我们可以在他人模版的基础上，根据自己的需求做一些自定义修改。

下面列出一些常用Jekyll主题站点：

- [GitHub.com \#jekyll-theme repos](https://github.com/topics/jekyll-theme)
- [jamstackthemes.dev](https://jamstackthemes.dev/ssg/jekyll/)
- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://jekyllthemes.io/](https://jekyllthemes.io/)
- [jekyll-themes.com](jekyll-themes.com)

关于主题设置的更多内容，参考[Jekyll Themes](https://jekyllrb.com/docs/themes/)。在[下篇](/posts/build-my-blog-2/)中，我将详细介绍本站采用的主题，即Chirpy主题的配置和使用。