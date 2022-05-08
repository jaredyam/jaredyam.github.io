---
title: "使用 Hugo 和 GitHub Pages 搭建并部署一个静态博客网站"
date: 2022-03-06T20:29:59+08:00
Description: "🔎 https://<your-github-id>.github.io ↩️"
tags: ["Hugo", "GitHub Pages"]
draft: false
---

最近打算将我在 [CSDN](https://blog.csdn.net/weixin_43958105) 上发布并持续优化的部分内容同步至 GitHub，这些内容十分需要借助 Git 记录和追溯修改历史，而 GitHub 也可以作为这些内容的另一个获取平台。

将文档直接丢进 GitHub 仓库好像又差点意思。[GitHub Pages](https://pages.github.com/) 为每个 GitHub 账户免费提供一项静态网站托管服务，通过域名 _https://\<your-github-id>.github.io_ 即可访问同名仓库托管的静态网站。借助这项服务，我们可以依托 GitHub 仓库搭建一个静态博客网站，将文档以博客形式开放展阅。

本文将介绍<strong>如何使用 [Hugo](https://gohugo.io/) 搭建静态博客网站、使用和定制 [hugo-PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题以及将网站部署在 [GitHub Pages](https://pages.github.com/)</strong>。

---

## 搭建静态网站

如今搭建静态网站的方法已经非常成熟，我根据[网友讨论](https://v2ex.com/t/785625)和个人喜好选择 [Hugo](https://gohugo.io/) 作为静态网站搭建框架，同样流行的框架还包括 [Hexo](https://hexo.io/zh-cn/)、[Jekyll](https://jekyllrb.com/) 等。

### 安装 Hugo 框架

Hugo 提供多种操作系统的[安装指引](https://gohugo.io/getting-started/installing)。以我的 macOS 系统为例，可以通过包管理器 `brew` 快速完成安装：
```bash
brew install hugo
```

### 创建网站模板

使用 Hugo 可以快速创建一个网站模板：
```bash
brew new site <your-github-id>.github.io -f yml
```
其中 `<your-github-id>.github.io` 对应新建网站的项目文件夹（`<your-github-id>` 需替换为 GitHub ID），选项 `-f` 将网站配置文件设置为 `YAML` 格式，其目的是为与下文主题配置保持一致。为方便起见，下文将网站项目文件夹简称为「网站」。

### 配置基础主题

在流行框架的基础上搭建网站其实并不是一个技术活，千差万别的网站体验则主要体现在网站主题的设计上。Hugo 提供丰富多样的[主题模板](https://themes.gohugo.io/)，我根据[网友讨论](https://v2ex.com/t/828677)选择 [adityatelange/hugo-PaperMod](https://github.com/adityatelange/hugo-PaperMod) 作为基础主题。[PaperMod](https://github.com/adityatelange/hugo-PaperMod) 页面简洁，支持暗黑模式及众多[实用特性](https://github.com/adityatelange/hugo-PaperMod/wiki/Features#default-theme-lightdarkauto)。

在网站根目录执行以下命令下载主题并初始化为 Git 子模块：
```bash
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
git init
git submodule--helper add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod --depth=1
```
<!-- TODO: 子模板更新 -->
随后，我们只需将主题名称写入配置文件 `./config.yml`：
```yaml
baseURL: https://jaredyam.github.io
languageCode: en-us
defaultContentLanguage: zh
title: 👨‍🌾 Jared Yam
# 主题
theme: PaperMod
```
便搭建完成了一个完整的静态博客网站。

执行以下命令可以在本地预览网站：
```bash
hugo server -D
```
其中选项 `-D` 允许在预览网站时渲染未发布的草稿文档。同时，我们还可以尝试新建一个文档：
```bash
hugo new posts/quickstart.md
```
刷新并点击进入该文档对应的网页视图。

除此之外，你还可以跟随[主题配置指引](https://github.com/adityatelange/hugo-PaperMod/wiki/Features#default-theme-lightdarkauto)，根据个人喜好调整主题基础设置，添加或取消部分功能。如果你没有其它定制主题的需求，可直接跳过下一节内容。

## 定制基础主题

定制主题除了修改配置文件，还可以[重写模板和风格](https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs#override-theme-template)。接下来介绍的个性化配置可能持续更新，且细节部分不做赘述。如果你对我的网站主题配置感兴趣，可以随时查看其对应的[仓库源文件](https://github.com/jaredyam/jaredyam.github.io)。

> <strong>❓ 如何查看我在基础主题上做过哪些调整？</strong>
> ```bash
> git clone https://github.com/jaredyam/jaredyam.github.io.git
> cat ./config.yml
> diff -ur themes/PaperMod/assets assets
> diff -ur themes/PaperMod/layouts layouts
> ```

### 转移目录至侧边栏

[Pull Request #675 · adityatelange/hugo-PaperMod](https://github.com/adityatelange/hugo-PaperMod/pull/675) 提出将文章目录转移至侧边栏，可以轻松实现上下文跳转。截至发文这一特性还未并入主分支，我们可以让主题子模块追踪该远程 PR 分支：
```bash
cd themes/PaperMod
git fetch origin pull/675/head:toc-on-the-side --depth=1
git checkout toc-on-the-side
cd ../..
```
除了切换到 PR 分支，我们还需要让子模块配置文件 `./gitmodules` 指向 PR 分支：
```ini
[submodule "themes/PaperMod"]
    path = "themes/PaperMod"
    url = "https://github.com/nielsbrakel/hugo-PaperMod.git"
    branch = "toc-on-the-side"
```
然后再同步子模块 URL 到 `.git/config`：
```bash
git submodule sync
```
基础主题默认不显示目录栏，我们需要在主题配置文件 `./config.yml` 中启用并展开目录：
```yaml
baseURL: https://jaredyam.github.io
languageCode: en-us
defaultContentLanguage: zh
title: 👨‍🌾 Jared Yam
# 主题
theme: PaperMod

params:
  # 目录
  ShowToc: true
  TocOpen: true
```

值得注意的是，远程 PR 分支可能并不涵盖主分支的最新特性。你可以通过创建一个 fork 仓库同时追踪主分支和 PR 分支的更新。

### 添加评论区

[utterances](https://utteranc.es/) 是一个基于 GitHub issues 的评论系统，包含多种主题可供选择。你应该首先跟随 [utterances 配置方法](https://utteranc.es/)了解可选配置项，并根据个人偏好自动生成一份包含个性化配置属性的 `HTML` 代码：
```html
<script src="https://utteranc.es/client.js"
        repo="jaredyam/jaredyam.github.io"
        issue-term="title"
        label="Comment"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

官方提供的自动生成代码仅支持设置一个固定主题，而我们希望网站主题和评论区主题亮暗模式可以保持同步。参照 [Issue #427 · utterance/utterances](https://github.com/utterance/utterances/issues/427) 的讨论，上述代码可以修改为：
```html
<div class="comments">
    <script>
    function loadComment() {
        let theme = localStorage.getItem('pref-theme') === 'dark' ? 'github-dark' : 'github-light';
        let s = document.createElement('script');
        s.src = 'https://utteranc.es/client.js';
        s.setAttribute('repo', '{{- .Site.Params.commentsParams.repo -}}');
        s.setAttribute('issue-term', '{{- .Site.Params.commentsParams.issueTerm -}}');
        s.setAttribute('label', '{{- .Site.Params.commentsParams.label -}}');
        s.setAttribute('theme', theme);
        s.setAttribute('crossorigin', 'anonymous');
        s.setAttribute('async', '');
        document.querySelector('div.comments').innerHTML = '';
        document.querySelector('div.comments').appendChild(s);
    }
    loadComment();
    </script>
</div>
```
将修改后的代码写入 `./layouts/partials/comments.html` 即可。

不难看出，我们还需要在主题配置文件 `./config.yml` 中添加评论区参数：
```yaml
baseURL: https://jaredyam.github.io
languageCode: en-us
defaultContentLanguage: zh
title: 👨‍🌾 Jared Yam
# 主题
theme: PaperMod

params:
  # 目录
  ShowToc: true
  TocOpen: true
  # 评论区
  comments: true
  commentsParams:
    repo: jaredyam/jaredyam.github.io
    issueTerm: title
    label: Comment
```
其中 `repo`，`issueTerm`，`label` 的属性值应当与 utterances 自动生成代码保持一致。

除了在加载网站时使其主题和评论区亮暗模式保持一致外，手动切换网站主题亮暗模式的同时也应该改变评论区主题。在 `./layouts/partials/footer.html:72` 区域加入如下代码块：
```diff
{{- if (not .Site.Params.disableThemeToggle) }}
<script>
    document.getElementById("theme-toggle").addEventListener("click", () => {
        if (document.body.className.includes("dark")) {
            document.body.classList.remove('dark');
            localStorage.setItem("pref-theme", 'light');
+           const message = {
+               type: 'set-theme',
+               theme: 'github-light'
+           };
+           const iframe = document.querySelector('.utterances-frame');
+           iframe.contentWindow.postMessage(message, 'https://utteranc.es');
        } else {
            document.body.classList.add('dark');
            localStorage.setItem("pref-theme", 'dark');
+           const message = {
+               type: 'set-theme',
+               theme: 'github-dark'
+           };
+           const iframe = document.querySelector('.utterances-frame');
+           iframe.contentWindow.postMessage(message, 'https://utteranc.es');
        }
    })
</script>
{{- end }}
```
即可实现这一目的。

## 部署网站服务

Hugo 提供非常详尽的 [GitHub Pages 部署指引](https://gohugo.io/hosting-and-deployment/hosting-on-github/)。这里对部署过程做简单梳理：

1. 在 `./.github/workflows/` 中放入 [gh-pages.yml](https://gohugo.io/hosting-and-deployment/hosting-on-github/#build-hugo-with-github-action)；
2. 将本地网站同步到 GitHub 同名仓库；
3. 在仓库设置 `Settings -> Pages` 中选择 `gh-pages` 作为网站资源分支。

完成以上步骤，你便可以通过 _https://\<your-github-id>.github.io_ 访问自己的个人博客。
