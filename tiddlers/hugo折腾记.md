## 0. 换行
	时间：2024.2.10/2.14
### 问题：
源码模式：
```powershell
Hello
World
```
渲染后（并未完成换行）：
```powershell
Hello World
```
### 原因
根据 [Markdown Syntax](https://daringfireball.net/projects/markdown/syntax#p) 文档，当从源码输出为 HTML 时，换行符（软换行）不会被转换为 `<br />` （硬换行），如果要插入硬换行，你需要在行尾输入两个（或更多）空格。

根据 [GitHub Flavored Markdown Spec](https://github.github.com/gfm/#hard-line-breaks) 文档，要插入硬换行，你还可以在行尾输入 \ 或者直接输入 `<br />` 。

### 解决  
Hugo 的硬换行默认关闭。
https://gohugo.io/getting-started/configuration-markup/#configure-markup

在 `hugo.toml` 中添加
```powershell
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      hardWraps = true
```

或者：

使用 typora 编辑，换行时会自动空出一行。

使用 obsidian 编辑，使用插件 easy typing，插件的配置中，有一个“一次回车产生两个换行符”的选项，打开即与 typora 相同。

## 1. 评论
	时间：2024.2.14
主题是 nostyleplease，所用评论系统是 [giscus](https://github.com/giscus/giscus).

具体操作：
```powershell
将 giscus 提供的<script>标签内容做对应配置后插入到 layouts/posts/single.html 的</article> 这一行后面
```
## 2. JsDelivr+picgo，加速 github 图床
默认是 github 图床，通过 picgo 将图片上传到 github.

鉴于太慢，使用 jsDelivr 加速图片访问地址。

在 picgo 配置中，修改自定义域名即可。

- 设定自定义域名：此处直接设置 jsDelivr 加速的访问地址，例如本人是：`https://cdn.jsdelivr.net/gh/luo029/blogimage@main`

- `gh` 表示来自 Github 的仓库
- `/luo029/blogimage` 仓库的具体位置
- `main` 仓库的分支
  

## 3. Rss 没有换行符，无法正常显示博文

### 前言

默认 rss 模板中，博文没有换行符 `<p>`。

例子：


```xml
<description> 早晚课，早晚功，咕咕。 折腾 tw，无甚事。 话说，天台今天过元宵节？ 应该吃汤圆吧。 午斋。 就酱~ 甲辰正月十四 嗣檙 于桐柏宫</description>
```

此时如果通过 rss 订阅的话，就会发现正文无法换行，堆叠在一块。

修改后：


```xml
<description><p>早晚课，早晚功，咕咕。</p> <p>折腾 tw，无甚事。</p> <p>话说，天台今天过元宵节？应该吃汤圆吧。</p> <p><img src="https://cdn.jsdelivr.net/gh/luo029/blogimage@main/24%200223%202146%2056.png" alt="image.png"></p> <p>午斋。</p> <p>就酱~</p> <p>甲辰正月十四</p> <p>嗣檙 于桐柏宫</p></description>
```

有了换行符，同时博文内容正常显示（博文里有外部链接图片的话，在默认模板里似乎无法显示。）


博客 Rss 查看：`域名/index.xml`

### 解决方案

在 `themes\nostyleplease (主题名）\layouts\_default` 中添加 `rss.xml` :

```xml
{{- /* Generate RSS v2 with full page content. */ -}}
{{- /* Upstream Hugo bug - RSS dates can be in future: https://github.com/gohugoio/hugo/issues/3918 */ -}}
{{- $page_context := cond .IsHome site . -}}
{{- $pages := $page_context.RegularPages -}}
{{- $limit := site.Config.Services.RSS.Limit -}}
{{- if ge $limit 1 -}}
  {{- $pages = $pages | first $limit -}}
{{- end -}}
{{- printf "<?xml version=\"1.0\" encoding=\"utf-8\" standalone=\"yes\" ?>" | safeHTML }}
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ if ne .Title site.Title }}{{ with .Title }}{{.}} | {{ end }}{{end}}{{ site.Title }}</title>
    <link>{{ .Permalink }}</link>
    {{- with .OutputFormats.Get "RSS" }}
      {{ printf "<atom:link href=%q rel=\"self\" type=%q />" .Permalink .MediaType | safeHTML }}
    {{ end -}}
    <description>{{ .Title | default site.Title }}</description>
    <generator>Source Themes Academic (https://sourcethemes.com/academic/)</generator>
    {{- with site.LanguageCode }}<language>{{.}}</language>{{end -}}
    {{- with site.Copyright }}<copyright>{{ replace (replace . "{year}" now.Year) "&copy;" "©" | plainify }}</copyright>{{end -}}
    {{- if not .Date.IsZero }}<lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end -}}
    {{- if .Scratch.Get "og_image" }}
    <image>
      <url>{{ .Scratch.Get "og_image" }}</url>
      <title>{{ .Title | default site.Title }}</title>
      <link>{{ .Permalink }}</link>
    </image>
    {{end -}}
    {{ range $pages }}
    <item>
      <title>{{ .Title }}</title>
      <link>{{ .Permalink }}</link>
      <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
      <guid>{{ .Permalink }}</guid>
      <description>{{ .Content | html }}</description>
    </item>
    {{ end }}
  </channel>
</rss>
```

## 4. Mailchimp 的插入

不清楚是哪个文件生效了……

分别在 `\themes\nostyleplease\layouts\_default\single.html` 

以及 `\themes\nostyleplease\layouts\index.html`

插入了 mailhcimp 的订阅代码。
## 5.添加 favicon

把 Favicon 图标直接放到 `themes/nostyleplease/static` 目录里

（我同时还放在了`主文件夹/static` 文件里，不过我觉得应该首先使用主题文件夹里的。）

在 `theme/nostyleplease（主题）/layouts/partials/head. html`  文件查找到下面的一行或是直接添加一行：

```html
<link rel="shortcut icon" href="xxx" />
```

然后修改为

```html
<link rel="shortcut icon" href="favicon.ico" />
```

## 6. 修改 hugo 的默认 url

以前的博文 url：`https://sicheng.taoooist.org/posts/diary/正月初一`

由于修真日报只放日记，于是删除了 `diary` 文件夹，不再划分文件。

博文 url 变成了：`https://sicheng.taoooist.org/posts/正月初一`

通过修改 `hugo.toml`:


```toml
[permalinks]
  posts = "/:slug/"
  [permalinks.page]
    '/' = '/:slug/'
  [permalinks.term]
    tags = '/:slug/'
```

参照了[小氯的文章](https://yoghurtlee.com/link-modifying-of-twikoo/)和[官方文档](https://gohugo.io/content-management/urls/)，起作用的是 `posts = "/:slug/"`

目前博文 url：`https://sicheng.taoooist.org/正月初一`