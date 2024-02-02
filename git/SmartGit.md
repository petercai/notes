# SmartGit



下载
--

| 名称 | 下载地址 |
| ---- | ---- |
| SmartGit v21.1.7 | [Windows](https://www.syntevo.com/downloads/smartgit/archive/smartgit-portable-21_1_7.7z) |
|  | [Mac](https://www.syntevo.com/downloads/smartgit/archive/smartgit-macosx-21_1_7.dmg) |
|  | [Linux](https://www.syntevo.com/downloads/smartgit/archive/smartgit-linux-21_1_7.tar.gz) |


1 . 先安装 SmartGit/SmartSynchronize，安装后运行一下。  
2 . 用编辑器打开 smartgit.vmoptions 文件，此文件可以在以下位置找到：

###### SmartGit
| 平台 | 位置 |
| --- | --- |
| mac | /Library/Preferences/SmartGit/ |
| linux | /.config/smartgit/ |
| windows | %APPDATA%\\syntevo\\SmartGit\ 或者你直接在bin目录下改。 |

###### SmartSynchronize

| 平台 | 位置 |
| --- | --- |
| mac | /Library/Preferences/SmartSynchronize/ |
| linux | /.config/smartsynchronize/ |
| windows | %APPDATA%\\syntevo\\SmartSynchronize\ 或者你直接在bin目录下改。 |

4 . 在打开的 smartgit.vmoptions 文件末行添加：-javaagent:/absolute/path/to/your-agent.jar，

| 平台 | 示例 |
| --- | --- |
| mac | javaagent:/Users/macwk.com/your-agent.jar |
| linux | javaagent:/home/macwk.com/your-agent.jar |
| windows | javaagent:C:\\Users\\macwk.com\\your-agent.jar |

5 . 启动 SmartGit，注册使用压缩包内的 license.zip 文件。  


