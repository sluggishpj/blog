---
title: 自动化构建并部署hexo博客
date: 2019-10-23 10:08:24
tags:
  - travis
  - ci
  - hexo
  - git submodule
categories:
  - CI&CD
---

## 说明

记录如何使用 travis-ci 自动化构建 hexo 博客并部署到 github pages。
最终效果：只需编写 markdown，然后推送到 github，之后会通过 travis-ci 自动化构建并部署到指定仓库的 github pages。

<!--more-->

## 前提条件

1. install node.js
2. install yarn
3. install git
4. install hexo

```bash
$ npm install -g hexo
```

> node.js, git 自行安装

## 流程

### 初始化操作

#### 初始化博客文件夹

```bash
$ hexo init blog
```

#### 将博客文件夹初始化 git 项目

1. 初始化

```bash
$ git init
```

2. 添加目标远程仓库

```bash
$ git remote add origin git@github.com:sluggishpj/blog.git
$ git push -u origin master
```

> 参考[Git 教程](https://www.liaoxuefeng.com/wiki/896043488029600/898732864121440)

## 项目配置

### 初始化配置

按官网说明配置下根目录下的 \_config.yml

> 官网：https://hexo.io/zh-cn/docs/configuration
> 可对比参照[本博客的\_config.yml](https://github.com/sluggishpj/blog/blob/master/_config.yml)，注意配置中的`主题配置`稍后会说明。

### 添加主题模块

可以添加自己看上的的主题模块，这里使用 next 主题

```bash
$ cd themes\
$ git submodule add https://github.com/theme-next/hexo-theme-next next
```

#### 为什么使用 git submodule

1. 主题是一个独立的仓库，本博客也是一个独立的项目。但是 git 项目里面不能直接包含另一个 git 项目，但是可以使用 submodule 达到同样的效果
2. 可根据需要更新主题仓库。如果把主题仓库复制过来，移除其`.git`文件夹，则不方便更新。

#### git submodule 常用命令

以下均在目标项目中操作，而不是在子模块目录中。

1. 添加子模块

```bash
$ git submodule [--quiet] add [<options>] [--] <repository> [<path>]
```

2. clone 包含子模块的项目，需进行子模块的初始化

```bash
$ git submodule init
$ git submodule update
```

3. 更新子模块

```bash
$ git submodule update --remote
```

> git submodule 更多请参考：[Git 工具-子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

4. 删除子模块

```bash
# 逆初始化模块，其中{MOD_NAME}为模块目录，执行后可发现模块目录被清空
$ git submodule deinit {MOD_NAME}
# 删除.gitmodules中记录的模块信息（--cached选项清除.git/modules中的缓存）
$ git rm --cached {MOD_NAME}
# 提交更改到代码库，可观察到'.gitmodules'内容发生变更
$ git commit -am "Remove a submodule."
```

### 新增博客所需功能

这里新增搜索、评论、分类、标签

#### 新增页面搜索

本博客使用 Local Search

1. 安装所需依赖

```bash
$ yarn add hexo-generator-searchdb
```

2. 配置根目录下的\_config.yml，新增主题配置。内容是从 next/\_config.yml 复制过来，修改 enable 为 true 而已。

```yml
# 主题配置
theme_config:
  local_search:
    enable: true
    # If auto, trigger search by changing input.
    # If manual, trigger search by pressing enter key or search button.
    trigger: auto
    # Show top n results per article, show all results by setting to -1
    top_n_per_article: 1
    # Unescape html strings to the readable one.
    unescape: false
    # Preload the search data when the page loads.
    preload: false
```

#### 新增评论

本博客使用 gitalk

1. 获取 client_id 和 client_secret。[点击新增一个 Github APP](https://github.com/settings/applications/new)
2. 新增完后保存 client_id 和 client_secret，下面的配置要用到。
   ![KJ5IN8.png](https://s2.ax1x.com/2019/10/23/KJ5IN8.png)

3. 修改项目根目录的\_config.yml，在主题配置 theme_config 中新增 gitalk 相关配置，其实也是 next/\_config.yml 中复制过来的，更改 enable 为 true 而已。

```yml
# 主题配置
theme_config:
  local_search:
    enable: true
    # If auto, trigger search by changing input.
    # If manual, trigger search by pressing enter key or search button.
    trigger: auto
    # Show top n results per article, show all results by setting to -1
    top_n_per_article: 1
    # Unescape html strings to the readable one.
    unescape: false
    # Preload the search data when the page loads.
    preload: false
  gitalk:
    enable: true
    github_id: sluggishpj
    repo: blog
    client_id: dd946c3d0eb6bxxxxxxxx
    client_secret: 5051b237a867f9673700xxxxxxxx
    admin_user: sluggishpj
    distraction_free_mode: true # Facebook-like distraction free mode
    # Gitalk's display language depends on user's browser or system environment
    # If you want everyone visiting your site to see a uniform language, you can set a force language value
    # Available values: en | es-ES | fr | ru | zh-CN | zh-TW
    language:
```

#### 添加分类和标签

##### 创建分类和标签 page

```bash
$ hexo new page categories
$ hexo new page tags
```

此时会在 source 中出现 categories 和 tags 文件夹。
依次进修改 2 个文件夹中的 index.md。categories 中的 index.md 添加 type 字段，并设置为 categories；tags 中的 index.md 添加 type 字段，并设置为 tags。

以下是 categories/index.md 的内容

```md
---
title: categories
date: 2019-10-13 20:38:19
type: categories
---
```

以下是 tags/index.md 的内容

```md
---
title: tags
date: 2019-10-13 20:58:33
type: tags
---
```

##### 修改根目录下的\_config.yml，在主题配置 theme_config 中新增相关配置。其实也是从 next/\_config.yml 中复制过来的。

```yml
# 主题配置
theme_config:
  # 本地搜索
  local_search:
    enable: true
    # If auto, trigger search by changing input.
    # If manual, trigger search by pressing enter key or search button.
    trigger: auto
    # Show top n results per article, show all results by setting to -1
    top_n_per_article: 1
    # Unescape html strings to the readable one.
    unescape: false
    # Preload the search data when the page loads.
    preload: false

  # 评论
  gitalk:
    enable: true
    github_id: sluggishpj
    repo: blog
    client_id: dd946c3d0eb6bxxxxxxxx
    client_secret: 5051b237a867f9673700xxxxxxxx
    admin_user: sluggishpj
    distraction_free_mode: true # Facebook-like distraction free mode
    # Gitalk's display language depends on user's browser or system environment
    # If you want everyone visiting your site to see a uniform language, you can set a force language value
    # Available values: en | es-ES | fr | ru | zh-CN | zh-TW
    language:

  # 添加分类和标签
  menu:
    home: / || home
    #about: /about/ || user
    tags: /tags/ || tags
    categories: /categories/ || th
    archives: /archives/ || archive
    #schedule: /schedule/ || calendar
    #sitemap: /sitemap.xml || sitemap
    #commonweal: /404/ || heartbeat
```

> 到此可以先将项目 push 到 github 上，之后配置自动化构建及部署

### 自动化构建及部署

#### 接入 Travis-ci

1. 使用 github 账户进行注册：https://travis-ci.org
2. 在 github 新增一个 access token，并保存在你看得到的地方：https://github.com/settings/tokens
   ![KJqORf.png](https://s2.ax1x.com/2019/10/23/KJqORf.png)
3. 在 travis-ci 中对应的博客项目配置该 token。下方.travis.yml 中的`$GITHUB_TOKEN`引用的就是刚配置的变量`GITHUB_TOKEN`
   ![KJLKoR.png](https://s2.ax1x.com/2019/10/23/KJLKoR.png)

#### 新增.travis.yml 配置文件

在项目根目录下新增.travis.yml

```yml
language: node_js
node_js: stable
sudo: required

# 构建的分支
branches:
  only:
    - master

before_install:
  # Repo for Yarn
  # RF：https://yarnpkg.com/lang/en/docs/install-ci/
  - sudo apt-key adv --fetch-keys http://dl.yarnpkg.com/debian/pubkey.gpg
  - echo "deb http://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
  - sudo apt-get update -qq
  - sudo apt-get install -y -qq yarn

  # 安装对应版本的 theme next
  - git submodule update --init

install:
  - yarn install

script:
  - hexo g

deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN
  local-dir: public
  target-branch: gh-pages
  keep-history: false
  on:
    branch: master

cache:
  yarn: true
```

> push 到 github 上，完毕

## 参考文章

> [简书-删除子模块](https://www.jianshu.com/p/ed0cb6c75e25) > [Git 工具-子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97) > [Travis-ci 自动编译部署 github 上的项目](https://troyyang.com/2017/06/24/Travis_Auto_Build_Deploy_Github_Projects/)
