# NeoVim vs. Vim
在 Linux 系统中，编辑器是开发和系统管理的必备工具。而在众多编辑器中，Vim 作为一款经典的文本编辑器，一直备受欢迎。然而，随着时间的推移，NeoVim 的出现逐渐成为了 Linux 运维人员的首选。那么，为什么 Linux 运维更喜欢 NeoVim 而不是 Vim 呢？本文将探讨这个问题。

![](https://static001.geekbang.org/infoq/01/016f599a3f03b54173da185210710d66.png)

1\. 改进了 Vim 的一些设计缺陷
-------------------

尽管 Vim 作为一款经典的文本编辑器备受青睐，但是它仍然存在一些设计缺陷。例如，Vim 缺乏异步处理、API 设计和插件管理等方面的支持，这些限制了其在一些特定的场景下的使用。

NeoVim 在这些方面进行了改进。它引入了异步处理和更好的插件管理功能，这意味着插件的开发和使用将更加容易。同时，它还重新设计了 API，使得插件开发者可以更轻松地与 NeoVim 进行交互。这些改进让 NeoVim 可以更好地满足 Linux 运维人员的需求。

2\. 提供更好的 Lua 脚本支持
------------------

Lua 是一种轻量级的脚本语言，它的高效和灵活性使得它成为许多编辑器和游戏开发者的首选语言之一。NeoVim 引入了 Lua 脚本支持，使得插件的开发和使用变得更加容易和高效。使用 Lua 编写插件可以极大地提高插件的开发效率，让 Linux 运维人员可以更快地完成任务。

3\. 改善了终端支持
-----------

Linux 运维人员通常需要在终端中进行编辑器操作。NeoVim 改善了对终端的支持，支持 24 位真彩色、异步输入等功能，这让它在终端中的使用体验更加出色。此外，NeoVim 还提供了一个内置的终端模式，让用户可以在编辑器内部使用终端，这大大提高了工作效率。

4\. 改善了多语言支持
------------

在 Linux 系统管理和开发中，通常需要使用多种编程语言。NeoVim 通过提供更好的多语言支持来支持更多的编程语言，并提供更好的代码补全和语法高亮等功能。这使得它更适合于在 Linux 系统管理和开发中进行编码和调试。

5\. 具有更好的性能
-----------

性能是编辑器的重要指标之一。NeoVim 比 Vim 更快，并可以在异步处理时保持更低的延迟。在 Linux 运维人员需要处理大量数据的情况下，更好的性能可以大大提高工作效率。

6\. 更好的社区支持
-----------

NeoVim 有一个活跃的社区支持，提供了丰富的插件和主题等资源，使得用户可以更好地定制自己的编辑器环境。此外，NeoVim 的社区也致力于不断改进和更新 NeoVim，使得它保持着最新的功能和技术。

7\. 易于配置和扩展
-----------

NeoVim 的配置和扩展都比 Vim 更加灵活和方便。NeoVim 通过提供更多的配置选项和插件支持来满足用户的不同需求。此外，NeoVim 还提供了一个内置的包管理器，使得插件的安装和更新更加简单。

8\. 更好的文档支持
-----------

文档是使用编辑器的重要组成部分。NeoVim 提供了更好的文档支持，可以通过内置的文档浏览器或浏览器插件查看相关文档。此外，NeoVim 的文档也更加详细和全面，对于新手用户更加友好。

NeoVim 和 Vim 的常用命令对比
--------------------

NeoVim 和 Vim 的许多常用命令是相同的，但是 NeoVim 也提供了一些新的命令和改进。下面是一些常用命令的对比：

### 1\. 打开文件

NeoVim 和 Vim 的打开文件命令都是 `:e`，但是 NeoVim 的 `:edit` 命令提供了更多的选项，如在水平或垂直分割窗口中打开文件、在当前文件的目录中打开文件等。

```
NeoVim: :e filename.txt
Vim: :e filename.txt
```

### 2\. 保存文件

NeoVim 和 Vim 的保存文件命令都是 `:w`，但是 NeoVim 的 `:write` 命令提供了更多的选项，如将文件保存为另一个文件、在保存时自动格式化等。

### 3\. 退出编辑器

NeoVim 和 Vim 的退出编辑器命令都是 `:q`，但是 NeoVim 的 `:quit` 命令提供了更多的选项，如强制退出编辑器、在编辑器中打开多个文件时只退出当前文件等。

### 4\. 复制和粘贴

NeoVim 和 Vim 的复制和粘贴命令都是类似的，但是 NeoVim 通过添加一些额外的命令和选项，使得复制和粘贴更加灵活和方便。

```
NeoVim: yy（复制当前行）、yiw（复制当前单词）、y$（复制当前光标到行尾的内容）
Vim: yy（复制当前行）、yiw（复制当前单词）、y$（复制当前光标到行尾的内容）
```

```
NeoVim: p（粘贴到当前光标后面）、P（粘贴到当前光标前面）、"[register]p（从指定寄存器中粘贴）
Vim: p（粘贴到当前光标后面）、P（粘贴到当前光标前面）、"[register]p（从指定寄存器中粘贴）
```

### 5\. 搜索和替换

NeoVim 和 Vim 的搜索和替换命令也非常相似，但是 NeoVim 通过添加一些额外的命令和选项，使得搜索和替换更加灵活和强大。

```
NeoVim: /pattern（向前搜索指定模式）、?pattern（向后搜索指定模式）、:%s/old/new/g（全局替换）
Vim: /pattern（向前搜索指定模式）、?pattern（向后搜索指定模式）、:%s/old/new/g（全局替换）
```

结论
--

在 Linux 运维中，使用一个高效的编辑器是至关重要的，因为它能够提高工作效率和代码质量。尽管 Vim 一直是 Linux 运维人员使用的常见编辑器之一，但现在 NeoVim 作为 Vim 的升级版已经受到越来越多的关注和使用。NeoVim 提供了更多的功能和特性，改善了 Vim 的一些设计缺陷，并提供了更好的性能和社区支持。此外，NeoVim 还提供了更多的命令和选项，使得编辑更加灵活和方便。因此，如果你是一个 Linux 运维人员，你应该尝试使用 NeoVim，看看它是否能够帮助你提高工作效率和代码质量。