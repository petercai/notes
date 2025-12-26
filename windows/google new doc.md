在 **Windows 11 桌面** 上创建一个 **“一键打开全新 Google Docs 文档（New Doc）” 的快捷方式**，有几种**稳定、好用**的方法。下面从 **最推荐 → 进阶自动化** 逐步说明。

---

## ✅ 最简单 & 最推荐：使用 `docs.new` 快捷方式（官方支持）

Google 提供了 **内置 New Doc URL**，每次打开都会创建一个**全新的空白文档**。

### 步骤

1. **右键桌面 → 新建 → 快捷方式**
    
2. 在“位置”中输入：
    
    ```
    https://docs.new
    ```
    
3. 点击 **下一步**
    
4. 名称填写：
    
    ```
    New Google Doc
    ```
    
5. 完成 ✅
    

### 使用效果

- 双击快捷方式
    
- 浏览器自动打开
    
- **直接创建一个全新的 Google Docs 文档**
    

📌 支持所有浏览器（Chrome / Edge / Firefox）

---

## 🎯 常用替代（其他 Google 文档类型）

你也可以顺手建一套：

|类型|URL|
|---|---|
|新 Docs|`https://docs.new`|
|新 Sheets|`https://sheets.new`|
|新 Slides|`https://slides.new`|
|新 Forms|`https://forms.new`|
|新 Drawings|`https://drawings.new`|

---

## ⭐ 进阶优化 1：指定浏览器打开（Edge / Chrome）

### 示例（Edge）

```
"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" https://docs.new
```

### 示例（Chrome）

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" https://docs.new
```

📌 这样可以避免系统默认浏览器不一致的问题。

---

## ⭐ 进阶优化 2：像 App 一样打开（无地址栏）

如果你想要 **更像本地应用** 的体验：

### Chrome / Edge 命令

```
chrome.exe --app=https://docs.new
```

或

```
msedge.exe --app=https://docs.new
```

效果：

- 无标签栏
    
- 无地址栏
    
- 类似桌面应用 📄
    

---

## 🧠 进阶自动化（可选）

### 方法 A：开机自动打开一个新 Doc

1. `Win + R` → 输入：
    
    ```
    shell:startup
    ```
    
2. 把刚才的快捷方式复制进去
    

👉 每次 Windows 启动都会新建一个 Google Doc

---

### 方法 B：设置快捷键（如 `Ctrl + Alt + D`）

1. 右键快捷方式 → **属性**
    
2. 在「快捷键」里输入：
    
    ```
    Ctrl + Alt + D
    ```
    

---

## 🎨 个性化（可选）

- 替换图标：  
    使用 Google Docs 的 `.ico` 图标文件
    
- 重命名为：
    
    - `✍ New Writing Doc`
        
    - `📄 Blank Google Doc`
        

---

## ✅ 推荐方案总结

|需求|最佳方案|
|---|---|
|最简单|`https://docs.new`|
|指定浏览器|浏览器 exe + URL|
|类 App 体验|`--app=https://docs.new`|
|自动打开|放入 Startup|
|快捷键|桌面快捷方式|

---

如果你愿意，我可以：

- ✅ 帮你生成 **已配置好的 `.lnk` 快捷方式**
    
- ✅ 帮你做一个 **PowerShell 一键安装脚本**
    
- ✅ 给你一个 **New Doc / Sheet / Slide 套装**
    

你想要哪种？