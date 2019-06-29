title: "使用 Travis 自动化部署 Hexo Blog"
date: 2019-06-28 22:30:00
tag:
- hexo
- travis
- CI
permalink: hexo-with-travis-ci

---

## 0x0 背景

使用 Hexo + Github Pages 搭建博客后，每次更新文章需要使用 `hexo d -g` 会在本地生成 `public` 静态博客网站和向 Github 推送的 `.deploy_git` 文件夹。`.deploy_git` 文件夹内容和 `public` 文件夹一致，但多了 Github 博客项目的仓库信息与提交信息。最终，`.deploy_git` 文件夹内的全部内容被推送到 Github 仓库中，由 Github Pages 服务完成静态网站的解析。

当切换工作环境后，需要重新安装 Nodejs 以及配置 Hexo 和它的依赖。而且每次更新文章后，都要 `hexo d -g` 手动部署。这样多次重复的工作非常低效，因此结合现在非常流行的 CI/CD 概念和工具，可以为 Hexo + Github Pages 博客集成 Travis CI 自动部署的能力。当推送博客仓库到 Github 后，由 Travis 自动获取当前 commit 并进行构建，把生成的静态网站推送到 Github Pages 分支。

## 0x1 理解 Hexo 的自动化部署

下图是 Hexo 手动部署的流程，hexo-blog 可以是本地一个项目，也可以是 Github、Gitlab 等仓库，本地配置好 Hexo 环境后，由 ① 触发部署，将本地生成的静态博客网站 `.delpoy_git` 推送到 Github 的静态博客仓库中。

![hexo-without-travis](https://wx2.sinaimg.cn/large/8656dc57ly1g4hvvvoi0wj20mz0n9q3l.jpg)

当引入 Travis 后，整个流程变成了下图所示的流程。hexo-blog 项目必须是一个 Github 仓库了，当有文章更新，本地由 ① 触发，把 Hexo-blog 的源码推送到 Github，剩下的工作由 Travis 完成：获取 Github hexo-blog 仓库中最新的 commit，运行我们定义的 `.travis.yml` 并把生成的静态博客网站 `.deploy_git` 推送 Github 静态博客仓库 xxx.github.io。

![hexo-without-travis](https://wx3.sinaimg.cn/large/8656dc57ly1g4hvvvqn78j213s0oldh5.jpg)

> 注意，这里要区分 Github 中的两个仓库：静态 blog repo 和 Hexo blog repo。前者是博客网站的静态网站项目，由 Github Pages 托管和解析；后者是 Hexo 项目，前者的内容是由后者生成的。

## 0x2 如何配置自动化部署

再看一次引入 Travis 后的流程图，绿色箭头的流程是 Travis 自动运行的。要想实现自动化部署，需要 Travis：

1. 能够自动获取 hexo-blog 仓库的代码提交；
2. 能够从 hexo-blog 仓库的源码中生成静态博客网站 .deploy_git 目录；
3. 能够把静态博客网站目录文件推送到静态博客仓库 xxx.github.io 中；


接下来，我们用三个步骤，分别解决上面提到的三个问题。

### 1. 配置 Travis 获取 Hexo blog

在 https://travis-ci.org/account/repositories 中勾选你的 hexo-blog 项目仓库，注意，这里不是静态博客项目而是 Hexo blog 的源码仓库。

这样，每当 hexo-blog 有新的 commit 时，Travis 都能收到 Gitub 的回调，并从 hexo-blog 仓库中拉取最新提交的代码。

### 2. 配置 Travis 自动编译

Travis 的自动编译是由项目根目录的 `.travis.yml` 来控制的。我们需要为 hexo-blog 项目编写一个 `.travis.yml` 文件，下面是一个 `.travis.yml` 示例。

```yaml
language: node_js
node_js:
  - '10'

branches:
  only:
  - master

before_install:
- git config --global user.name "YOUR NAME"
- git config --global user.email "YOUR EMAIL"
- sed -i'' "s~https://github.com/<YOUR NAME>/<YOUR BLOG REPO>.git~https://${ACCESS_TOKEN}@github.com/<YOUR NAME>/<YOUR REPO>.git~" _config.yml

install:
- npm install hexo-cli -g
- npm install

script:
- npm run deploy
```

其中：

* `language` 和 `node_js` 指定了使用 NodeJS docker 进行构建，NodeJS 版本为 10.x；

* `branches` 指定只有 master 分支的提交才进行构建；

* `before_install` 用于配置推送静态博客网站项目的 git 信息，最重要的是 `sed -i` 这条命令，该命令把 hexo-blog 配置文件 `_config.yml` 中的 git remote URL 替换为携带认证 `ACCESS TOKEN` 的 URL。  这样避免了 Travis 中需要输入用户名密码的问题。关于 `${ACCESS_TOKEN}` 变量会在接下来进行说明；

  ```yaml
  # _config.yml
  ...
  deploy:
    type: git
    repo: https://github.com/<YOUR NAME>/<YOUR REPO>.git
    branch: master
  ...
  ```

* `install` 指定了每次构建之前需要先安装 hexo 和 hexo-blog 项目中的依赖；

* `script` 指定了运行的 npm deploy 命令去执行 `hexo d -g`。deploy 命令配置在了 hexo-blog 项目的 package.json 中：

  ```json
  {
    ...
    "scripts": {
      "deploy": "hexo clean && hexo d -g"
    },
    ...
  }
  ```

### 3. 配置 Travis 推送博客仓库的权限

Travis 需要把生成的静态博客网站推送到 Github 博客网站仓库，需要有仓库的读写权限，有两个办法为 Travis 配置权限：

* 一个是把 Github SSH Key 放到 Hexo-blog 项目中，Travis 构建完成推送时，使用 SSH Key 推送；
* 另一种办法是在 Github 中为 Travis 生成一个拥有读写仓库权限的 Personal access tokens。上面的 .travis.yml 中使用到的 `${ACCESS_TOKEN}` 变量便是第二种方法。

关于第一种使用 Github SSH Key 的方法，因为 Hexo-blog 是公开项目，所以直接把 SSH Key 放到项目里是不安全的，因此需要使用 Travis 先把 SSH Key 加密，并在 .travis.yml 中配置解密。该方法配置稍微复杂一点，这里介绍更简单的 Personal access tokens 方法。

1. 登录静态网站项目的 Github 账号，在 https://github.com/settings/tokens 中生成新的 token，并勾选 repo 权限。

  ![github-access-token](https://wx2.sinaimg.cn/large/8656dc57ly1g4hyzt6lkzj20yh0jw0tk.jpg)

  生成的 token 一定要复制保存下来。

2. 在 https://travis-ci.org/account/repositories 中点击 hexo-blog 项目的 settings，添加环境变量，把上面僧成的 token 设置为变量，这里的变量名字设置为 `ACCESS_TOKEN` ，和 .travis.yml 中的配置一致。注意，为了安全不要勾选 `DISPLAY VALUE IN BUILD LOG`。

  ![travis-env-var](https://wx4.sinaimg.cn/large/8656dc57ly1g4hzakh6ksj20sz0b6q39.jpg)

至此，自动化部署已经配置好了。

在 Hexo blog 项目中新建一个 git commit，可以在 Travis 中查看项目构建的过程了，不管成功还是失败，你的邮箱都会收到一封构建邮件。

## 0x3 后记

不论是本文的 Travis 配置，还是其他新鲜的知识，只有理解了技术的原理，才能在遇到问题的时候知道怎么解决，看到别人的解决方法时知道他的思路和为什么要这么做，是否还有更优的方法。

最近一年基本没有新的文章，倒是时不时更新一下之前的文章。有些东西已经过时了，但依然保留在了这个博客中。这里，不仅仅是记录和分享，也见证了自己点滴的成长。