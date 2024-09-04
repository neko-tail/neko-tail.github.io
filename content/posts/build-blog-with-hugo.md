---
title: '用 hugo 搭建博客'
date: 2024-07-09T09:01:51+08:00
comments: true
draft: false
categories:
  - 技术
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
    - name: Search
      url: /search/
      weight: 60
    - name: Archives
      url: /archives/
      weight: 20
    - name: Tags
      url: /tags/
      weight: 30
    - name: Categories
      url: /categories/
      weight: 40
    - name: Series
      url: /series/
      weight: 50
```

在文章的头信息中指定对应的分类，如：

```md
---
title: '用 hugo 搭建博客'
date: 2024-07-09T09:01:51+08:00
draft: true
categories:
  - 开发
tags:
  - blog
series:
  - 博客搭建
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
      # 搜索
    - name: Search
      url: /search/
      weight: 60
      # 存档
    - name: Archives
      url: /archives/
      weight: 20
      # 标签
    - name: Tags
      url: /tags/
      weight: 30
      # 分类
    - name: Categories
      url: /categories/
      weight: 40
      # 系列
    - name: Series
      url: /series/
      weight: 50

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

  # 
  # homeInfoParams:
  #     Title: Hi there wave
  #     Content: Can be Info, links, about...

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

## favicon

在`static`目录中添加`favicon.ico`文件来配置网站图表

## 字体

> [JetBrains Mono](https://github.com/JetBrains/JetBrainsMono)
>
> [Noto Sans CJK](https://github.com/notofonts/noto-cjk)

为避免网络问题，不使用 Google Fonts 的方案

### woff2 文件

woff2 文件更适合在网页中使用，JetBrains Mono 有直接提供 woff2 格式的文件，但 Noto Sans CJK 没有，这里选择用 woff2 来转换

事先下载好 NotoSansCJKsc-Regular.otf 文件

安装 woff2 工具

```sh
sudo apt install woff2
```

转换文件

```sh
woff2_compress NotoSansCJKsc-Regular.otf
```

### 自定义字体

将所有 woff2 文件复制到`static/fonts`目录下

编辑`assets/css/extended/fonts.css`文件，添加以下内容

```css
@font-face {
  font-family: "JetBrains Mono";
  src: url("/fonts/JetBrainsMono-Bold.woff2");
  font-weight: 700;
  font-style: normal;
}

@font-face {
  font-family: "JetBrains Mono";
  src: url("/fonts/JetBrainsMono-Regular.woff2");
  font-weight: 400;
  font-style: normal;
}

@font-face {
  font-family: "JetBrains Mono";
  src: url("/fonts/JetBrainsMono-Thin.woff2");
  font-weight: 300;
  font-style: normal;
}

@font-face {
  font-family: "JetBrains Mono";
  src: url("/fonts/JetBrainsMono-BoldItalic.woff2");
  font-weight: 700;
  font-style: italic;
}

@font-face {
  font-family: "JetBrains Mono";
  src: url("/fonts/JetBrainsMono-Italic.woff2");
  font-weight: 400;
  font-style: italic;
}

@font-face {
  font-family: "JetBrains Mono";
  src: url("/fonts/JetBrainsMono-ThinItalic.woff2");
  font-weight: 300;
  font-style: italic;
}

@font-face {
  font-family: "Noto Sans CJK SC";
  src: url("/fonts/NotoSansCJKsc-Bold.woff2") format("woff2");
  font-weight: 700;
  font-style: normal;
}

@font-face {
  font-family: "Noto Sans CJK SC";
  src: url("/fonts/NotoSansCJKsc-Regular.woff2") format("woff2");
  font-weight: 400;
  font-style: normal;
}

@font-face {
  font-family: "Noto Sans CJK";
  src: url("/fonts/NotoSansCJK-Thin.woff2") format("woff2");
  font-weight: 300;
  font-style: normal;
}

body,
code {
  font-family: "JetBrains Mono", "Noto Sans CJK SC", "Noto Sans CJK",
    -apple-system, BlinkMacSystemFont, segoe ui, Roboto, Oxygen, Ubuntu,
    Cantarell, open sans, helvetica neue, sans-serif;
}
```

## 侧边目录

> [Toc on the side](https://github.com/adityatelange/hugo-PaperMod/pull/675)
>
> Verdict: I am not positive to go ahead with this one, sticking to single column layouts.

PaperMod 的作者拒绝增加此功能[原始评论](https://github.com/adityatelange/hugo-PaperMod/pull/675#issuecomment-1416736188)，所以这里要自己实现

参考[此提交](https://github.com/JannikArndt/jannikarndt.github.io/commit/8b99f6cbc61c6b4f0238c592d5315cefe07c0599#diff-a50a6578f9ba7dd3f4331dc134731435d490ce889e33eb2cf32e633b6770bbe8)实现

新增`layouts/partials/toc.html`文件

```html
<div class="toc">
    <details {{if (.Param "TocOpen") }} open{{ end }}>
      <summary accesskey="c" title="(Alt + C)">
        <div class="details">{{ .Title }}</div>
      </summary>
      <div class="inner">
{{- $headers := findRE "<h[1-6].*?>(.|\n])+?</h[1-6]>" .Content -}}
{{- $has_headers := ge (len $headers) 1 -}}
{{- if $has_headers -}}

{{- $largest := 6 -}}
{{- range $headers -}}
{{- $headerLevel := index (findRE "[1-6]" . 1) 0 -}}
{{- $headerLevel := len (seq $headerLevel) -}}
{{- if lt $headerLevel $largest -}}
{{- $largest = $headerLevel -}}
{{- end -}}
{{- end -}}

{{- $firstHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers 0) 1) 0)) -}}

{{- $.Scratch.Set "bareul" slice -}}
<ul>
    {{- range seq (sub $firstHeaderLevel $largest) -}}
    <ul>
        {{- $.Scratch.Add "bareul" (sub (add $largest .) 1) -}}
        {{- end -}}
        {{- range $i, $header := $headers -}}
        {{- $headerLevel := index (findRE "[1-6]" . 1) 0 -}}
        {{- $headerLevel := len (seq $headerLevel) -}}

        {{/* get id="xyz" */}}
        {{- $id := index (findRE "(id=\"(.*?)\")" $header 9) 0 }}

        {{- /* strip id="" to leave xyz, no way to get regex capturing groups in hugo */ -}}
        {{- $cleanedID := replace (replace $id "id=\"" "") "\"" "" }}
        {{- $header := replaceRE "<h[1-6].*?>((.|\n])+?)</h[1-6]>" "$1" $header -}}

        {{- if ne $i 0 -}}
        {{- $prevHeaderLevel := index (findRE "[1-6]" (index $headers (sub $i 1)) 1) 0 -}}
        {{- $prevHeaderLevel := len (seq $prevHeaderLevel) -}}
        {{- if gt $headerLevel $prevHeaderLevel -}}
        {{- range seq $prevHeaderLevel (sub $headerLevel 1) -}}
        <ul>
            {{/* the first should not be recorded */}}
            {{- if ne $prevHeaderLevel . -}}
            {{- $.Scratch.Add "bareul" . -}}
            {{- end -}}
            {{- end -}}
            {{- else -}}
            </li>
            {{- if lt $headerLevel $prevHeaderLevel -}}
            {{- range seq (sub $prevHeaderLevel 1) -1 $headerLevel -}}
            {{- if in ($.Scratch.Get "bareul") . -}}
        </ul>
        {{/* manually do pop item */}}
        {{- $tmp := $.Scratch.Get "bareul" -}}
        {{- $.Scratch.Delete "bareul" -}}
        {{- $.Scratch.Set "bareul" slice}}
        {{- range seq (sub (len $tmp) 1) -}}
        {{- $.Scratch.Add "bareul" (index $tmp (sub . 1)) -}}
        {{- end -}}
        {{- else -}}
    </ul>
    </li>
    {{- end -}}
    {{- end -}}
    {{- end -}}
    {{- end -}}
    <li>
        <a href="#{{- $cleanedID  -}}" aria-label="{{- $header | plainify -}}">{{- $header | safeHTML -}}</a>
        {{- else -}}
    <li>
        <a href="#{{- $cleanedID -}}" aria-label="{{- $header | plainify -}}">{{- $header | safeHTML -}}</a>
        {{- end -}}
        {{- end -}}
        <!-- {{- $firstHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers 0) 1) 0)) -}} -->
        {{- $firstHeaderLevel := $largest }}
        {{- $lastHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers (sub (len $headers) 1)) 1) 0)) -}}
    </li>
    {{- range seq (sub $lastHeaderLevel $firstHeaderLevel) -}}
    {{- if in ($.Scratch.Get "bareul") (add . $firstHeaderLevel) -}}
</ul>
{{- else -}}
</ul>
</li>
{{- end -}}
{{- end -}}
</ul>
{{- end -}}
      </div>
    </details>
  </div>
```

新增`assets/css/extended/side_toc.css`文件，样式我做了些修改

```css
.toc {
  padding: 14px;
  font-size: 16px;
}

@media (min-width: 1280px) {
  .toc {
    /* 去掉背景和边框 */
    border: none;
    background: none !important;

    /* 固定在右侧 */
    position: sticky;
    float: right;
    top: 90px;

    /* 左侧偏移 */
    --toc-left: calc(100vw / 40);
    /* 最小宽度 */
    --default-toc-width: calc(
      (1280px - var(--main-width)) / 2 - var(--toc-left) - 14px
    );
    /* 目录宽度 */
    --toc-width: max(
      calc(
        ((100vw - var(--main-width) - 2 * var(--gap)) / 2 - 2 * var(--toc-left)) *
          1
      ),
      var(--default-toc-width)
    );
    width: var(--toc-width);
    /* 向右偏移 */
    margin-right: calc(-1 * var(--toc-width) - var(--toc-left));

    padding: 14px;
    font-size: 16px;
  }

  .toc .inner {
    padding: 0;
  }

  .toc details summary {
    margin-inline-start: 0;
    margin-bottom: 10px;
  }
}

summary {
  cursor: pointer !important;
}
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

## 评论

使用 [giscus](https://github.com/giscus/giscus) 来实现博客评论功能

giscus 使用 GitHub 的 Discussions 来实现评论，评论者需要登录 GitHub 账号之后才能使用，不过现在有看博客习惯的人，大多数也都有用 GitHub 就是了

而且 giscus 的评论系统用起来和 GitHub 的体验一致，也省去了学习成本

在博客中添加 giscus 的操作非常简单

首先，我们需要一个公开的 GitHub 仓库，如果有执行过上面 [GitHub Pages](#github-pages) 章节的操作，那么可以直接用博客自身的仓库

安装 [giscus app](https://github.com/apps/giscus)，`for these repositories`一项最好选择`Only select repositories`，让 giscus app 只能操作你选择的仓库

然后来到仓库的`Settings`页面，勾选`General > Features`中的`Discussions`，为仓库开启评论功能

完成以上三个步骤之后，来到 [giscus.app](https://giscus.app/zh-CN)，在`配置 > 仓库`下填入仓库地址，然后根据自己的需求，修改其他配置项，复制下方自动生成的`<script>`标签，如：

```html
<script src="https://giscus.app/client.js"
        data-repo="user/repo"
        data-repo-id="xxx"
        data-category="Announcements"
        data-category-id="xxx"
        data-mapping="pathname"
        data-strict="1"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="top"
        data-theme="noborder_dark"
        data-lang="zh-CN"
        data-loading="lazy"
        crossorigin="anonymous"
        async>
</script>
```

打开你的博客项目，创建`layouts/partials/comments.html`文件，将`<script>`标签粘贴进去即可

评论默认是关闭的，可以在文章头信息中加入`comments: true`（yaml 格式）来开启对应文章的评论，或者你也可以直接加到文章模板中
