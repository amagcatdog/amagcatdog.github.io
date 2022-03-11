---
layout: wiki
title: 使用 jekyll 构建 github 主页
cate1: tools
cate2: jekyll
description: 使用 jekyll 构建 github 主页
keywords: github pages, jekyll, blog
---

## 搭建过程

### jekyll 安装

jekyll 是一个将纯文本转换为静态博客网站的工具，支持自定义页面布局，不需要配置数据库，评论功能可以接入`gitalk`等第三方组件。该工具也是 github pages 推荐的建站工具。

1. 安装 jekyll

    参考[官方教程](https://jekyllrb.com/docs/installation/windows/)安装，推荐安装到 linux 系统，我是通过 windows 的 wsl 安装，只需要正确安装 ruby 后，通过 ruby 的包管理工具安装 jekyll 即可：

    ```bash
    sudo apt-get update -y && sudo apt-get upgrade -y

    sudo apt-add-repository ppa:brightbox/ruby-ng
    sudo apt-get update
    sudo apt-get install ruby2.5 ruby2.5-dev build-essential dh-autoreconf

    gem update
    gem install jekyll bundler

    jekyll -v
    ```

2. 创建 jekyll 项目

    使用 jekyll 创建新的项目，可以以本地服务的方式进行预览：

    ```bash
    jekyll new ./
    jekyll build
    bundle exec jekyll serve
    ```

    至此 jekyll 搭建完成，将自己的文档放到 `_posts`目录下，以下面格式命名，建议文件名不要含有中文，否则可能存在编码问题。

    ```txt
    2011-12-31-new-years-eve-is-awesome.md
    2012-09-12-how-to-write-a-blog.md
    ```

    重新 build 之后，对应的页面将生成到`_site`目录下。

### github pages 搭建

参考[github pages教程](https://pages.github.com/)搭建个人页面，先创建用于存储页面的 public 仓库，仓库名称要与个人 github 账号名称相同，生成的个人主页地址是`yourgithubname.github.io`。

第二步将仓库 clone 到本地，使用上面的 jekyll 命令在仓库内创建项目，注意 github 已经集成了 jekyll ，只需要将 jekyll 项目推送到远端仓库中，就会自动进行构建，生成对应的`_site`目录，因此本地预览的`_site`目录不需要推送到远端仓库，可以在`.gitignore`文件中进行排除。

### 使用自定义模板

网上有些自定义的 jekyll 模板可以美化自己的页面，几个模板集中营：

- [https://github.com/topics/jekyll-theme](https://github.com/topics/jekyll-theme)
- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://github.com/jekyll/jekyll/wiki/Sites](https://github.com/jekyll/jekyll/wiki/Sites)

我的主页目前使用的模板仓库`https://github.com/mzlogin/mzlogin.github.io`。

自定义模板的使用也比较简单，将自定义模板仓库内的相关文件拷贝到自己的仓库，替换已有的 jekyll 相关目录，按照自定义模板的[使用说明](https://github.com/mzlogin/mzlogin.github.io)修改相关的配置和个人信息即可，主要的配置文件是`_config.yml`。

## trouble shooting

### case 1

本地预览时打印`GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.`，按照网上的一些方法设置了 github token 等环境变量，但并没有效果，后来发现不影响使用，没有继续研究解决方法。

### case 2

本地预览时没有从cnd下载到相关的css和js文件，可能是网络问题，目前没有找到好的解决方法，后面自动恢复了。

可以尝试访问下面两个链接，刷新本地缓存：

- <https://purge.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js>
- <https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js?v=1.7.2>

### case 3

gitalk 配置。需要申请一个 github oauth application ，其中的 Homepage URL 和 Authorization callback URL 设置为个人主页的地址，比如本地预览时可以设置为`http://localhost:4000/`。

## 参考

- [setting up a github pages site with jekyll](https://docs.github.com/cn/pages/setting-up-a-github-pages-site-with-jekyll)
