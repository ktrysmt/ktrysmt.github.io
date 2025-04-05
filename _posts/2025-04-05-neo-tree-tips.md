---
layout: post
title: "neo-tree.nvimのちょっとした手直し"
date: 2025-04-05 22:00:00 +0900
categories: Vim
published: true
use_toc: false
description: ""
---

### 自然順ソート

vim-fernから微妙に不満だったのがディレクトリやフォルダのソート順がいわゆる自然順ではなかった点。
`exa` あらため `eza` はこのへんいい感じにやってくれるので余計に差が気になっていた。

sortのカスタマイズ性なども踏まえneo-tree.nvimに移行しつつ設定を固めた。

natural sort のコードは以下

```lua
      local function natural_sort(a, b)
        local function tonumber_if_possible(s)
          local num = tonumber(s)
          if num then
            return num
          else
            return s
          end
        end

        local function split_by_number(s)
          local parts = {}
          local current_part = ""
          for i = 1, #s do
            local char = s:sub(i, i)
            if char:match("%d") then
              if current_part ~= "" and not current_part:match("%d") then
                table.insert(parts, current_part)
                current_part = ""
              end
            else
              if current_part ~= "" and current_part:match("%d") then
                table.insert(parts, tonumber(current_part))
                current_part = ""
              end
            end
            current_part = current_part .. char
          end
          if current_part ~= "" then
            table.insert(parts, tonumber_if_possible(current_part))
          end
          return parts
        end

        local parts_a = split_by_number(a.path)
        local parts_b = split_by_number(b.path)

        local len_a = #parts_a
        local len_b = #parts_b
        local min_len = math.min(len_a, len_b)

        for i = 1, min_len do
          local part_a = parts_a[i]
          local part_b = parts_b[i]

          if type(part_a) == "number" and type(part_b) == "number" then
            if part_a ~= part_b then
              return part_a < part_b
            end
          elseif part_a ~= part_b then
            return tostring(part_a) < tostring(part_b)
          end
        end

        return len_a < len_b
      end
```

呼び出しは以下

```lua
require("neo-tree").setup({
  sort_function = function(a, b)
    if a.type ~= b.type then
      return a.type < b.type
    else
      return natural_sort(a, b)
    end
  end,
```

## focus表示とgit root

あとよくfiler出すと同時に git root に移動しつつcurrent bufferのファイルにfocusしたいのだが、これも非常にやりやすかった。
キーマップを少し調整して以下のようになった。

```lua
vim.keymap.set("n", "<C-e>", "<Cmd>Neotree left toggle<CR>", { silent = true })
vim.keymap.set("n", "<C-w>f", function()
  local path = vim.fn.system("git rev-parse --show-toplevel")
  if vim.bo.filetype == "" then
    vim.cmd("Neotree left")
    return
  end
  vim.defer_fn(function()
    vim.cmd("Neotree action=focus reveal_file=% dir=" .. path)
  end, 100)
end, { silent = true })
```

`<C-w>f`のほうを少し解説すると起動直後など`&ft`がNoneなら単にfiler表示、current bufferがあるときは非同期にdelay(100ms)をいれつつgit rootに移動しつつファイルの位置をfocs表示。
`dir=`や`reveal_file=`で引数を直接指定しても表示とfocusが同時に走ると描画の関係上カーソルの移動が追いつかないことがあるので、delayをいれている。


## trashコマンド対応

今更感あるがtrash/trash-cliを最近は使うようにしているので、それも対応。
neo-treeはカスタムコマンドをセットしやすくなっているのもいい。
設定は[このへん](https://github.com/nvim-neo-tree/neo-tree.nvim/issues/202)を参考に。

```lua
commands = {
  trash = function(state)
    local inputs = require("neo-tree.ui.inputs")
    local path = state.tree:get_node().path
    local _, name = utils.split_path(path)

    local msg = string.format("Are you sure you want to trash '%s'? (y/n) ", name)
    inputs.confirm(msg, function(confirmed)
      if not confirmed then
        return
      end
      vim.fn.system({"trash", vim.fn.fnameescape(path)})
      require("neo-tree.sources.manager").refresh(state.name)
    end)
  end,
},
window = {
  mappings = {
    ["d"] = "trash",
```


残タスクとしてはdelete時、insertモードで直接`<cr>`するとconfirmedがtrueにならず、一度`<esc>`や`<C-c>`でnormalモードにしてから`<cr>`する必要がある点。ここだけなんとかしたいが今日は時間切れ。また時間あるときに。


