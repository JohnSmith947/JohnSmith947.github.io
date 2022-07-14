---
title: 博客搭建流程
date: 2022-07-14 23:25:00
# img: /source/images/xxx.jpg
summary: 我姥姥看了都会的博客搭建流程。
categories: Blog
tags:
  - 教程
---

​	简单记录一下博客的搭建流程，都是采用现成的框架，操作起来并不复杂。之前碰到过一些奇怪的问题，后来也慢慢的也找到问题了，现在有时间来记录一下流程。

​	Hexo 是一个快速、简洁且高效的博客框架，重点是使用简单并且完全免费。在官网可以找到详细的说明（ https://hexo.io/zh-cn/docs/ ）。



### 搭建流程：

#### 首先需要安装以下两个软件：

- 安装 node.js （建议版本12.0及以上）
- 安装 git

以上两个软件的安装步骤在网上很简单就能找到，在这里不赘述。



#### 安装Hexo框架：

```shell
npm install -g hexo-cli
```

一般不会出现什么问题，如果出现了问题，可以参照官网的文档进行解决。

安装完成后，可以通过两种方式执行Hexo指令：

1. `npx hexo <command>`
2. 添加环境变量，将Hexo所在目录下的`node_modules`添加到环境变量中，即可直接使用`hexo <command>`。



#### 生成博客：

```shell
npx hexo init <folder>
cd <folder>
npm install
npx hexo server  # server 可以简化为 s
```

执行完以上命令后可以通过本地浏览器直接访问 http://localhost:4000。



#### 更换主题：

​	通过上面的步骤，在本地访问应该可以初步看到网站的雏形。在网站的样式和主题方面Hexo可玩性很高，可以在网上寻找自己喜欢的主题并且替换上，就我用的主题来举例：

​	该主题的详细介绍都在源码的自述文件中，包括一些配置和样式修改方法，都有详细的讲解，熟悉 js 的可能事半功倍，建议在更换完成后看一下该主题的自述文件讲解。

> 主题名称：hexo-theme-matery
>
> 示例网站：http://blinkfox.com/
>
> 主题源码：https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md

​	步骤：

> 1. 在 themes 文件夹下，可以先清空该文件夹自带的文件，然后使用 git clone 命令来下载该主题：
>
> ```shell
> git clone https://github.com/blinkfox/hexo-theme-matery.git
> ```
>
> 2. 在根目录的配置文件 `_config.yml` 中将 theme 的值由  `landscape` 改为 `hexo-theme-matery`（与刚下载的主题文件夹的名称一致）；
> 3.  可以将根目录下的 `config.landscape.yml` 文件删除；
> 4. 将 `themes/hexo-theme-matery` 文件夹中的 `.git` 文件夹删除；
> 5. 更改语言为中文：在根目录的配置文件 `_config.yml` 中将 `language` 的值改为 `zh-CN`；
> 6. 以上步骤执行完后，重新运行服务可以确认主题是否更换成功。



#### 通过 Github 部署：

> ​	本来可以通过 Travis 自动进行集成部署的，每次只要 push 到远程仓库就可以了，但是现在 Travis 好像免费的名额用完了，不选择收费的 plan 做不到自动部署，只能选择手动进行部署了，也不算复杂，多两个命令而已。

1. 在 Github 创建账号，并新建一个 repository，你的 repository 应该直接命名为 <你的 Github 用户名>.github.io。
2. 将你的 Hexo 站点文件夹推送到 repository 中。不要将 public 目录推送到repository 中，检查.gitignore文件中是否包含 public 一行，如果没有请加上。
3. 在博客的根目录执行以下命令：

```shell
git init
git add .
git commit -m "init"
git remote add origin <你创建的仓库的连接>
git push -u origin master
```

4. 安装 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git) ：

```shell
npm install hexo-deployer-git --save
```

5. 此时远程仓库的 `master` 分支是我们的源代码，但是还需要将博客的页面单独生成一个分支，在博客根目录  `_config.yml` 中更改如下配置：

```yaml
deploy:
  type: git
  repo: <仓库地址>
  branch: blog  # 这里是可以自定义的
```

6. 将以上更改提交到远程仓库；
7. 执行以下命令：

```shell
hexo clean  # 清除缓存文件
hexo generate  # 构建静态文件，gnerate 可简写为 g
hexo deploy  # 部署网站，deploy 可简写为 d
```

8. 此时 Github 的 `Action` 中应该已经开始部署了，并且你的仓库自动生成了 `blog` 分支，本次部署肯定会失败，先不管它，在仓库的 `Settings` 中找到 `Pages` 选项，在其中的 `Source` 下面可以选择你的页面是由哪个分支部署，此时选择 `blog` ，并且点击 `Save` 按钮。
9. 再观察 `Action` 中部署任务，本次成功后就可以直接在线访问你的博客网站了。
10. 关于文章的撰写和修改，只需要在 `source\_post` 文件夹下新建或者修改 `.md` 文件，保存后执行步骤 7 的命令即可。

> ​	源代码跟网页的代码不属同一分支，所以在 `master` 分支执行完部署命令后，定期还是将源代码 push 一下的好。



#### ~~通过 Travis CI 部署~~:

1. ~~在 Github 创建账号，并新建一个 repository，你的 repository 应该直接命名为 <你的 Github 用户名>.github.io。~~
2. ~~将你的 Hexo 站点文件夹推送到 repository 中。不要将 public 目录推送到repository 中，检查.gitignore文件中是否包含 public 一行，如果没有请加上。~~
3. ~~将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 GitHub 账户中。~~
4. ~~前往 GitHub 的 [Applications settings](https://github.com/settings/installations)，配置 Travis CI 权限，使其能够访问你的 repository。~~
5. ~~GitHub 新建 [Personal Access Token](https://github.com/settings/tokens)，只勾选 repo 的权限并生成一个新的 Token。~~
6. ~~回到 [Travis CI](https://github.com/marketplace/travis-ci)，前往你的 repository 的设置页面，在 Environment Variables 下新建一个环境变量，Name 为 `GH_TOKEN`，Value 为刚才你在 GitHub 生成的 Token。确保 DISPLAY VALUE IN BUILD LOG 保持 不被勾选 避免你的 Token 泄漏。点击 Add 保存。~~
7. ~~在你的 Hexo 站点文件夹中新建一个 .travis.yml 文件：~~

```yaml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
```

8. ~~将 .travis.yml 推送到 repository 中。Travis CI 应该会自动开始运行，并将生成的文件推送到同一 repository 下的 gh-pages 分支下~~
9. ~~在 GitHub 中前往你的 repository 的设置页面，修改 GitHub Pages 的部署分支为 gh-pages。~~