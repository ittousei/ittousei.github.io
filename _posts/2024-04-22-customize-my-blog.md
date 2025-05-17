---
title: Chirpy主题的进阶使用
description: 一些基于Chirpy主题的个性化设置
author: MsEspeon
date: 2024-04-22 16:45:00 +0800
categories: [Tutorial, Jekyll]
pin: false
math: true
mermaid: true
---

## 前言

本文介绍Chirpy主题的几个进阶使用案例，包括自定义网页布局、优化文本搜索、配置评论系统等。其中一些案例会涉及到底层代码的修改，要求读者具备一定的编程基础。

关于Chirpy主题的安装、配置、基础使用等内容，请阅读[Jekyll博客搭建教程（下篇）](/posts/build-my-blog-2/)。

## 自定义网页布局

### Favicon设置

Chirpy主题的favicon存放在目录`assets/img/favicons/`下，用户可以使用[Favicon Generator](https://realfavicongenerator.net/)生成favicon文件，再替换掉目录中的预设favicon即可。

### 语言和字体

博客的默认语言为英语，可以设置`_config.yml`中的`lang`变量来修改，变量取值详见[ISO语言代码](http://www.lingoes.net/en/translator/langcode.htm)（如英语的`lang`取值为`en`，中文的`lang`取值为`zh-CN`）。

每种语言对应一套默认字体，如英语默认的标题字体为Lato，段落字体为Source Sans Pro。如果要自定义，需要修改样式文件`_sass/addon.variables.scss`，参考这篇[说明](https://github.com/cotes2020/jekyll-theme-chirpy/pull/986)。

```scss
/* fonts */
$font-family-base: 'Source Sans Pro', 'Microsoft Yahei', 'PingFang SC', sans-serif !default;
$font-family-heading: Lato, 'Microsoft Yahei', 'PingFang SC', sans-serif !default;
```

至于为何会特意提到自定义字体，因为预设的中文字体太难看了，便改用了PingFang SC作为默认项。

### 侧边菜单栏排序

侧边菜单栏选项的默认排序为：

- HOME
- CATEGORIES
- TAGS
- ARCHIVES
- ABOUT

如果要更改顺序，则需前往`_tabs`目录修改相应文件。以archives选项为例，我们需要修改`_tabs/archives.md`：

```md
---
layout: archives
icon: fas fa-archive
order: 3
---
```

其中，`layout`项表示前往`_layouts`目录，读取`_layouts/archives.html`作为网页模版；`icon`项表示使用`fa-archive`作为icon图标，更多图标详见[Font Awesome 4](https://fontawesome.com/v4/icons/)；`order`项指定了选项在菜单栏中的顺序。

例如，我想把archives选项放在菜单栏的第一位，只需将`order`项设置为`1`。

### 创建和删除子页面

例如我希望在个人网站上创建一个留言板，放在网站子页`/treehole/`下。我们可以向`_tabs`目录添加模版文件`_tabs/treehole.md`来实现这一需求。

```md
---
layout: treehole
icon: fas fa-tree
order: 2
---
```

网站会自动生成子页面`/treehole/`，并读取`_layouts/treehole.html`作为该页面的模版，我们可以在模版文件`treehole.html`中写入自己想要的模块，给网页添加新的功能。以下是html模版的一个简单示例（没有添加任何模块，只是输出‘Hello, world!’）：

```html
---
layout: page
# The layout for Treehole page 
---

<p>Hello, world!</p>
```

同样如果想要删除某个子页面，比如标签页`/tags/`（因为我讨厌打标签），只需删除对应的模版文件`_tabs/tags.md`。

## 优化文本搜索

本节旨在讨论博客文本搜索功能的实现思路，并针对原搜索算法的不足提出一些改进优化。如果想在博客中更新文本搜索功能，可以[fork](https://github.com/ittousei/ittousei.github.io/fork)我的博客仓库。

博客的文本搜索可分解为三个步骤：

- 读取用户输入的关键词
- 遍历博客的所有文章检索关键词
- 输出包含关键词的文章

Chirpy主题在博客中内置了文本搜索功能，使用的搜索算法是Christian Fei的[Simple Jekyll Search](https://github.com/christian-fei/Simple-Jekyll-Search)。该算法的实现思路是创建一个文本数据集`search.json`，将每篇文章作为一级对象，文章的`title`、`author`、`content`等信息作为二级对象，写入json文件中。数据模版如下：

```json
{
  {
    "title": "post.title",
    "url": "post.url",
    "date": "post.date",
    "categories": "post.categories",
    "content": "post.content"
  }
}
```

这样一来，要在博客的文章中检索关键词，不用到每个文章的html页面一一查找，只需遍历一遍json文件即可。虽然实现思路很好，但成品代码却一言难尽，而且原项目在2022年3月就停止维护了，也没法向作者提修改建议。

### 精准搜索

首当其冲的问题是Simple Jekyll Search居然不支持精准搜索，考虑下面这个例子。我想搜索所有包含短语`let it go`的文章，这个句子会出现在搜索结果中：

<center>
<mark style="background-color: rgba(255, 255, 153, 0.5);">Let</mark>'s take a photo here. I'll print <mark style="background-color: rgba(255, 255, 153, 0.5);">it</mark> on my shirt and <mark style="background-color: rgba(255, 255, 153, 0.5);">go</mark> to school.
</center>

这显然是不合理的，我希望三个单词作为一个整体在句中出现，而不是去分别检索三个单词：

<center>
<mark style="background-color: rgba(255, 255, 153, 0.5);">Let it go</mark>, since tomorrow is another day.
</center>

查阅相关讨论，我发现文本的精准匹配这个问题，实际上已经被作者修复了：

- [Simple-Jekyll-Search/issues/182](https://github.com/christian-fei/Simple-Jekyll-Search/issues/182)
- [Simple-Jekyll-Search/pull/166](https://github.com/christian-fei/Simple-Jekyll-Search/pull/166)

既然问题已经解决了，为什么搜索结果还是那么奇怪呢？仔细阅读[项目代码](https://cdn.jsdelivr.net/npm/simple-jekyll-search@1.10.0/)，会发现非常坑的一点，作者虽然在源目录`/src/`更新了代码，却没有在终目录`/dest/`重新编译项目。

- Q：bug修了吗？ \\
  A：修了 \\
  Q：修好了吗？ \\
  A：如修

因此，我们需要重新编写搜索算法，并替换掉原先失效的算法库。具体步骤如下：

- 在`assets/lib/simple-jekyll-search/simple-jekyll-search.js`中重写精确匹配的搜索算法；

```js
function LiteralSearchStrategy () {
  this.matches = function (str, crit) {
    if (!str) return false
    str = str.toLowerCase()
    crit = crit.toLowerCase()

    return str.indexOf(crit) >= 0
 }
}
```

- 使用[在线转换](https://www.toptal.com/developers/javascript-minifier)将.js文件转成.min.js格式，生成的新文件保存至`assets/lib/simple-jekyll-search/simple-jekyll-search.min.js`；
- 在`_data/orginal/cors.yml`中修改了搜索算法的引用地址。

```yaml
search:
# js: https://cdn.jsdelivr.net/npm/simple-jekyll-search@1.10.0/dest/simple-jekyll-search.min.js
  js: /assets/lib/simple-jekyll-search/simple-jekyll-search.min.js
```

此外，我对文本数据`assets/js/data/search.json`做了一些格式上的优化，使搜索结果更为精确，在此不做详细说明。

### 搜索结果展示

另一个问题是Simple Jekyll Search虽然能告诉你哪些文章包含关键词，但它不会显示关键词具体出现在文章的哪个位置。这是使用原Chirpy主题进行关键词检索的一个示例：

![](/assets/img/custom-my-blog/original-search.jpeg)

可以看到，搜索引擎只是简单列出了包含关键词`Chirpy`的文章标题和简介，而实际上我更想知道关键词出现在文章中的哪个位置，以及提及该关键词的前后文。这篇[博客](https://reesdraminski.com/garden/search-with-simplejekyllsearch/)提出了一种可行的优化思路：Simple Jekyll Search在完成全局搜索后，会输出所有包含关键词的文章。此时对这些文章再重复做一次关键词检索，记录下关键词在一篇文章中首次出现的位置，输出该位置向前数的50个字符（上文）和向后数的50个字符（下文）。

我在`_includes/search_loader.html`中实现了这个思路，以下是部分代码片段：

```js
  var input = document.getElementById("search-input").value.toLowerCase();
  let firstOccurrence = value.toLowerCase().indexOf(input);
  if (firstOccurrence != -1) {
      let start = firstOccurrence - 50;
      if (start < 0) {
      start = 0;
      }
      while (start > 0 && value[start-1] != " " && value[start-1] != "，" && value[start-1] != "。") {
      start--;
      }
      return `${value.substring(start, firstOccurrence)}<mark style="background-color: rgba(255, 255, 153, 0.5);">${value.substring(firstOccurrence, firstOccurrence+input.length)}</mark>${value.substring(firstOccurrence+input.length, start+250)}`;
  }
```

优化后的搜索结果如图，改进后的算法能标注出关键词所在位置，并给出提及关键词的上下文。

![](/assets/img/custom-my-blog/updated-search.jpeg)

## 配置评论系统

Chirpy主题支持为文章添加评论系统，在`config.yml`的`comments`一项中设置，可选择的评论系统有

- disqus
- utterances
- giscus

这里以giscus为例，说明如何在博客中开启评论。先在`config.yml`中声明开启gicus。

```yaml
comments:
  # Global switch for the post comment system. Keeping it empty means disabled.
  provider: giscus # [disqus | utterances | giscus]
```

再前往[giscus app](https://giscus.app/)，按网页要求进行操作：

- 将博客repository的访问权限设置为[public](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility#making-a-repository-public)；
- 下载[giscus](https://github.com/apps/giscus)；
- 访问博客所在的repository，在<kbd>Settings</kbd>><kbd>General</kbd>><kbd>Features</kbd>一栏中勾选上[Discussions](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/enabling-or-disabling-github-discussions-for-a-repository)。

在giscus app中填入博客repository的地址，获取giscus的配置参数，填入`config.yml`中的对应项。

```yaml
  # Giscus options › https://giscus.app
  giscus:
    repo: [ENTER REPO HERE] # <gh-username>/<repo>
    repo_id: [ENTER REPO ID HERE]
    category: [ENTER CATEGORY NAME HERE]
    category_id: [ENTER CATEGORY ID HERE]
    mapping: "title" # optional, default to 'pathname'
    strict: # optional, default to '0'
    input_position: "top" # optional, default to 'bottom'
    lang: # optional, default to the value of `site.lang`
    reactions_enabled: # optional, default to the value of `1`
```

最后，重新生成静态网页并推送到GitHub仓库，就能在文章的评论区发表评论了。

```zsh
bundle exec jekyll s
```

如果需要对giscus做更详细的配置，可在博客项目的根目录创建配置文件`giscus.json`，参考这篇[giscus Advanced Usage](https://github.com/giscus/giscus/blob/main/ADVANCED-USAGE.md)进行配置。

## 结语

关于博客搭建的记录就此告一段落，希望我把教程写清楚了。最后感谢[Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)为本站提供Jekyll主题，提供一些扩展阅读：

- [Chirpy Wiki](https://github.com/cotes2020/jekyll-theme-chirpy/wiki)
- [使用Chirpy模版进行Markdown写作](https://chirpy.cotes.page/posts/write-a-new-post/)
- [Markdown渲染效果演示](https://chirpy.cotes.page/posts/text-and-typography/)