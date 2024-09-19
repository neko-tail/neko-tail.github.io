---
title: '用 hugo 搭建博客（三）- 评论'
date: 2024-09-04T16:47:08+08:00
draft: false
categories:
  - tech
tags:
  - blog
series:
  - blog
---

使用 [giscus](https://github.com/giscus/giscus) 来实现博客评论功能

<!--more-->

giscus 使用 GitHub 的 Discussions 来实现评论，评论者需要登录 GitHub 账号之后才能使用，不过现在有看博客习惯的人，大多数也都有用 GitHub 就是了

而且 giscus 的评论系统用起来和 GitHub 的体验一致，也省去了学习成本

在博客中添加 giscus 的操作非常简单

首先，我们需要一个公开的 GitHub 仓库，如果已使用 GitHub Pages，那么可以直接用博客自身的仓库

安装 [giscus app](https://github.com/apps/giscus)，其中`for these repositories`一项最好选择`Only select repositories`，让 giscus app 只能操作你选择的仓库

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
