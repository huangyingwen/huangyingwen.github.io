+++
title = "Post 1"
date = 2017-07-12T17:31:56-04:00
draft = false
+++

Export this **first** post only by bringing point here and doing `M-x org-hugo-export-wim-to-md`.


## ox-hugo 配置 {#ox-hugo-配置}


### 每个组织子树一篇文章 {#每个组织子树一篇文章}

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

HUGO_BASE_DIR
: Hugo 站点的源目录。如果设置为 ~/hugo/ ，则导出的 Markdown 文件将保存到 ~/hugo/content/&lt;HUGO_SECTION&gt;/ 目录。默认情况下，Markdown 文件驻留在站点根目录下的 content/ 层次结构中。

    如果尝试导出而不设置此属性，则会收到此错误：
    ```text
    user-error: It is mandatory to set the HUGO_BASE_DIR property
                or the `org-hugo-base-dir' local variable
    ```
    可以通过以下两种方式之一设置此属性：

    1.  在组织文件中设置 `#+hugo_base_dir:` 关键字。
    2.  在 `.dir-locals.el` 或 文件局部变量中设置 `org-hugo-base-dir` 变量。
