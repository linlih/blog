---
title: "Neovim for Go"
description: "neovim_for_go"
keywords: "neovim_for_go"

date: 2024-03-24T11:11:27+08:00
lastmod: 2024-03-24T11:11:27+08:00

categories:
 -
tags:
  -
  -

# Post's origin author name
#author:
# Post's origin link URL
#link:
# Image source link that will use in open graph and twitter card
#imgs:
# Expand content on the home page
#expand: true
# It's means that will redirecting to external links
#extlink:
# Disabled comment plugins in this post
#comment:
# enable: false
# Disable table of content int this post
# Notice: By default will automatic build table of content 
# with h2-h4 title in post and without other settings
#toc: false
# Absolute link for visit
#url: "neovim_for_go.html"
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

#  Neovim 安装
Neovim 项目提供了非常详细的[安装方法](https://github.com/neovim/neovim/blob/master/INSTALL.md)，可以根据自己的环境选择对应的安装方法进行安装。

# NvChad 安装
NvChad 是使用 lua 编写的 Neovim 配置，提供了开箱即用的配置和好看的 UI，可以基于这个定制化自己的 Neovim

安装命令，这里安装的是 2.0 的版本：
```bash
mv ~/.config/nvim ~/.config/nvim.backup
rm -rf ~/.local/share/nvim
git clone -b v2.0 https://github.com/NvChad/NvChad ~/.config/nvim --depth 1
```

# 配置

输入：`nvim`，会提示是否进行配置的修改，输入 `N` 使用默认配置，然后会进入一些插件的安装（需要比较好的网络，不然可能下载会失败）

切换主题：`space + t + h` ，选择 catppuccin 主题。

注意，这个场景的配置中，机器的网络要保证访问 github 比较顺畅，如果是云平台机器如果访问 github 较慢，可以考虑更换 github 的解析地址或者开启代理。

**语法高亮**：可以使用 `:TSInstall language`，查看有哪些已经安装了，可以使用 `:TSInstallInfo` 来查看。

比如 go 语言的语法高亮，在 vim  的 command 命令下执行：`:TSIntall go`

**文件树**：在 NvChad 默认安装了对应的文件树的插件，可以通过 `ctrl + n` 打开文件树的面板，使用方向键可以选择文件，回车打开文件夹或者打开文件。

如果想要标记文件，使用 `m` 键。

创建新的文件，使用 `a` 键。复制文件 `c` 键，黏贴文件 `v` 键，重命名文件 `r` 键。

如果知道要打开的文件是什么，可以直接通过文件搜索来打开，输入 `space + f + f` （记忆 find in files）会打开一个搜索框来查找文件，这个是查找当前目录下的所有文件，如果想查找已经打开的文件，则可以输入 `space + f + b` （记忆 find in buffers）

这样一下子有太多的快捷键要记忆了是吧，NvChad 提供了 cheat sheet 供查看，输入 `space + c + h` （记录：call for help)，另外 NvChad 也提供了另外的帮助方式，输入 `space` 等一会就会提示可输入的选择了。

在 NvChad 可以使用 `:vsp` 垂直分割 `:sp` 水位分割，使用 `ctrl + h/j/k/l` 进行窗口移动，推出窗口 `:q`

**行号**：使用 `space + n` 进行打开和关闭，如果要开启相对行号，则使用 `space + r + n`

**窗口管理**：当打开一个文件的时候，则会开启一个 tab，使用 tab 键可以切换不同打开的文件，`space + x` 可以关闭 tab

**Terminal窗口**：`space + v` 打开水平终端，`space + h` 打开垂直终端，输入 exit 可以推出终端。

自定义配置：需要在特定的路径下创建对应的自定义配置文件，在路径 `~/.config/nvim/lua/custom` 进行创建，因为我们使用了 NvChad，所以这个目录下已经有了 `chadrc.lua` 文件。
# Go开发配置
安装 go 的工具：
```shell
go install mvdan.cc/gofumpt@latest
go install -v github.com/incu6us/goimports-reviser/v3@latest
go install github.com/segmentio/golines@latest
go install github.com/go-delve/delve/cmd/dlv@latest
```

最终的配置来自博主 Dreams of Code：https://github.com/dreamsofcode-io/neovim-go-config，这里把配置贴过来，方便直接在这个页面内直接复制。

custom 目录配置的结构如下：
```bash
├── chadrc.lua
├── configs
│   ├── lspconfig.lua
│   └── null-ls.lua
├── mappings.lua
└── plugins.lua
```

具体的配置信息，chadrc.lua：
```lua
---@type ChadrcConfig
local M = {}

M.ui = { theme = 'catppuccin' }
M.plugins = "custom.plugins"
M.mappings = require "custom.mappings"

return M
```
mappings.lua：
```lua
local M = {}

M.dap = {
  plugin = true,
  n = {
    ["<leader>db"] = {  -- debug breakpoint 设置断点
      "<cmd> DapToggleBreakpoint <CR>",
      "Add breakpoint at line"
    },
    ["<leader>dus"] = { -- debug ui sidebar 打开调试变量侧边栏
      function ()
        local widgets = require('dap.ui.widgets');
        local sidebar = widgets.sidebar(widgets.scopes);
        sidebar.open();
      end,
      "Open debugging sidebar"
    }
  }
}

M.dap_go = {
  plugin = true,
  n = {
    ["<leader>dgt"] = {   -- debug go test 调试go测试函数
      function()
        require('dap-go').debug_test()
      end,
      "Debug go test"
    },
    ["<leader>dgl"] = {  -- debug go last 调试上次的内容
      function()
        require('dap-go').debug_last()
      end,
      "Debug last go test"
    }
  }
}

M.gopher = {
  plugin = true,
  n = {
    ["<leader>gsj"] = {  -- go struct json 添加结构体的 json tag
      "<cmd> GoTagAdd json <CR>",
      "Add json struct tags"
    },
    ["<leader>gsy"] = { -- go struct yaml 添加结构体的 yaml tag
      "<cmd> GoTagAdd yaml <CR>",
      "Add yaml struct tags"
    }
  }
}

return M
```

plugins.lua
```lua
local plugins = {
  {
    "williamboman/mason.nvim",
    opts = {
      ensure_installed = {
        "gopls",
      },
    }
  },
  {
    "neovim/nvim-lspconfig",
    config = function ()
      require "plugins.configs.lspconfig"
      require "custom.configs.lspconfig"
    end
  },
  {
    "jose-elias-alvarez/null-ls.nvim",
    ft="go",
    opts=function ()
      return require "custom.configs.null-ls"
    end,
  },
  {
    "olexsmir/gopher.nvim",
    ft = "go",
    config = function(_, opts)
      require("gopher").setup(opts)
      require("core.utils").load_mappings("gopher")
    end,
    build = function()
      vim.cmd [[silent! GoInstallDeps]]
    end,
  },
}

return plugins
```

configs/lspconfig.lua
```lua
local on_attach = require("plugins.configs.lspconfig").on_attach
local capabilities = require("plugins.configs.lspconfig").capabilities

local lspconfig = require("lspconfig")
local util = require("lspconfig/util")

lspconfig.gopls.setup {
  on_attach = on_attach,
  capabilities = capabilities,
  cmd = {"gopls"},
  filetypes = { "go", "gomod", "gowork", "gotmpl" },
  root_dir = util.root_pattern("go.work", "go.mod", ".git"),
  settings = {
    gopls = {
      completeUnimpoted = true,
      usePlaceholders = true,
      analyses = {
        unusedparams = true,
      }
    }
  }
}
```

config/null-ls.lua
```lua
local null_ls = require("null-ls")
local augroup = vim.api.nvim_create_augroup("LspFormatting", {})

local opts = {
  sources = {
    --null_ls.builtins.formatting.gofmt,
    --null_ls.builtins.formatting.goimports,
    null_ls.builtins.formatting.gofumpt,
    null_ls.builtins.formatting.goimports_reviser,
    null_ls.builtins.formatting.golines,
  },
  on_attach = function(client, bufnr)
    if client.supports_method("textDocument/formatting") then
      vim.api.nvim_clear_autocmds({
        group = augroup,
        buffer = bufnr,
      })
      vim.api.nvim_create_autocmd("BufWritePre", {
        group = augroup,
        buffer = bufnr,
        callback = function ()
          vim.lsp.buf.format({ bufnr = bufnr })
        end,
      })
    end
  end,
}

return opts
```

code navigation:
`space + g + d` go to definition，回到跳转的地方使用： `ctrl + o` 

# 调试
现在是按照 Dream of Code 博主的步骤一步一步配置的，关于更加详细的配置还没有搞的很明白，需要看下这些配置的具体含义，到时候再补充下这部分内容。

# 参考
[Turn VIM into a full featured IDE with only one command](https://www.youtube.com/watch?v=Mtgo-nP_r8Y)
[The perfect Neovim setup for Go](https://www.youtube.com/watch?v=i04sSQjd-qo)