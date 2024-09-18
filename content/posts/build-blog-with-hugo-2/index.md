---
title: '用 hugo 搭建博客（二）- 修改样式与美化'
date: 2024-07-09T09:01:52+08:00
comments: true
draft: false
categories:
  - tech
tags:
  - blog
series:
  - blog
---

修改主题样式，进行自定义

<!--more-->

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
