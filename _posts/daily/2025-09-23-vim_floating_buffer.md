---
layout: post
title: vim popup으로 floating window를 사용해보자
subtitle: my vim popup configuration
tags: [vim,vim-popup,floating-window]
comments: true
author: widehyo
---

- vim 8.x 이상부터 popup이라는 기능을 제공한다(`:h popup`)
- vim의 popup은 floating window를 제공하는 기능으로, 주로 `popup_menu()`를 사용한다
- `popup_menu()`의 디폴트 동작은 주어진 리스트를 j,k로 navigating하며 엔터로 팝업을 닫는다
  - 팝업이 닫히는 기본 동작을 변경하려면 callback 함수를 만들어 config의 `callback`키에 연결한다.
    - `callback`으로 연결된 함수는 첫번째 인자로 popup buffer의 window id, 두번째 인자로 item index가 전달되어 실행된다.
  - 팝업에서 입력된 키를 이용해 동작방식을 변경하려면 config의 `filter`를 통해 제어한다.
    - `filter`로 연결된 함수는 첫번째 인자로 popup buffer의 window id, 두번째 인자로 입력된 키가 전달되어 실행된다.
    - vim의 노말키를 보내려면 팝업의 winid를 이용하여 `call win_execute(winid, cmd)`를 이용한다.

- less 바인딩과 visual mode 및 yank 기능을 포함한 popup filter 예시
```vim
function! PopupFilter(winid, key) abort
    if a:key ==# "j"
        call win_execute(a:winid, "normal! j")
    elseif a:key ==# "k"
        call win_execute(a:winid, "normal! k")
    elseif a:key ==# "\<c-d>"
        call win_execute(a:winid, "normal! \<c-d>")
    elseif a:key ==# "\<c-u>"
        call win_execute(a:winid, "normal! \<c-u>")
    elseif a:key ==# "\<c-f>"
        call win_execute(a:winid, "normal! \<c-f>")
    elseif a:key ==# "\<c-b>"
        call win_execute(a:winid, "normal! \<c-b>")
    elseif a:key ==# "\<space>"
        call win_execute(a:winid, "normal! \<c-f>")
    elseif a:key ==# "u"
        call win_execute(a:winid, "normal! \<c-b>")
    elseif a:key ==# "v"
        call win_execute(a:winid, "normal! v")
    elseif a:key ==# "V"
        call win_execute(a:winid, "normal! V")
    elseif a:key ==# "y"
        call win_execute(a:winid, "normal! y")
    elseif a:key ==# "G"
        call win_execute(a:winid, "normal! G")
    elseif a:key ==# "g"
        call win_execute(a:winid, "normal! gg")
    elseif a:key ==# 'q'
        call popup_close(a:winid)
    else
        return v:false
    endif
    return v:true
endfunction
```

- 이를 이용하면 vimscript를 이용해 기능을 만들 때 echomsg로 print debugging할 때, 내용을 yank-paste 할 수 있다
- 이 외에도 vim에서 여러 줄로 출력되는 메시지를 플로팅 윈도우에서 편하게 보고 싶을 때 사용하기 적합하다
- 정확히 이 기능을 위하여 만든 함수 예시는 다음과 같다.

```vim
function! OpenExcommandPopup(excommand, from_start = v:true)
  let pop_width = float2nr(&columns * 0.8)
  let pop_height = float2nr(&lines * 0.8)
  let lines = split(execute(a:excommand, 'silent'), '\n')
  let popup_config = #{
        \ scrollbar: 1,
        \ maxheight: pop_height,
        \ minheight: pop_height,
        \ maxwidth: pop_width,
        \ minwidth: pop_width,
        \ filter: 'PopupFilter'
        \ }

  let winid = popup_menu(lines, popup_config)

  if !a:from_start
    call win_execute(winid, 'normal! G')
    call win_execute(winid, 'normal! \<c-b>')
  endif
endfunction
```

- 같은 방법으로 `:! system command`도 vim을 벗어나지 않고 플로팅 윈도우에서 볼 수도 있다.

```vim
function! OpenSystemPopup(command, from_start = v:true)
  let pop_width = float2nr(&columns * 0.8)
  let pop_height = float2nr(&lines * 0.8)

  let lines = split(system(a:command, 'silent'), '\n')
  let popup_config = #{
        \ scrollbar: 1,
        \ maxheight: pop_height,
        \ minheight: pop_height,
        \ maxwidth: pop_width,
        \ minwidth: pop_width,
        \ filter: 'PopupFilter'
        \ }

  let winid = popup_menu(lines, popup_config)

  if !a:from_start
    call win_execute(winid, 'normal! G')
    call win_execute(winid, 'normal! \<c-b>')
  endif
endfunction
```

- 만약 특정한 excommand나 system command를 자주 사용한다면 키매핑을 등록해 사용한다.

```vim
" floating window
nnoremap <leader>msg :call OpenExcommandPopup('messages', v:false)<CR>
nnoremap <leader>map :call OpenExcommandPopup('map')<CR>
nnoremap <leader>reg :call OpenExcommandPopup('registers')<CR>
nnoremap <leader>jumps :call OpenExcommandPopup('jumps', v:true)<CR>
nnoremap <leader>changes :call OpenExcommandPopup('changes', v:false)<CR>
nnoremap <leader>history :call OpenExcommandPopup('history', v:false)<CR>
nnoremap <leader>undo :call OpenExcommandPopup('undo', v:false)<CR>
nnoremap <leader>autocmd :call OpenExcommandPopup('autocmd')<CR>

nnoremap <leader>af :call OpenSystemPopup('awk -f ~/script.awk ~/temp.txt')<CR>
nnoremap <leader>jf :call OpenSystemPopup('jq -f ~/script.jq ~/temp.json')<CR>
```

- 다음은 neovim의 floating window를 사용하는 같은 내용으로, vim에서처럼 `filter`를 통해 키바인딩을 직접 제어할 필요가 없어 편하다.
- 특히 vim에서는 `filter`가 하나의 키만 전달하기 때문에 popup window 내에서 문자열을 검색하는 기능을 넣는 방법을 아직 찾지 못했지만, neovim에서는 하나의 버퍼로 취급하기 때문에 이런 제약이 존재하지 않는다.

```lua
local M = {}

function M.floating_window(lines, field, win_opt, buf_opt)
  lines = lines or {}
  local contents = {}

  win_opt = win_opt or {}
  buf_opt = buf_opt or {}

  for _, line in ipairs(lines) do
    table.insert(contents, line)
  end

  local buf = vim.api.nvim_create_buf(false, true)
  vim.api.nvim_buf_set_lines(buf, 0, -1, false, contents)

  local width = vim.fn.float2nr(vim.o.columns * 0.8)
  local height = vim.fn.float2nr(vim.o.lines * 0.8)

  local win = vim.api.nvim_open_win(buf, true, vim.tbl_deep_extend("force", {
    relative = 'editor',
    width = width,
    height = height,
    row = math.floor((vim.o.lines - height) / 2),
    col = math.floor((vim.o.columns - width) / 2),
    style = 'minimal',
    border = 'rounded',
  }, win_opt))

  -- default window, buffer option
  vim.api.nvim_set_option_value('modifiable', false, {buf=buf})
  vim.api.nvim_set_option_value('cursorline', true, {win=win})

  for key, value in pairs(buf_opt) do
    vim.api.nvim_set_option_value(key, value, {buf=buf})
  end

  return win, buf
end

function M.command_window(cmd, lseek)
  if cmd == nil or cmd == '' then return end
  lseek = lseek or false
  local cmd_result = vim.fn.execute(cmd)
  local lines = vim.split(cmd_result, "\n", { trimempty = true })
  local win, buf = M.floating_window(lines)
  if lseek then
    vim.cmd.normal('G')
  end
end

function M.system_window(cmd, lseek)
  if cmd == nil or cmd == '' then return end
  lseek = lseek or false
  local cmd_result = vim.fn.system(cmd)
  local lines = vim.split(cmd_result, "\n", { trimempty = true })
  local win, buf = M.floating_window(lines)
end

function M.awk_window()
  M.system_window('awk -f ~/script.awk ~/temp.txt', true)
end

function M.jq_window()
  M.system_window('jq -r -f ~/script.jq ~/temp.json', false)
end

function M.message_window()
  M.command_window('message', false)
end

function M.map_window()
  M.command_window('map', false)
end

function M.autocmd_window()
  M.command_window('autocmd', false)
end
```

- 역시 키매핑을 이용해 사용한다

```lua
local cmd_window = require('util.buf.cmd_window')

-- floating window
vim.keymap.set('n', '<leader>af', cmd_window.awk_window)
vim.keymap.set('n', '<leader>jf', cmd_window.jq_window)

vim.keymap.set('n', '<leader>msg', cmd_window.message_window)
vim.keymap.set('n', '<leader>map', cmd_window.map_window)
vim.keymap.set('n', '<leader>autocmd', cmd_window.autocmd_window)
```
