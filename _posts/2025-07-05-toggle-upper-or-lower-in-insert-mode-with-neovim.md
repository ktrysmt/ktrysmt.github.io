---
layout: post
title: "Neovimのインサートモードで大文字・小文字を切り替え"
date: 2025-07-05 09:00:00 +0900
categories: Vim
published: true
use_toc: false
---

結構便利だったので少し改変して
- Toggle UPPER/lower
- 対象範囲をカーソル下の単語に変更
- カーソル位置の保持


```lua
vim.keymap.set("i", "<C-l>",
  function()
    local word = vim.fn.expand('<cword>')
    if word == "" then
      return ""
    end
    local pos = vim.fn.getpos('.')
    if word == word:upper() then
      return "<C-o>diw" ..
          word:lower() ..
          "<C-o>:call setpos('.', [" .. pos[1] .. "," .. pos[2] .. "," .. pos[3] .. "," .. pos[4] .. "])<CR>"
    else
      return "<C-o>diw" ..
          word:upper() ..
          "<C-o>:call setpos('.', [" .. pos[1] .. "," .. pos[2] .. "," .. pos[3] .. "," .. pos[4] .. "])<CR>"
    end
  end,
  { expr = true }
)
```

## 参考

* <https://zenn.dev/vim_jp/articles/2024-10-07-vim-insert-uppercase>
