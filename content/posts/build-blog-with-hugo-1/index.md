---
title: '用 hugo 搭建博客（一）- 搭建与部署'
date: 2024-07-09T09:01:51+08:00
comments: true
draft: false
categories:
  - tech
tags:
  - blog
series:
  - blog
---

用 hugo 来搭建一个博客网站，使用 PaperMod 作为主题，部署到 GitHub Pages

<!--more-->

## hugo

> [Quick start](https://gohugo.io/getting-started/quick-start/)

按照 Quick start 中的步骤安装好 hugo 之后，便可以创建一个 blog 项目，默认的配置文件是`hugo.toml`，创建时使用`--format`可以指定为其他格式，比如`yaml`

```sh
hugo new site blog --format yaml
```

创建好的 blog 项目是没有被 git 管理的，可以在 blog 目录中`git init`一下，然后就可以安装主题了

## PaperMod

> [wiki](https://github.com/adityatelange/hugo-PaperMod/wiki)

使用 wiki 中 Installation 章节推荐的方法来安装主题

```sh
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```

然后在 hugo 的配置文件（这里是`hugo.yaml`）中添加配置

```yaml
theme: ["PaperMod"]
```

现在就可以用`hugo server`命令启动服务，访问 [http://localhost:1313/](http://localhost:1313/) 来查看网站了

## 文章

使用下面的命令可以新增一篇文章，文章默认会被放在`content`目录下，如果是类似于`posts/test.md`的写法，就是在`content/posts`目录，这样可以把文章和之后要添加的`archives.md`、`search.md`等页面分开，好管理一些

```sh
hugo new content posts/test.md
```

默认情况下，新增文章的头信息(`front matter`)会是`tomal`语法，可以修改`archetypes/default.md`来修改模板，比如改成`yaml`格式

```md
---
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
date: {{ .Date }}
draft: true
---
```

## 配置

### 存档和搜索页面

这两个页面需要在`content`目录中添加对应的 md 文件，如：

archives.md

```md
---
title: "Archive"
layout: "archives"
url: "/archives/"
summary: archives
---
```

search.md

```md
---
title: "Search"
layout: "search"
summary: "search"
placeholder: "placeholder text in search input box"
---
```

然后在`hugo.yaml`中配置菜单，类似于下面这样，url 要和 md 文件名一致

```yaml
menu:
  main:
    - name: Archives
      url: /archives/
      weight: 10
    - name: Search
      url: /search/
      weight: 20
```

### 分类

修改`hugo.yaml`，在`taxonomies` 中指定默认的标签`tag`、分类`category`和自定义的`series`三种分类方法，详细内容可以参考[Taxonomies](https://gohugo.io/content-management/taxonomies/)

```yaml
# 分类
taxonomies:
  tag: tags
  category: categories
  series: series
```

之后添加三种分类方法对应的菜单，用`weight`来指定菜单的顺序

```yaml
menu:
  main:
    - name: Archives
      url: /archives/
      weight: 20
    - name: Categories
      url: /categories/
      weight: 30
    - name: Tags
      url: /tags/
      weight: 40
    - name: Series
      url: /series/
      weight: 50
    - name: Search
      url: /search/
      weight: 60
```

在文章的头信息中指定对应的分类，如：

```md
---
title: '用 hugo 搭建博客'
date: 2024-07-09T09:01:51+08:00
draft: true
categories:
  - tech
tags:
  - blog
series:
  - blog
---
```

### 更多配置

```yaml
# url
baseURL: https://thmaster.net
# 语言
languageCode: zh
# 标题
title: th-master
# 主题
theme: "PaperMod"

# 一页文章数
paginate: 5

# 分类方法
taxonomies:
  # 标签
  tag: tags
  # 分类
  category: categories
  # 系列
  series: series

# 菜单
menu:
  main:
    - name: 归档
      url: /archives/
      weight: 20
    - name: 技术
      url: /categories/tech/
      weight: 21
    - name: 生活
      url: /categories/life/
      weight: 22
    - name: fun
      url: /categories/fun/
      weight: 23
    - name: 分类
      url: /categories/
      weight: 30
    - name: 标签
      url: /tags/
      weight: 40
    - name: 系列
      url: /series/
      weight: 50
    - name: 搜索
      url: /search/
      weight: 60

# 启用 emoji，可以使用 :cry: 的语法显示 emoji
# 详情见 https://gohugo.io/getting-started/configuration/#enableemoji
enableEmoji: true
# 内联短码，详情见 https://gohugo.io/templates/shortcode/#inline-shortcodes
enableInlineShortcodes: true
# 生成 robots.txt
enableRobotsTXT: true
buildDrafts: false
buildFuture: false
buildExpired: false
# 代码高亮中使用 css class 来指定样式
pygmentsUseClasses: true
# 自动检测内容中的中文/日文/韩文，确保词数统计等功能正常
# 详情见https://gohugo.io/getting-started/configuration/#hascjklanguage
hasCJKLanguage: true

outputs:
  home:
    - HTML
    - RSS
    - JSON # necessary for search

params:
  env: production
  description: "th-master's blog"
  # 作者
  author: th-master
  # 日期格式化
  DateFormat: "2006-01-02"
  # 默认主题
  defaultTheme: dark
  # 阅读用时
  ShowReadingTime: true
  # 上一篇/下一篇 导航链接
  ShowPostNavLinks: true
  # 面包屑导航
  ShowBreadCrumbs: true
  # 代码块复制按钮
  ShowCodeCopyButtons: true
  # 在存档页面中显示所有页面
  ShowAllPagesInArchive: true
  # 显示分页
  ShowPageNums: true
  # 显示字数统计
  ShowWordCount: true
  # 显示目录
  ShowToc: true
  # 默认展开目录
  TocOpen: true

  # 主页
  homeInfoParams:
    Title: "Hi :innocent:"
    Content: Welcome to my blog

  # 社交链接
  socialIcons:
    - name: github
      title: "Github"
      url: "https://github.com/"

  assets:
    # 不加载代码高亮库
    disableHLJS: true

markup:
  goldmark:
    renderer:
      # 允许在 md 中使用不安全的 HTML 标签
      unsafe: true
  # 代码高亮
  highlight:
    # 使用 css class 来指定代码高亮的样式
    noClasses: false
    # 启用代码高亮
    codeFences: true
    # 没有指定语言时猜测代码的语言并应用对应的高亮
    guessSyntax: true
    # 代码块显示行号
    lineNos: true
    # 代码高亮的样式主题
    style: monokai

```

## GitHub Pages

### 部署

> [Host on GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
>
> [关于 GitHub Pages](https://docs.github.com/zh/pages/getting-started-with-github-pages/about-github-pages#types-of-github-pages-sites)

直接参考上面的两篇文档，创建一个名为`<user>.github.io`的仓库，将本地的 blog 项目推送上去

然后在 GitHub 仓库页面选择 **Settings > Pages**，将 **Build and deployment > Source** 选项从`Deploy from a branch`修改为`GitHub Actions`

之后在本地仓库中创建`.github/workflows/hugo.yaml`文件，将[Host on GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)中提供的内容粘贴进去，视情况修改其中的分支名称，比如从`main`改为`master`

提交此修改并推送至 GitHub

之后在每次推送时，都会产生一个 workflow 进行构建和部署，在 **Actions** 页面可以查看，待 workflow 成功结束之后就可以用`<user>.github.io`来访问了

### 自定义域名

文档中的建议是给顶级域名和 www 二级域名都配置上 DNS，然后自定义域名设置为 www 二级域名，这样访问顶级域名时会自动跳转到 www 二级域名

参考文档建议，修改自己域名的 DNS，给根域名和 www 添加 CNAME 类型的记录，内容为`<user>.github.io`

然后回到 GitHub 仓库的 **Settings > Pages** 设置，在 **Custom domain** 下填入域名如`www.xxx.com`，点击 **Save** 保存，等待 DNS 检查完毕之后，就可以用自定义域名来访问了
