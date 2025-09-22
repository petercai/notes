# HomeBrew 从安装到精通


[介绍](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E4%25BB%258B%25E7%25BB%258D "https://www.codermast.com/dev-tools/homebrew.html#%E4%BB%8B%E7%BB%8D")
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

官方描述

Homebrew is the easiest and most flexible way to install the UNIX tools Apple didn’t include with macOS. It can also install software not packaged for your Linux distribution without requiring sudo.

Homebrew 是安装苹果没有包含在 macOS 中的 UNIX 工具的最简单、最灵活的方式。它还可以安装不适合您的 Linux 发行版的软件，而不需要 sudo。

使用 Homebrew 可以安装 Mac（或Linux）没有预装但你需要的东西。

总的来说，Homebrew 是一款在 UNIX 平台下的软件安装管理器，又或者包管理器。

[安装](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%25AE%2589%25E8%25A3%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E5%AE%89%E8%A3%85")
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### [官方安装](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%25AE%2598%25E6%2596%25B9%25E5%25AE%2589%25E8%25A3%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E5%AE%98%E6%96%B9%E5%AE%89%E8%A3%85")

官方地址

*   官网：[brew.sh/open in new…](https://link.juejin.cn/?target=https%3A%2F%2Fbrew.sh%2F "https://brew.sh/")
    
*   中文：[brew.sh/zh-cn/open …](https://link.juejin.cn/?target=https%3A%2F%2Fbrew.sh%2Fzh-cn%2F "https://brew.sh/zh-cn/")
    
*   安装指令
    

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

```

### [国内镜像](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%259B%25BD%25E5%2586%2585%25E9%2595%259C%25E5%2583%258F "https://www.codermast.com/dev-tools/homebrew.html#%E5%9B%BD%E5%86%85%E9%95%9C%E5%83%8F")

*   安装指令

```sh
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"

```

*   卸载指令

```sh
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"

```

*   Gitee 仓库：[gitee.com/cunkai/Home…](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fcunkai%2FHomebrewCN "https://gitee.com/cunkai/HomebrewCN")

[常用指令](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%25B8%25B8%25E7%2594%25A8%25E6%258C%2587%25E4%25BB%25A4 "https://www.codermast.com/dev-tools/homebrew.html#%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A4")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### [更新brew](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259B%25B4%25E6%2596%25B0brew "https://www.codermast.com/dev-tools/homebrew.html#%E6%9B%B4%E6%96%B0brew")

定期更新 Brew 可以确保您拥有最新的软件包和版本

```sh
brew update 

```

### [搜索软件包](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%2590%259C%25E7%25B4%25A2%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E6%90%9C%E7%B4%A2%E8%BD%AF%E4%BB%B6%E5%8C%85")

```sh
brew search package-name

```

*   package-name：你想要搜索的软件包名

例子

举个例子，如果你想要搜索 Node.js 那么你就可以执行

```sh
brew search node

```

![](_assets/de33c2a4bb1e4d56bc01603ee9e06fd3~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### [安装软件包](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%25AE%2589%25E8%25A3%2585%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E5%AE%89%E8%A3%85%E8%BD%AF%E4%BB%B6%E5%8C%85")

```go
brew install package-name

```

*   package-name : 你想安装的软件包名

例子

举个例子，如果你想要安装 Node.js，并且你知道他在 brew 中的软件包名，那么你可以直接使用该指令安装，如果你不知道的话，可以先试用 `brew search` 指令进行查找，然后再进行安装。

```null
brew install node

```

![](_assets/5b30744bf94f4c278f36c0b0ec5d77b1~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### [查看已安装的软件包](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B%25E5%25B7%25B2%25E5%25AE%2589%25E8%25A3%2585%25E7%259A%2584%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B%E5%B7%B2%E5%AE%89%E8%A3%85%E7%9A%84%E8%BD%AF%E4%BB%B6%E5%8C%85")

```null
brew list

```

![](_assets/3486be60c7b04af989a24ba947173e59~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### [卸载软件包](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%258D%25B8%25E8%25BD%25BD%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E5%8D%B8%E8%BD%BD%E8%BD%AF%E4%BB%B6%E5%8C%85")

```go
brew uninstall package-name

```

*   package-name：已安装的软件包名

例子

卸载掉我们前面安装的 Node.js，可以使用

```null
brew uninstall node

```

![](_assets/f9562484ccf8417fbf9fac2caf1d3032~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### [查看软件包信息](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585%25E4%25BF%25A1%25E6%2581%25AF "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B%E8%BD%AF%E4%BB%B6%E5%8C%85%E4%BF%A1%E6%81%AF")

使用 info 命令可以查看有关软件包的详细信息，包括其依赖项和链接

```go
brew info package-name

```

*   package-name：要查看的软件包名

例子

前面我们卸载了 Node.js 这里就不以 Node.js 为例了，查看 MySQL 的信息。

```null
brew info mysql

```

![](_assets/a366505382da48fcba8deeb2111c4032~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### [清理过期的软件包](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%25B8%2585%25E7%2590%2586%25E8%25BF%2587%25E6%259C%259F%25E7%259A%2584%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E6%B8%85%E7%90%86%E8%BF%87%E6%9C%9F%E7%9A%84%E8%BD%AF%E4%BB%B6%E5%8C%85")

定期清理不再需要的旧版本和无用的库文件可以释放磁盘空间：

```null
brew cheanup

```

### [显示 Brew 版本信息](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%2598%25BE%25E7%25A4%25BA-brew-%25E7%2589%2588%25E6%259C%25AC%25E4%25BF%25A1%25E6%2581%25AF "https://www.codermast.com/dev-tools/homebrew.html#%E6%98%BE%E7%A4%BA-brew-%E7%89%88%E6%9C%AC%E4%BF%A1%E6%81%AF")

查看 homebrew 的版本信息，可以运行

```css
brew --version

```

![](_assets/fbab4fd3b3ec4858a5621497d69d6120~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

提示

查看一个软件包是否成功安装或成功配置的简单方式就是看能否查看其版本信息。

### [列出过时的软件包](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%2588%2597%25E5%2587%25BA%25E8%25BF%2587%25E6%2597%25B6%25E7%259A%2584%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E5%88%97%E5%87%BA%E8%BF%87%E6%97%B6%E7%9A%84%E8%BD%AF%E4%BB%B6%E5%8C%85")

```null
brew outdated

```

### [更新软件包](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259B%25B4%25E6%2596%25B0%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E6%9B%B4%E6%96%B0%E8%BD%AF%E4%BB%B6%E5%8C%85")

```css
brew upgrade [package-name]

```

*   package-name：软件包名称，可选，未填写默认为所有软件包。

### [安装 Cask 扩展](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%25AE%2589%25E8%25A3%2585-cask-%25E6%2589%25A9%25E5%25B1%2595 "https://www.codermast.com/dev-tools/homebrew.html#%E5%AE%89%E8%A3%85-cask-%E6%89%A9%E5%B1%95")

Brew Cask 是 Brew 的扩展，用于安装和管理 macOS 应用程序。您可以使用以下命令安装 Cask 扩展：

```bash
brew tap homebrew/cask

```

### [安装应用程序](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%25AE%2589%25E8%25A3%2585%25E5%25BA%2594%25E7%2594%25A8%25E7%25A8%258B%25E5%25BA%258F "https://www.codermast.com/dev-tools/homebrew.html#%E5%AE%89%E8%A3%85%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F")

使用 Brew Cask 可以轻松安装 macOS 应用程序。例如，要安装 Visual Studio Code：

```css
brew install --cask visual-studio-code

```

### [查看软件包的依赖关系](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585%25E7%259A%2584%25E4%25BE%259D%25E8%25B5%2596%25E5%2585%25B3%25E7%25B3%25BB "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%9A%84%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB")

要查看软件包的依赖关系，可以使用 deps 命令。这将列出软件包所依赖的其他软件包。例如：

```null
brew deps package_name

```

### [查看软件包的可选依赖关系](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585%25E7%259A%2584%25E5%258F%25AF%25E9%2580%2589%25E4%25BE%259D%25E8%25B5%2596%25E5%2585%25B3%25E7%25B3%25BB "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%9A%84%E5%8F%AF%E9%80%89%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB")

有些软件包具有可选的依赖关系，可以使用 options 命令查看这些选项。例如：

```null
brew options package_name

```

### [查看已安装软件包的版本历史](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B%25E5%25B7%25B2%25E5%25AE%2589%25E8%25A3%2585%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585%25E7%259A%2584%25E7%2589%2588%25E6%259C%25AC%25E5%258E%2586%25E5%258F%25B2 "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B%E5%B7%B2%E5%AE%89%E8%A3%85%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%9A%84%E7%89%88%E6%9C%AC%E5%8E%86%E5%8F%B2")

使用 versions 命令可以查看已安装软件包的版本历史。这将列出所有可用版本以及它们的安装状态。例如：

```null
brew versions package_name

```

### [使用 Brew Cask 安装 GUI 应用程序](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E4%25BD%25BF%25E7%2594%25A8-brew-cask-%25E5%25AE%2589%25E8%25A3%2585-gui-%25E5%25BA%2594%25E7%2594%25A8%25E7%25A8%258B%25E5%25BA%258F "https://www.codermast.com/dev-tools/homebrew.html#%E4%BD%BF%E7%94%A8-brew-cask-%E5%AE%89%E8%A3%85-gui-%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F")

使用 Brew Cask 可以轻松安装 macOS GUI 应用程序。例如，要安装 Google Chrome：

```css
brew install --cask google-chrome

```

### [查看 Brew 配置信息](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B-brew-%25E9%2585%258D%25E7%25BD%25AE%25E4%25BF%25A1%25E6%2581%25AF "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B-brew-%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF")

要查看当前 Brew 配置信息，包括仓库和版本库的位置，可以使用 config 命令：

```arduino
brew config

```

### [查看 Brew 更新日志](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B-brew-%25E6%259B%25B4%25E6%2596%25B0%25E6%2597%25A5%25E5%25BF%2597 "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B-brew-%E6%9B%B4%E6%96%B0%E6%97%A5%E5%BF%97")

要查看 Brew 的更新日志，可以使用 log 命令：

brew log package_name

### [查看软件包的安装路径](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B%25E8%25BD%25AF%25E4%25BB%25B6%25E5%258C%2585%25E7%259A%2584%25E5%25AE%2589%25E8%25A3%2585%25E8%25B7%25AF%25E5%25BE%2584 "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%9A%84%E5%AE%89%E8%A3%85%E8%B7%AF%E5%BE%84")

使用 ls 命令可以查看特定软件包的安装路径。例如：

```bash
brew ls --full package_name

```

### [查看本地镜像源](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B%25E6%259C%25AC%25E5%259C%25B0%25E9%2595%259C%25E5%2583%258F%25E6%25BA%2590 "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B%E6%9C%AC%E5%9C%B0%E9%95%9C%E5%83%8F%E6%BA%90")

```bash
cd "$(brew --repo)" && git remote -v

```

### [查看哪些包可以更新](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259F%25A5%25E7%259C%258B%25E5%2593%25AA%25E4%25BA%259B%25E5%258C%2585%25E5%258F%25AF%25E4%25BB%25A5%25E6%259B%25B4%25E6%2596%25B0 "https://www.codermast.com/dev-tools/homebrew.html#%E6%9F%A5%E7%9C%8B%E5%93%AA%E4%BA%9B%E5%8C%85%E5%8F%AF%E4%BB%A5%E6%9B%B4%E6%96%B0")

```null
brew outdated

```

### [更新包 Homebrew 会安装新版本的包，但旧版本依然会保留](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%259B%25B4%25E6%2596%25B0%25E5%258C%2585-homebrew-%25E4%25BC%259A%25E5%25AE%2589%25E8%25A3%2585%25E6%2596%25B0%25E7%2589%2588%25E6%259C%25AC%25E7%259A%2584%25E5%258C%2585-%25E4%25BD%2586%25E6%2597%25A7%25E7%2589%2588%25E6%259C%25AC%25E4%25BE%259D%25E7%2584%25B6%25E4%25BC%259A%25E4%25BF%259D%25E7%2595%2599 "https://www.codermast.com/dev-tools/homebrew.html#%E6%9B%B4%E6%96%B0%E5%8C%85-homebrew-%E4%BC%9A%E5%AE%89%E8%A3%85%E6%96%B0%E7%89%88%E6%9C%AC%E7%9A%84%E5%8C%85-%E4%BD%86%E6%97%A7%E7%89%88%E6%9C%AC%E4%BE%9D%E7%84%B6%E4%BC%9A%E4%BF%9D%E7%95%99")

```css
brew upgrade [FORMULA|CASK...]

```

### [清理旧版本和缓存](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E6%25B8%2585%25E7%2590%2586%25E6%2597%25A7%25E7%2589%2588%25E6%259C%25AC%25E5%2592%258C%25E7%25BC%2593%25E5%25AD%2598 "https://www.codermast.com/dev-tools/homebrew.html#%E6%B8%85%E7%90%86%E6%97%A7%E7%89%88%E6%9C%AC%E5%92%8C%E7%BC%93%E5%AD%98")

brew cleanup # 清理所有包的旧版本  
brew cleanup \[FORMULA ...\] # 清理指定包的旧版本  
brew cleanup -n # 查看可清理的旧版本包，不执行实际操作

### [锁定不想更新的包](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E9%2594%2581%25E5%25AE%259A%25E4%25B8%258D%25E6%2583%25B3%25E6%259B%25B4%25E6%2596%25B0%25E7%259A%2584%25E5%258C%2585 "https://www.codermast.com/dev-tools/homebrew.html#%E9%94%81%E5%AE%9A%E4%B8%8D%E6%83%B3%E6%9B%B4%E6%96%B0%E7%9A%84%E5%8C%85")

```ini
brew pin [FORMULA ...]      
brew unpin [FORMULA ...]    

```

tips：因为update会一次更新所有的包的，当我们想忽略的时候可以使用这个命令

### [软件服务管理](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E8%25BD%25AF%25E4%25BB%25B6%25E6%259C%258D%25E5%258A%25A1%25E7%25AE%25A1%25E7%2590%2586 "https://www.codermast.com/dev-tools/homebrew.html#%E8%BD%AF%E4%BB%B6%E6%9C%8D%E5%8A%A1%E7%AE%A1%E7%90%86")

```python
brew services list  				
brew services run formula|--all  	
brew services start formula|--all  	
brew services stop formula|--all   	
brew services restart formula|--all 

```

### [切换镜像源](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%2588%2587%25E6%258D%25A2%25E9%2595%259C%25E5%2583%258F%25E6%25BA%2590 "https://www.codermast.com/dev-tools/homebrew.html#%E5%88%87%E6%8D%A2%E9%95%9C%E5%83%8F%E6%BA%90")

切换镜像源有三个库要切换：

*   brew.git
*   homebrew-core.git
*   homebrew-bottles

1.  中科大源

```shell
# 替换brew.git:
$ cd "$(brew --repo)"
$ git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 替换homebrew-core.git:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# 替换homebrew-bottles:
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile

# 应用生效:
$ brew update

```

2.  清华大学源

```shell
# 替换brew.git:
$ cd "$(brew --repo)"
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

# 替换homebrew-core.git:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

# 替换homebrew-bottles:
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile

# 应用生效:
$ brew update

```

[参考资料](https://link.juejin.cn/?target=https%3A%2F%2Fwww.codermast.com%2Fdev-tools%2Fhomebrew.html%23%25E5%258F%2582%25E8%2580%2583%25E8%25B5%2584%25E6%2596%2599 "https://www.codermast.com/dev-tools/homebrew.html#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

*   Homebrew官方文档：[docs.brew.sh/open in new…](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.brew.sh%2F "https://docs.brew.sh/")
*   知乎文章：[zhuanlan.zhihu.com/p/691010258…](https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F691010258 "https://zhuanlan.zhihu.com/p/691010258")
*   博客园文章：[www.cnblogs.com/FuYingju/p/…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2FFuYingju%2Fp%2F14342924.html "https://www.cnblogs.com/FuYingju/p/14342924.html")