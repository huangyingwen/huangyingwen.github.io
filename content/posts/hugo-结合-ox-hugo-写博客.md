+++
title = "hugo 结合 ox-hugo 写博客"
author = ["huangyingwen"]
lastmod = 2023-07-28T10:32:16+08:00
tags = ["hugo", "org"]
categories = ["emacs"]
draft = true
weight = 2002
+++

## 安装配置 hugo {#安装配置-hugo}

1.  archlinux 安装 git 和 hugo
    ```bash
    sudo pacman -S hugo git
    ```
2.  初始化网站通过 **hugo** 初始化网站
    ```bash
    hugo new site myblog
    ```
    cd 到 myblog 目录
    ```bash
    cd ./myblog
    ```
    git 初始化 当前目录
    ```bash
    git init
    ```
    将 Ananke 主题克隆到 themes 目录中，将其作为 Git 子模块添加到项目中。
    ```bash
    git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
    ```
    设置主题
    ```bash
    echo "theme = 'ananke'" >> hugo.toml
    ```
    启动 Hugo 的开发服务器以查看站点。
    ```bash
    hugo server
    ```


## GitHub Pages 配置 {#github-pages-配置}


### 添加 GitHub 仓库 {#添加-github-仓库}

<span class="timestamp-wrapper"><span class="timestamp">[2023-07-18 二 15:45]</span></span>
在 GitHub 个人账户下新建 Repository, Repository name 的位置填写域名，格式是 {username}.GitHub.io
添加成功之后，复制 code ssh 地址，并添加远程仓库

```bash
git remote add origin git@github.com:{username}/{username}.github.io.git
```


### 通过 GitHub action 自动发布博客 {#通过-github-action-自动发布博客}

添加 **.github/workflows/hugo.yaml** 文件

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - master

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.114.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```


## 配置 ox-hugo {#配置-ox-hugo}


### spacemacs 开启 {#spacemacs-开启}

```emacs-lisp
(setq-default dotspacemacs-configuration-layers
              '((org :variables
                     org-enable-hugo-support t)))
```


### ox-hugo 配置 {#ox-hugo-配置}

所有文章都在一个文件中


#### 导出配置 {#导出配置}

HUGO_BASE_DIR
: Hugo 站点的源目录。如果设置为 ~/hugo/ ，则导出的 Markdown 文件将保存到 ~/hugo/content/&lt;HUGO_SECTION&gt;/ 目录。默认情况下，Markdown 文件驻留在站点根目录下的 content/ 层次结构中。如果尝试导出而不设置此属性，则会收到此错误：
    ```text
    user-error: It is mandatory to set the HUGO_BASE_DIR property
                or the `org-hugo-base-dir' local variable
    ```
    可以通过以下两种方式之一设置此属性：

    1.  在组织文件中设置 `#+hugo_base_dir:` 关键字。
    2.  在 `.dir-locals.el` 或 文件局部变量中设置 `org-hugo-base-dir` 变量。


HUGO_SECTION
: 所有帖子的默认 Hugo 版块名称。有关 Hugo 版块的更多信息，请参见
    [此处](https://gohugo.io/content-management/sections/)。该属性通常设置为文章或博客。默认值通过 org-hugo-default-section-directory
    设置。详情请参阅 [Hugo Section](https://ox-hugo.scripter.co/doc/hugo-section/)。

**重要提示**: 如果您选择将 Org 子树导出为帖子，则需要设置 `EXPORT_FILE_NAME` 子树属性。此包使用该属性来确定当前帖子的开始位置。因此，具有属性的子树不能嵌套具有
`EXPORT_FILE_NAME` 该属性的另一个子树。如果可以类比[分支/叶数据结构](https://en.wikipedia.org/wiki/Tree_(data_structure))术语，则具有属性的 `EXPORT_FILE_NAME` 子树必须是叶节点。


#### 导出快捷键 {#导出快捷键}

`, e e H H`
: 导出" What I Mean ”。这与在 Emacs Lisp 中交互或通过
    `(org-hugo-export-wim-to-md)` 调用 `org-hugo-export-wim-to-md` 函数相同。如果 point 位于有效的雨果发布子树中，则将该子树导出为 Markdown 格式的 Hugo post。

`, e e H H`
: 导出所有“ What I Mean ”。这与在 Emacs Lisp 中执
    `(org-hugo-export-wim-to-md :all-subtrees)` 相同。如果组织文件有一个或多个“有效的雨果帖子子树”，请将它们导为 Markdown 格式的 Hugo post。


### org-mode-capture 配置 {#org-mode-capture-配置}

如果您不想为每个新帖子手动键入 ， EXPORT_FILE_NAME 以下是组织捕获模板可以提供帮助的示例

```emacs-lisp
;; Populates only the EXPORT_FILE_NAME property in the inserted heading.
(with-eval-after-load 'org-capture
  (defun org-hugo-new-subtree-post-capture-template ()
    "Returns `org-capture' template string for new Hugo post.
See `org-capture-templates' for more information."
    (let* ((title (read-from-minibuffer "Post Title: ")) ;Prompt to enter the post title
           (fname (org-hugo-slug title)))
      (mapconcat #'identity
                 `(
                   ,(concat "* TODO " title)
                   ":PROPERTIES:"
                   ,(concat ":EXPORT_FILE_NAME: " fname)
                   ":END:"
                   "%?\n")          ;Place the cursor here finally
                 "\n")))

  (add-to-list 'org-capture-templates
               '("h"                ;`org-capture' binding + h
                 "Hugo post"
                 entry
                 ;; It is assumed that below file is present in `org-directory'
                 ;; and that it has a "Blog Ideas" heading. It can even be a
                 ;; symlink pointing to the actual location of all-posts.org!
                 (file+olp "all-posts.org" "Blog Ideas")
                 (function org-hugo-new-subtree-post-capture-template))))
```

上面的捕获将自动插入一个以 `TODO` 为前缀的标题。将 `org-log-done` 设置为 `'time`
时，在将 `TODO` 状态更改为 `DONE` 状态 (`C-c C-t`) 时，标题下方将自动插入一个名为
CLOSED 的 [_Special Property_](https://orgmode.org/manual/Special-Properties.html "Emacs Lisp: (info \"(org) Special Properties\")")。下面是一个例子。

```text
*** DONE Narrowing the Author column in Magit                       :org:log:
CLOSED: [2017-12-18 Mon 16:36]
```


### 保存时自动导出 {#保存时自动导出}

每次保存文件自动导出并预览，需要启用次要模式 `org-hugo-auto-export-mode`
以下是所有文章都在一个文件中的配置


#### 1. 将以下内容添加到 Org 文件的最后并保存文件： {#1-dot-将以下内容添加到-org-文件的最后并保存文件}

```org
* Footnotes
* COMMENT Local Variables                          :ARCHIVE:
# Local Variables:
# eval: (org-hugo-auto-export-mode)
# End:
```


#### 2. 启动引擎（Hugo Server） {#2-dot-启动引擎-hugo-server}

启动 hugo server 以便每次保存组织文件时都可以看到实时预览。

在 Hugo 站点根目录（包含站点 config.toml 的目录）中运行以下命令以启动服务器：

```text
hugo server -D --navigateToChanged
```


#### 3. 打开浏览器 {#3-dot-打开浏览器}

默认情况下，站点在本地主机上的端口 1313 上本地提供。因此，上述步骤将在最后打印如下内容：

```text
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```


## 参考 {#参考}

-   <https://zlearning.netlify.app/linux/emacs/emacs-org-hugo.html>
-   <https://blog.xuxiangyang.com/posts/org/ox-hugo/>
-   <https://ox-hugo.scripter.co/>


[//]: # "Exported with love from a post written in Org mode"
[//]: # "- https://github.com/kaushalmodi/ox-hugo"