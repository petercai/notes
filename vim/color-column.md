可以的，而且 **`vi / vim` 对“80 列参考线”支持得非常好** 👍
常见有 **两种方式**，用途不一样。

---

## ✅ 方法一：显示 **80 列竖线（推荐）**

这是最常用、最直观的方式 —— 在第 80 列画一条**竖线参考线**。

### 临时生效（当前会话）

```vim
:set colorcolumn=80
```

### 多列（比如 80–100 都提示）

```vim
:set colorcolumn=80,100
```

### 永久生效（写入 `~/.vimrc`）

```vim
set colorcolumn=80
```

📌 **这是“编码/写文档最推荐”的做法**
它不会强制换行，只是提醒你：**这里已经超宽了**。

---

## ✅ 方法二：80 列自动换行（不太推荐）

如果你是想 **真的在 80 列强制换行**（比如写邮件、commit message）：

```vim
:set textwidth=80
```

### 配合自动格式化

```vim
:set textwidth=80
:set formatoptions+=t
```

⚠️ 注意：

* 写代码 **一般不建议** 自动换行
* 适合：`git commit`、邮件、纯文本

---

## 🔥 推荐组合（程序员常用）

```vim
set colorcolumn=80
set textwidth=0
```

含义：

* ✅ **只提示，不自动换行**
* ❌ 不破坏代码结构

---

## 🎨 让 80 列线更明显（可选）

默认颜色可能很淡，你可以改高亮：

```vim
highlight ColorColumn ctermbg=darkgray
```

或在 GUI / 256 色终端：

```vim
highlight ColorColumn guibg=#2a2a2a
```

---

## 🧠 总结（怎么选）

| 需求            | 设置                          |
| ------------- | --------------------------- |
| 只想知道有没有超 80 列 | `colorcolumn=80` ✅          |
| 想强制 80 列换行    | `textwidth=80`              |
| 写代码           | 只用 `colorcolumn`            |
| 写文档 / 邮件      | `textwidth + formatoptions` |

---

如果你愿意，我可以帮你：

* ✨ 给你一份 **极简 vimrc**（不影响性能）


你这个需求里**有一个关键点需要先纠正**，否则后面一定会踩坑：

> **❌ nvim-tree 不能安装在 Vim 里**
> **✅ nvim-tree = Neovim 专用插件**

你现在的环境是：**Windows + 终端 Vim（不是 Neovim）**
所以我们要 **把目标拆成三件事，分别用 Vim 能做到的“最优解”来完成**。

---

## 一、先说清楚：为什么 Vim 装不了 nvim-tree（很重要）

### ❌ 为什么不行

* **Neovim** ≠ **Vim**
* **nvim-tree** 使用：

  * Lua
  * Neovim API
* **Vim 不支持 Lua 插件**

👉 所以在 **Vim** 里 *不可能* 安装 nvim-tree
这是技术层面的“硬不可能”，不是你配置问题。

---

## 二、Windows 终端 Vim 的正确方案（完全不用鼠标）

你要的是：

> ✔ 左侧文件树
> ✔ 光标移动 → 文件自动跟随
> ✔ 像 VS Code
> ✔ 纯键盘

### ✅ Vim 里的最佳组合只有两个选择

| 插件           | 是否 Vim 原生 | 类 VS Code | 稳定    |
| ------------ | --------- | --------- | ----- |
| **netrw**    | ✅ 内置      | ⭐⭐        | ⭐⭐⭐⭐⭐ |
| **NERDTree** | ❌ 插件      | ⭐⭐⭐⭐      | ⭐⭐⭐⭐  |

我会给你 **两个方案**，你可以任选：

---

# 方案 A（强烈推荐）：Vim 内置 netrw（零安装）

## 1️⃣ 基础设置（80 列线号 + 60 列提示线）

在 **Windows 的 `_vimrc`** 里加：

```vim
" ===== 基础显示 =====
set number
set relativenumber
set cursorline

" 60 列参考线
set colorcolumn=60

" 自动换行显示（不是真的换行）
set wrap
set linebreak
```

---

## 2️⃣ 打开左侧文件树（netrw）

```vim
:Ex
```

或直接左右分屏：

```vim
:Vex
```

---

## 3️⃣ netrw 的核心键位（必须会）

| 键       | 功能          |
| ------- | ----------- |
| `j k`   | 上下移动        |
| `l`     | 进入目录 / 打开文件 |
| `h`     | 返回上级        |
| `-`     | 上一级目录       |
| `%`     | 新建文件        |
| `d`     | 新建目录        |
| `R`     | 重命名         |
| `D`     | 删除          |
| `Enter` | 打开          |

---

## 4️⃣ 光标跟随（模拟 VS Code）

加到 `_vimrc`：

```vim
let g:netrw_browse_split = 4
let g:netrw_altv = 1
let g:netrw_winsize = 25
```

效果：

* 文件树固定在左侧
* 打开文件自动在右侧
* 不需要鼠标

---

## 5️⃣ 完全键盘文件管理流（推荐）

**典型流程：**

```text
Vex        → 打开文件树
j/k        → 选文件
l / Enter  → 打开
Ctrl-w h   → 回到文件树
```

👉 这已经是 **纯 Vim 世界里的 VS Code 流程**

---
