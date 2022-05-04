---
filename: "hugo-netlify-cms-support"
title: "Hugo 添加 Netlify CMS 支持"
subtitle: ""
date: 2022-05-04T21:15:39+08:00
draft: false
author: ""
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: "本文以 CC BY-NC 4.0 许可证发布"
comment: false
weight: 0

tags:
  - Hugo
  - Netlify CMS
categories:
  - Tech

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

# See details front matter: /theme-documentation-content/#front-matter
---

为了解决偶尔在手机上写博客这一~~不存在~~的需求，考虑为这个博客添加一个 CMS。在很早以前，我就用过 Netlify CMS，比较熟悉，所以就继续用它了。

这里假设源码储存在 GitHub 上，且博客部署在 Vercel 上，其他原理类似。

## 原理
像 Netlify CMS 这样的 Headless CMS，一般是独立于静态博客的其他组件的。以我这个博客为例，使用 Git 来管理由 makrdown 组成的博客原始内容，储存在 GitHub 上，并通过 GitHub Action 来构建，最后部署到 Vercel 上。而 Netlify CMS 要与它们一起工作，需要做比较多的工作。

整个 Netlify CMS 的部署要考虑以下几个内容：

 - 获取博客源代码
 - 提供修改的 UI
 - 提交对内容的修改

接下来就对这几个部分进行讲解。

## 添加 Netlify CMS 到站点
Hugo 中，任何放在 `static` 目录下的东西都不会被渲染，所以在 `static` 目录下添加一个目录，用于访问 CMS，这里就叫 `management`，然后按照[官方的步骤](https://www.netlifycms.org/docs/add-to-your-site/)添加两个文件 `config.yml` 和 `index.html`，分别表示 CMS 配置文件和 CMS 访问的 UI。

目录结构如下：

```plain
blog
└── static
    └── management
        ├── config.yml
        └── index.html
```

在 `index.html` 中添加以下代码：

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Content Manager</title>
</head>
<body>
  <script src="https://unpkg.com/netlify-cms@^2.0.0/dist/netlify-cms.js"></script>
  <!-- Alternative CDN: https://cdn.jsdelivr.net/npm/netlify-cms@^2.0.0/dist/netlify-cms.js -->
</body>
</html>
```

接下来填写配置文件 `config.yml`，建议现在本地测试：

```yaml
backend:
  name: github
  repo: owner/repo # 博客源码仓库
  branch: your-branch # 源码的分支

publish_mode: "editorial_workflow"

media_folder: "static/images" # 图片存放的目录
public_folder: "images" # 构建后图片存放的目录

# collections 中的每一项代表一种文章类型
collections:
  - name: "posts"
    label: "Posts"
    folder: "content/posts"
    path: "{{slug}}/index" # {{slug}} 即 {{fields.filename}}
    slug: "{{fields.filename}}"
    media_folder: "" # 相对于 content/posts/{{fields.filename}}
    public_folder: "" # 相对于 content/posts/{{fields.filename}}
    create: true # 允许创建新文章
    fields:
      - {label: "File Name", name: "filename", widget: "string", default: "", required: false}
      - {label: "Title", name: "title", widget: "string"}
      - {label: "Date", name: "date", widget: "datetime"}
      - {label: "Draft", name: "draft", widget: "boolean", default: true}
      - {label: "Tags", name: "tags", widget: "list"}
      - {label: "Categories", name: "categories", widget: "list"}
      - {label: "Body", name: "body", widget: "markdown"}
```

这应该还是比较好懂的，我采用了 Hugo 的 Page Bundle 功能，每个位置单独一个目录，其中 `index.md` 存放内容，同时图片等资源也放在同一目录，可以直接以相对路径的方式引用。当然，Netlify CMS 现在只是提供了全局的 `media_folder` 的上传，其他的还是要手动用 Git 上传。

在 `collections.fields` 中配置的是文章的处理，除了最后一行的 `- {label: "Body", name: "body", widget: "markdown"}` 是用于编写正文的，其他都是处理元信息，通过 Front-Matter 的方式存储。

注意到 `File Name` 那一行是为了让 Netlify CMS 能够知道文章是在哪个目录下的，如果你不介意直接 Netlify CMS 生成的默认 `slug` 作为目录的名字，那可以直接去掉 `slug: "{{fields.filename}}"` 这一项。

进阶配置可以参考以下链接：

 - <https://www.netlifycms.org/docs/configuration-options/>
 - <https://www.netlifycms.org/docs/collection-types/>
 - <https://www.netlifycms.org/docs/widgets/>

接下来在本地运行 `hugo serve`，访问 `http://localhost:1313/management/`，应该就可以正常访问了。

## 添加认证服务
接下来如果直接把代码部署到 GitHub 上，你会发现登录。这是因为 Netlify CMS 相对于 GitHub 是第三方应用，想要访问和修改你的仓库是需要授权的，这要通过 OAuth 解决。如果使用 Netlify 部署网站的话，Netlify 已经做好了相关工作，而使用 Vercel，就缺少一个 OAuth 的认证服务器。好在有人已经做出了现成的解决方案了，我们可以不用再造轮子。

### 部署 OAuth 服务器
建议先看看 [netlify-cms-oauth](https://github.com/ublabs/netlify-cms-oauth) 的 readme，接下来就使用它。

按照 readme 里的办法，我们只要点击那个蓝色的 Deploy 按钮，就可以把这个认证服务部署到自己的 Vercel 上，中途可能要求填写 `OAUTH_GITHUB_CLIENT_ID` 等环境变量，这个可以先随便填，和后面的东西无关。或者直接使用这个人的部署好的服务。

### 创建 GitHub OAuth App
接下来，假设你使用的服务器的域名是 `netlify-cms-oauth.vercel.app`。

在 [Developer settings](https://github.com/settings/developers) 中点击 New OAuth App，在 Homepage URL 中填上你的博客的 URL，Authorization callback URL 填上 `https://netlify-cms-oauth.vercel.app/callback`，创建即可。

接下来再点击 Generate a new client secret，用于接下来的操作。

### 设置 Vercel 环境变量
在 Vercel 中的博客站点的设置中编辑 Environment Variables，添加 `OAUTH_GITHUB_CLIENT_ID` 为 Client ID，`OAUTH_GITHUB_CLIENT_SECRET` 为 Client secrets。

我不知道要不要把上面的 OAuth 认证服务器的环境变量也改成这个，但是我测试的结果是没有影响。

### 修改配置文件
在 `config.yml` 的 `backend` 中添加 `base_url: https://netlify-cms-oauth.vercel.app`，设置认证服务器。

接下来再部署，应该就可以使用了。