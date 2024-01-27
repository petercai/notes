# 利用 github action 生成 docsify 的文件目录
前言
--

[docsify](https://link.juejin.cn/?target=https%3A%2F%2Fdocsify.js.org%2F%23%2Fzh-cn%2F%3Fid%3Ddocsify "https://docsify.js.org/#/zh-cn/?id=docsify") 可以快速帮你生成文档网站。不同于 GitBook、Hexo 的地方是它不会生成静态的 `.html` 文件，所有转换工作都是在运行时。

根据 `docsify` 官网的[快速开始](https://link.juejin.cn/?target=https%3A%2F%2Fdocsify.js.org%2F%23%2Fzh-cn%2Fquickstart "https://docsify.js.org/#/zh-cn/quickstart")可以很快地就搭建一个个人博客。 搭配 [github pages](https://link.juejin.cn/?target=https%3A%2F%2Fpages.github.com "https://pages.github.com") 就可以拥有一个公网可访问的静态博客。

网上关于如何使用`docsify`和`github pages`的文章网上已经有挺多的了，这里就不做详细介绍。这里主要是介绍如何使用`github action`来生成文件目录。因为`docsify`本身是一个静态网页，只需要生成一次`index.html`就可以运行，所以本身是不支持动态去显示文件目录。但是`docsify`支持自定义[侧边栏](https://link.juejin.cn/?target=https%3A%2F%2Fdocsify.js.org%2F%23%2Fzh-cn%2Fmore-pages "https://docsify.js.org/#/zh-cn/more-pages")，只需要在根目录增加一个`_sidebar.md`和打开loadSidebar选项，侧边栏就会自动显示`_sidebar.md`的内容。

我们只需要在`_sidebar.md`加上目录信息， 再搭配[docsify-sidebar-collapse](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FiPeng6%2Fdocsify-sidebar-collapse "https://github.com/iPeng6/docsify-sidebar-collapse")插件就可以显示以下的效果了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b88ec9d99974440ca64cdf65d0a8fb50~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=195&h=269&s=13709&e=png&b=fefefe)

原理
--

> GitHub Actions 是 GitHub 提供的一种自动化工具，它可以帮助你在 GitHub 仓库中自动执行软件开发的工作流程。你可以使用 GitHub Actions 来构建、测试和部署你的代码。你也可以使用它来自动化其他的任务，例如发送电子邮件通知、创建问题和拉取请求，甚至发布到 GitHub Pages 或其他的云平台。
> 
> GitHub Actions 的工作流程是由一系列的任务（称为"actions"）组成的，这些任务可以在同一个虚拟环境中按照你定义的顺序执行。每个任务都可以运行一个命令或者一个脚本，也可以运行一个你从 GitHub Marketplace 或者其他地方获取的预定义的action。
> 
> 你可以在你的仓库中创建一个 `.github/workflows` 目录来存放你的工作流程文件。每个工作流程文件都是一个 YAML 文件，它定义了一个或者多个工作流程。当你的仓库中发生一个特定的事件（例如 push、pull request 或者 issue）时，GitHub Actions 就会自动执行相应的工作流程。

如果使用`github pages`来部署我们的博客，就需要一个github仓库，那这样我们只需要利用`github action`在代码推送时，都生成一次`_sidebar.md`，这样就可以保证我们仓库上面的目录一直都是最新的。

`_sidebar.md`:

```md
- [README.md](./README.md)
- directory1
  - [file1.1.md](./directory1/file1.1.md)
  - [file1.md](./directory1/file1.md)
- directory2
  - [file2.md](./directory2/file2.md)
- directory3
  - [file3.md](./directory3/file3.md)
- [_sidebar.md](./_sidebar.md)

```

教程
--

在仓库目录建立一个新文件`.github/workflows/main.yml`

```yaml
on: [push] 

jobs:
  add_sidebar_job:
    runs-on: ubuntu-latest
    name: job to add _sidebar.md
    steps:
      - uses: actions/checkout@v4                     
      - uses: if-nil/docsify-file-catalog-action@main 
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}   
          

```

`docsify-file-catalog-action`是一个用来生成_sidebar.md并推送到仓库的action，有两个参数

| inputs | 描述 | 必须 | 默认值 |
| --- | --- | --- | --- |
| github_token | 用来推送`_sidebar.md`到仓库内，需要写权限 | true | null |
| include | 希望显示的文件列表（正则表达式） | false | `.*\.md` |

需要检查一下`github_token`是否有写权限，位置在仓库的 Setting -> Actions -> General:

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e0fad6f4e194d8abd6a6b4128f1417c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1850&h=118&s=24228&e=png&b=f5f7f9)
 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/310b16cd2e464a90aceacf67402a08cc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2120&h=762&s=137506&e=png&b=ffffff)

然后提交代码即可

```shell
git add .github/workflows/main.yml
git commit -m "add workflow"
git push

```

点击仓库的Action按钮查看执行情况

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e88e19d6bab4a9190f268b600157276~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2940&h=1496&s=169336&e=png&b=24292e)

一切正常的话就可以看到我们的根目录有一个`_sidebar.md`文件了

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96cbf96eacdd479cbfec001ebc85b40f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1898&h=726&s=110019&e=png&b=ffffff)

参考
--

*   [docsify-file-catalog-action](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fif-nil%2Fdocsify-file-catalog-action "https://github.com/if-nil/docsify-file-catalog-action")
*   [actions-demo](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fif-nil%2Factions-demo "https://github.com/if-nil/actions-demo")