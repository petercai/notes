# 技术|使用 dnf 进行 Linux 包管理
作者： [Seth Kenlon](https://opensource.com/article/21/6/dnf-linux) 译者： [LCTT](https://linux.cn/lctt/) [Hans zhao](https://linux.cn/lctt/hanszhao80)

| 2022-05-04 10:15   评论: [](https://linux.cn/portal.php?mod=comment&id=14542&idtype=aid "查看全部评论") 收藏: _1_    

> 了解如何在 Linux 上使用 `dnf` 命令安装软件包，然后下载我们的速查表，让正确的命令触手可及。

在计算机系统上安装应用程序非常简单：就是将档案（如 `.zip` 文件）中的文件复制到目标计算机上，放在操作系统预期放应用程序的位置。因为我们中的许多人习惯于使用花哨的安装“向导”来帮助我们在计算机上安装软件，所以这个过程似乎在技术上应该比实际更复杂。

然而，复杂的是，是什么构成了一个程序？用户认为的单个应用程序实际上包含了分散在操作系统中的软件库的各种依赖代码（例如：Linux 上的 .so 文件、Windows 上的 .dll 文件和 macOS 上的 .dylib 文件）。

为了让用户不必担心这些程序代码之间的复杂的互相依赖关系， Linux 使用 包管理系统package management system 来跟踪哪些应用程序需要哪些库，哪些库或应用程序有安全或功能更新，以及每个软件会附带安装哪些额外的数据文件。包管理器本质上是一个安装向导。它们易于使用，提供了图形界面和基于终端的界面，让你的生活更轻松。你越了解你的发行版的包管理器，你的生活就会越轻松。

如果你在使用 Linux 桌面时，偶尔想要安装一个应用程序，那么你可能正在寻找 [GNOME “软件”](https://wiki.gnome.org/Apps/Software)，它是一个桌面应用程序浏览器。

![](_assets/101540ua7z7xaipq5pq17z.png)

_GNOME “软件” 程序_

它会按你的预期工作：点击它的界面，直到你找到一个看起来有用的应用程序，然后单击 “安装” 按钮。

或者，你可以在 GNOME “软件” 中打开从网络下载的 `.rpm` 或 `.flatpakref` 软件包，以便它进行安装。

但如果你更倾向于使用命令行，请继续阅读。

在安装应用程序之前，你可能需要确认它是否存在于你的发行版的服务器上。通常，使用 `dnf` 搜索应用程序的通用名称就足够了。例如，假设你最近阅读了 [一篇关于 Cockpit 的文章](https://opensource.com/article/20/11/cockpit-server-management)，并决定尝试一下。你可以搜索 `cockpit` 验证该发行版是否包含它：

```


1.  `$ dnf search cockpit`
2.   `Last metadata expiration check:  0:01:46 ago on Tue  18  May  2021  19:18:15 NZST.`
3.   `====  Name  Exactly  Matched: cockpit ====`
4.   `cockpit.x86_64 :  Web  Console  for  Linux servers`

6.  `====  Name  &  Summary  Matched: cockpit ==`
7.   `cockpit-bridge.x86_64 :  Cockpit bridge server-side component`
8.   `cockpit-composer.noarch :  Composer GUI for  use  with  Cockpit`
9.   `[...]`


```

有一个精确的匹配。上面列出的匹配的软件包名为 `cockpit.x86_64`，但名称中的 `.x86_64` 部分仅表示它兼容该 CPU 架构。默认情况下，你的系统会安装适配当前 CPU 架构的软件包，因此你可以忽略该扩展名。所以你确认你要查找的软件包确实简称为 `cockpit`。

现在你可以放心地使用 `dnf install` 安装它。 此步骤需要管理员权限：

```


1.  `$ sudo dnf install cockpit`


```

一般来说，这就是典型的 `dnf` 工作流：搜索并安装。

然而，有时 `dnf search` 的结果并不清晰，或者你想要关于一个软件包的更多信息，而不仅仅是它的通用名称。有一些相关的 `dnf` 子命令，具体取决于你想要的信息。

如果你觉得你的搜索已 _接近_ 想要的结果，但还不确定，查看软件包的元数据通常会有所帮助，例如项目的网址和描述。要获取此信息，请使用顾名思义的 `dnf info` 命令：

```


1.  `$ dnf info terminator`
2.  `Available  Packages`
3.  `Name  : terminator`
4.  `Version  :  1.92`
5.  `Release  :  2.el8`
6.  `Architecture  : noarch`
7.  `Size  :  526 k`
8.  `Source  : terminator-1.92-2.el8.src.rpm`
9.  `Repository  : epel`
10.  `Summary  :  Store  and run multiple GNOME terminals in one window`
11.  `URL : https://github.com/gnome-terminator`
12.  `License  :  GPLv2`
13.  `Description  :  Multiple GNOME terminals in one window.  This  is a project to produce`
14.   `: an efficient way of filling a large area of screen space with`
15.   `: terminals.  This  is  done by splitting the window into a resizeable`
16.   `: grid of terminals.  As such, you can  produce a very flexible`
17.   `: arrangements of terminals for different tasks.`


```

这个信息告诉你可用软件包的版本、在你系统中注册的哪一个存储库提供了它、该项目的网站以及详细的功能描述。

软件包名称并不总是与你要查找的内容相匹配。例如，假设你正在阅读的文档告诉你必须安装名为 `qmake-qt5` 的东西：

```


1.  `$ dnf search qmake-qt5`
2.  `No matches found.`


```

`dnf` 数据库非常广泛，因此你不要局限于搜索完全匹配的内容。你可以使用 `dnf provides` 命令来了解你正在寻找的东西是否作为某个更大的软件包的一部分而提供：

```


1.  `$ dnf provides qmake-qt5`
2.  `qt5-qtbase-devel-5.12.5-8.el8.i686  :  Development files for qt5-qtbase`
3.  `Repo  : appstream`
4.  `Matched  from:`
5.  `Filename  :  /usr/bin/qmake-qt5`

7.  `qt5-qtbase-devel-5.15.2-3.el8.x86_64 :  Development files for qt5-qtbase`
8.  `Repo  : appstream`
9.  `Matched  from:`
10.  `Filename  :  /usr/bin/qmake-qt5`


```

可以确认应用程序 `qmake-qt5` 是名为 `qt5-qtbase-devel` 的软件包的一部分。它还告诉你，该应用程序会安装到 `/usr/bin`，因此你知道了安装后它的确切位置。

有时我发现自己会从完全不同的角度来对待 `dnf`。有时，我已经确认我的系统上安装了一个应用程序；我只是不知道我是怎么得到它的。还有一些时候，我知道我安装了一个特定的软件包，但我不清楚这个软件包到底在我的系统上安装了什么。

如果你需要对包的有效负载payload进行 “逆向工程reverse engineer”，可以使用 `dnf repoquery` 命令和 `--list` 选项。这将查看存储库中有关软件包的元数据，并列出该软件包提供的所有文件：

```


1.  `$ dnf repoquery --list qt5-qtbase-devel`
2.  `/usr/bin/fixqt4headers.pl`
3.  `/usr/bin/moc-qt5`
4.  `/usr/bin/qdbuscpp2xml-qt5`
5.  `/usr/bin/qdbusxml2cpp-qt5`
6.  `/usr/bin/qlalr`
7.  `/usr/bin/qmake-qt5`
8.  `/usr/bin/qvkgen`
9.  `/usr/bin/rcc-qt5`
10.  `[...]`


```

这些列表可能很长，使用 `less` 或你喜欢的分页命令配合管道操作会有所帮助。

如果你决定系统中不再需要某个应用程序，可以使用 `dnf remove` 卸载它，该软件包本身安装的文件以及不再需要的任何依赖项都会被移除：

```


1.  `$ dnf remove bigapp`


```

有时，你发现随着一个应用程序一起安装的依赖项对后来安装的其他应用程序也有用。如果两个包需要相同的依赖项，`dnf remove` _不会_ 删除依赖项。在安装和卸载大量应用程序之后，孤儿软件包散落各处的现象并不少见。大约每年我都要执行一次 `dnf autoremove` 来清除所有未使用的软件包：

```


1.  `$ dnf autoremove`


```

这不是必需的，但这是一个让我的电脑感觉更好的大扫除步骤。

你对包管理器的工作方式了解得越多，在必要时安装和查询应用程序就越容易。即便你不是 `dnf` 的重度使用者，当你发现自己与基于 RPM 的发行版交互时，了解它也会很有用。

告别 `yum` 后，我最喜欢的包管理器之一是 `dnf` 命令。虽然我不喜欢它的所有子命令，但我发现它是目前最健壮的 包管理系统package management system 之一。 [下载我们的 dnf 速查表](https://opensource.com/downloads/dnf-cheat-sheet) 习惯该命令，不要害怕尝试一些新技巧。一旦熟悉了它，你可能会发现很难使用其他任何东西替代它。

> **[dnf 速查表](https://opensource.com/downloads/dnf-cheat-sheet)**

* * *

![](_assets/linisi.svg)