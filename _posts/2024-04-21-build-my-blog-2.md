---
title: Jekyll博客搭建教程（下篇）
description: Chirpy主题的使用说明：使用Chirpy设置博客布局，在GitHub Page上部署博客
author: MsEspeon
date: 2024-04-21 20:17:00 +0800
categories: [Tutorial, Jekyll]
pin: false
math: true
mermaid: true
---

## 前言

[Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)是一款基于Jekyll架构开发的主题模版，主要面向技术类博客的写作，这是该主题的一个[Demo](https://chirpy.cotes.page/)。本站的主题也是由Chirpy主题魔改而来。

本文以Chirpy主题为例，介绍如何在Jekyll博客中应用主题模版，并将博客网站部署到GitHub Page上。其他主题的应用方法可能略有不同，但总体思路是一致的。

## 下载主题

Chirpy主题有两种下载方法：

- Chirpy Starter
- Git

两种方法均会使用到[Git](https://git-scm.com/)，请事先配置好Git的使用环境。

### Chirpy Starter

前往官方的模版页面[Chirpy Start](https://github.com/cotes2020/chirpy-starter)，点击按钮<kbd>Use this template</kbd> > <kbd>Create a new repository</kbd>，将新的repository命名为`USERNAME.github.io`，其中`USERNAME`为GitHub用户名。

再将项目下载到本地，并安装依赖库。（记得将`USERNAME`替换为GitHub用户名）

```zsh
git clone https://github.com/USERNAME/USERNAME.github.io
bundle
```

这个方法适合只想专心写文章不想接触底层代码的用户。用户无需关注原项目中那一大坨html模版和js脚本，只需将文章添加至`_post`目录中即可实现博客的更新。当然不能修改代码也意味着，如果你想在原模版的基础上做些个性化修改，或是添加新的功能，用这个方法是无法实现的。

### Git

作为一个事多的人，只是套用模版而不做任何修改是无法满足我的需求的，因此我采用了第二种方法：用Git将整个项目下载至本地。这样便能直接修改源代码，实现我想要的功能。

```zsh
git clone https://github.com/cotes2020/jekyll-theme-chirpy
```

由于后续操作会生成JavaScript文件，需事先安装[Node.js](https://nodejs.org/en)。再前往项目所在目录，运行初始化程序。

```zsh
bash tools/init
```

完成初始化后，还需安装依赖库。

```zsh
bundle
```

## 配置主题

### 主要配置

博客的配置信息存放在根目录的`./_config.yml`，常用的变量有：

- lang - 语言
- timezone - 时区
- title - 网站标题
- tagline - 网站副标题
- description - 网站描述
- url - 部署博客的地址，如`https://USERNAME.github.io`
- avatar - 作者头像，如`https://chirpy-img.netlify.app/commons/avatar.jpg`

### 可选配置

在子目录`_data`下还存放有一些可选填的配置文件，主要用于设置网页的外观，可根据需求修改。以下是对这些配置文件的简要说明：

博客的语言配置存放在`_data/locate/`，其中英语的配置文件为`_data/locate/en.yml`。如下面这段配置定义了英文语言下侧边菜单栏的选项名称。

```yaml
tabs:
  # format: <filename_without_extension>: <value>
  home: Home
  categories: Categories
  tags: Tags
  treehole: Treehole
  archives: Archives
  about: About
```

文章的作者信息存放在`_data/authors.yml`，可填写多个作者。

```yaml
MsEspeon:
  name: MsEspeon
  url: https://github.com/ittousei/
```

侧边菜单栏的社交账号信息存放在`_data/contact.yml`。比如我不想显示Twitter，只需在配置文件中注释掉相应内容。

```yaml
# - type: twitter
#   icon: "fa-brands fa-x-twitter"
```

文章的分享信息存放在`_data/share.yml`。比如我想分享文章到Twitter，可在配置文件中添加以下内容。

```yaml
platforms:
  - type: Twitter
    icon: "fa-brands fa-square-x-twitter"
    link: "https://twitter.com/intent/tweet?text=TITLE&url=URL"
```

## 本地测试

在服务器上部署博客前，可以先在本地[http://localhost:4000](http://localhost:4000)预览网页的效果。

```zsh
bundle exec jekyll s
```

注意：以上命令包含两个功能，一是根据项目文件生成静态网页文件，二是将静态网页部署至本地服务器。因此，在将博客部署到远程服务器之前，即使没有本地测试的需求，仍需运行一遍上述命令，以生成必要的静态网页文件。

## 部署至GitHub Page

在实现本地运行后，我们将博客部署到[GitHub Page](https://pages.github.com/)上。在部署之前，我们需要做好以下准备工作：

- 在GitHub上创建博客专用的repository，并命名为`USERNAME.github.io`，`USERNAME`为GitHub用户名。
- 将本地的Chirpy-Jekyll项目关联至GitHub的仓库`USERNAME.github.io`。这一步我使用的是[GitHub Desktop](https://desktop.github.com/)，一款基于Git的图形化界面，相对于Git更为简单易用，当然直接使用Git也能实现关联操作。
- 在本地项目的根目录运行：

```zsh
bundle lock --add-platform x86_64-linux
```

然后，我们到GitHub上配置Page服务：

- 访问博客所在仓库`USERNAME.github.io`，点击<kbd>Settings</kbd>，在左侧菜单栏中选中<kbd>Pages</kbd>，再到`Build and deployment`的`Source`一栏选中<kbd>GitHub Actions</kbd>选项。

![Build source](/assets/img/build-my-blog/pages-source-light.png){: .light .border .normal w='375' h='140' }
![Build source](/assets/img/build-my-blog/pages-source-dark.png){: .dark .normal w='375' h='140' }

- 在<kbd>GitHub Actions</kbd>选项的下方将出现配置文件`GitHub Pages Jekyll`，点击<kbd>Configure</kbd>按钮，会显示配置文件的具体内容，再点击文件右上角的<kbd>Commit changes...</kbd>，即可在仓库中生成配置文件。

- 最后，我们将配置文件同步到本地。完成上述步骤后，只要我们将代码push到GitHub的仓库中，GitHub Actions便会自动生成静态网页并部署到GitHub Page上。

如果需要在个人服务器上部署博客，可参考[Chirpy官方文档](https://chirpy.cotes.page/posts/getting-started/#manually-build-and-deploy)，在本教程中不做额外说明。

## 更新博客

如果要在博客上发布新的文章，只需遵循以下步骤：

- 在`_posts`目录中添加新的Markdown文件。
- 执行命令生成静态网页：

```zsh
bundle exec jekyll s
```

- 推送至GitHub仓库，更新后的网页会自动部署到GitHub Page上。

## 结语

通过简单的三步，我们实现了个人博客的搭建：

1. 安装Jekyll框架生成博客的静态网页
2. 应用Chirpy主题设置博客的布局
3. 使用GitHub Page部署个人网站

回顾下来，整个博客搭建的过程还是相当水到渠成的，没有遇上重大bug。当然，之所以过程这么顺利，是因为去年我已经尝试过使用[Hexo](https://hexo.io/)搭建个人网站，不过当时的我十分头铁，妄图自己写一套网站的前端（此前没有前端经验，主要使用ChatGPT进行前端开发）。

结果也在预料之中，起初我对这个项目颇有兴致，然而随着开发的深入，bug接二连三的出现，我更新代码的频率也逐渐降低，后来我的旧电脑不幸故障无法开机，又换了新的MacBook，忙于配置新的电脑，这个旧电脑上的[前端项目](https://github.com/ittousei/eephi.github.io)于是便不了了之了。

有点偏题了，说回正题。这次博客的搭建历时两天，4.19完成了Chirpy主题的应用和GitHub Page的部署，4.20实现博客布局的个性化、优化搜索算法、以及添加评论系统。本教程主要记录了4.19的工作，4.20主要是一些基于个人审美而做的改动，在本文的后续[Chirpy主题的进阶使用](/posts/customize-my-blog/)中说明。最后，就简单用一句话收尾吧：

<center><i>搭建个人博客很简单，因为我们站在巨人的肩膀上。</i></center>