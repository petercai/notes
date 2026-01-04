# install in windows
# Windows Console Vim 安装 FZF + Ripgrep 快速指南

## 1. 安装必需工具

### 安装 Scoop (包管理器)
打开 PowerShell，执行：
```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
```

### 安装 FZF 和 Ripgrep
```powershell
scoop install fzf ripgrep
```

## 2. 安装 Vim 插件管理器

在 `~/_vimrc` (或 `~/vimfiles/vimrc`) 文件开头添加：

```vim
" 安装 vim-plug
if empty(glob('~/vimfiles/autoload/plug.vim'))
  silent !curl -fLo ~/vimfiles/autoload/plug.vim --create-dirs
    \ https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
endif

call plug#begin('~/vimfiles/plugged')
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
call plug#end()

" FZF 快捷键配置
nnoremap <C-p> :Files<CR>
nnoremap <C-f> :Rg<CR>
nnoremap <C-b> :Buffers<CR>
```

## 3. 安装插件

重启 Vim，执行：
```vim
:PlugInstall
```

## 4. 使用

- **Ctrl+P** - 快速查找文件
- **Ctrl+F** - 在文件内容中搜索文本 (ripgrep)
- **Ctrl+B** - 切换打开的缓冲区

完成！现在可以在 Vim 中快速搜索文件和文本了。



# 重置 FZF 配置

## 快速诊断和修复

### 1. 检查 FZF 是否正常工作
在 PowerShell 中测试：
```powershell
fzf --version
```

### 2. 清除可能冲突的环境变量
在 PowerShell 中执行：
```powershell
# 查看当前 FZF 相关环境变量
Get-ChildItem Env: | Select-String FZF

# 临时清除（本次会话）
$env:FZF_DEFAULT_OPTS = ""
$env:FZF_DEFAULT_COMMAND = ""
```

### 4. 设置推荐的 FZF 配置
在 PowerShell 配置文件中添加（`$PROFILE`）：
```powershell
# 编辑配置文件
notepad $PROFILE

# 添加以下内容：
$env:FZF_DEFAULT_OPTS = "--height 40% --layout=reverse --border"
$env:FZF_DEFAULT_COMMAND = "dir /s /b"
```

这个配置问题出在 **Windows 的引号处理** 上。

## 问题分析

1. **单引号在 Windows 中不被识别** - Windows PowerShell 和 CMD 不支持单引号作为字符串分隔符
2. **glob 模式需要转义** - Windows 需要不同的引号处理方式

## 修复方案

### 对于 PowerShell
在 `$PROFILE` 文件中使用双引号：

```powershell
$env:FZF_DEFAULT_COMMAND = "rg --files --hidden --follow --glob ""!.git/"" --glob ""!.gitignore"" --glob ""!node_modules/"""
```

或者更简洁的写法：
```powershell
$env:FZF_DEFAULT_COMMAND = 'rg --files --hidden --follow --glob "!.git/" --glob "!.gitignore" --glob "!node_modules/"'
```

### 对于 CMD
如果使用 CMD，在系统环境变量中设置：
```cmd
set FZF_DEFAULT_COMMAND=rg --files --hidden --follow --glob "!.git/" --glob "!.gitignore" --glob "!node_modules/"
```

### 在 Vim 配置中设置（推荐）
在 `_vimrc` 中直接设置，避免 shell 差异：

```vim
let $FZF_DEFAULT_COMMAND = 'rg --files --hidden --follow --glob "!.git/" --glob "!.gitignore" --glob "!node_modules/"'
```

## 验证配置
```powershell
echo $env:FZF_DEFAULT_COMMAND
fzf
```

关键：**Windows 用双引号，PowerShell 变量中的双引号需要转义或用单引号包裹整个命令**。

# 添加排除 .venv 目录

## PowerShell 配置
```powershell
$env:FZF_DEFAULT_COMMAND = 'rg --files --hidden --follow --glob "!.git/" --glob "!.gitignore" --glob "!node_modules/" --glob "!.venv/"'
```

## Vim 配置（推荐）
在 `_vimrc` 中：
```vim
let $FZF_DEFAULT_COMMAND = 'rg --files --hidden --follow --glob "!.git/" --glob "!.gitignore" --glob "!node_modules/" --glob "!.venv/"'
```

## 更简洁的写法
可以合并多个排除模式：
```vim
let $FZF_DEFAULT_COMMAND = 'rg --files --hidden --follow --glob "!{.git,.venv,node_modules}/"'
```

## 如果还想排除其他 Python 相关目录
```vim
let $FZF_DEFAULT_COMMAND = 'rg --files --hidden --follow --glob "!{.git,.venv,node_modules,__pycache__,.pytest_cache,.mypy_cache}/"'
```

**关键点**：每个排除模式加一个 `--glob "!目录名/"` 即可，或使用 `{}` 花括号语法批量排除。

# install in general

To install and configure **`fzf`** and **`ripgrep`** as plugins for **Vim** (or Neovim) on **macOS**, follow these steps:

---

## ✅ Step 1: Install `fzf` and `ripgrep`

You can use [https://brew.sh/](https://brew.sh/) to install both tools:

After installing `fzf`, run its install script to enable shell integration:

$(brew --prefix)/opt/fzf/install

---

## ✅ Step 2: Install Vim Plugin Manager (if needed)

If you don’t already use a plugin manager, install one. Popular choices:

- **vim-plug**:
    
    curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    
        https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
    

---

## ✅ Step 3: Add Plugins to `.vimrc` or `init.vim`

If you're using **vim-plug**, add the following to your `~/.vimrc` (or `~/.config/nvim/init.vim` for Neovim):

call plug#begin('~/.vim/plugged')

  

" fzf plugin

Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }

Plug 'junegunn/fzf.vim'

  

call plug#end()

Then run:

:source ~/.vimrc

:PlugInstall

---

## ✅ Step 4: Use `fzf` and `ripgrep` in Vim

Once installed, you can use commands like:

- `:Files` — fuzzy find files
- `:Rg <pattern>` — search using `ripgrep`
- `:Buffers` — switch between open buffers
- `:Lines` — search across all open buffers

You can also map keys for convenience:

nnoremap <silent> <C-p> :Files<CR>

nnoremap <silent> <C-f> :Rg<CR>

---

Would you like help customizing the search behavior or integrating it with LSP or Telescope (for Neovim)?