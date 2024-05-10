# gu：轻松切换个人和工作的 Git 用户配置
      

#### 引言

开发者在处理多个项目时常面临切换Git用户配置的需求。`gu`命令行工具，专为简化这一流程而设计，允许用户在个人和工作项目间轻松切换Git配置。

#### 什么是gu？

> 仓库：[github.com/YOUNGmaxer/…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FYOUNGmaxer%2Fgit-user "https://github.com/YOUNGmaxer/git-user")

`gu`是一个用于在单个机器上管理多个Git用户配置的命令行工具，它支持用户快速在不同的Git账户间切换，特别适合同时参与多个项目的开发者。

#### 如何安装gu？

要安装`gu`，只需执行以下命令：

```bash
curl -sSL https://raw.githubusercontent.com/YOUNGmaxer/git-user/main/install.sh | bash

```

此命令会下载并安装`gu`，让你能够立即开始使用。

#### gu的核心功能及使用示例

1.  **设置用户信息**
    
    使用`gu set`命令可以为当前目录设置Git用户信息。例如，如果你想为当前项目设置用户名和邮箱，可以这样做：
    
    ```bash
    > gu set
    
    > Enter user name: aaa
    > Enter email: aaa@example.com
    > Enter alias: aaa
    
    ```
    
    若要在全局范围内设置用户信息，使用`gu set --global`命令。
    
2.  **显示当前用户**
    
    `gu show`命令可以显示当前目录或全局配置的Git用户信息。这在确认你正使用正确的用户配置时非常有用：
    
    ```bash
    > gu show
    
    > Name: your-username, Email: your-email@example.com
    
    ```
    
3.  **添加和删除用户配置文件**
    
    你可以通过`gu add`命令添加新的用户配置。如果需要删除不再使用的用户配置，可以使用`gu delete`命令并选择相应的配置文件进行删除。
    
4.  **切换和列出用户配置**
    
    `gu switch`命令允许你在不同的用户配置文件之间切换。使用时，它会显示所有配置文件列表供你选择：
    
    ```bash
    > gu switch
    
    > Available profiles:
    > 2) Alias: bbb, Name: bbb, Email: bbb@example.com
    > 3) Alias: aaa, Name: aaa, Email: aaa@example.com
    > Enter the number of the profile to switch to: 
    
    ```
    
    要查看所有已配置的用户信息，可以使用`gu list`命令：
    
    ```bash
    gu list
    
    ```
    

#### 使用场景

`gu`工具特别适合需要在多个Git账户间切换的开发者，比如同时管理个人和工作的Git项目。它使得用户能够快速更改Git配置，确保代码提交总是使用正确的用户信息。
