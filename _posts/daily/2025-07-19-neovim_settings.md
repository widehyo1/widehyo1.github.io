---
layout: post
title: 네오빔 환경설정 구성기록
subtitle: neovim configs
tags: [neovim]
comments: true
author: widehyo
---

## 07/19
### buffer menu 포팅
#### 기존 vimscript

기존 vimscript 버전의 경우 vim8부터 지원된 popup_menu를 이용하여 현재 열려있는 버퍼 목록을 조회하고, 선택하여 불러오는 방식으로 구현했었다. 단, vimscript에서는 closure가 지원되지 않아, 이 작업을 하는 데 공통으로 활용해야 하는 변수는 script-scoped 변수인 s:buf_dict를 사용해야 했던 점이 아쉬웠다.

자세한 동작방식은 다음과 같다.
1. getbufinfo() API를 통해 session에서 buffer 정보 조회
2. 이렇게 얻은 각 bufinfo는 너무 많은 정보를 담고 있으므로, popup에서 보여줄 텍스트와 해당하는 버퍼로 이동하기 위한 buffer number만 추출하여 buf_dict로 저장(이 과정에서 path를 보기 좋게 보여주기 위해 expand(":.:~") 적용
  - expand(path, "format_str")은 path를 변형하는데 자주 사용되며, `:.`은 :pwd 기준으로 찾아갈 수 있다면 해당 상대경로를, `:~`는 홈 디렉터리를 기준으로 찾아갈 수 있다면 absoulte path 대신 ~를 포함한 경로로 변환해 주는 modifier다.
3. buf_dict의 수가 0이면 warning popup을 띄우고 바로 닫음
4. buf_dict가 수가 1이상이면 각 path를 popup_menu에 보여주고 선택시 선택한 메뉴의 번호(`result`)를 callback(`LoadBuffer`)으로 받아 buf_dict용 인덱스로 변환하고(vim의 dictionary는 0-based, popup_menu의 callback에 전달되는 `result`는 1-based) 해당하는 아이템의 bufnr을 이용하여 버퍼 이동(`execute 'buffer! ' . target_buffer.bufnr`)
5. 만약 search_text가 주어진다면 popup menu에서 보여지는 path에서 해당하는 search_text가 matching되는 buffer정보만 필터링해서 조회

`~/.vimrc`
```vim
" load custom script
source ~/.vim/util/common.vim

nnoremap <silent> <leader><leader><leader> :call BufferMenu()<CR>
command -nargs=1 BufferMenu :call BufferMenu(<f-args>)
nnoremap <leader><leader>s :BufferMenu 
```


`~/.vim/util/common.vim`
```vim
function! BufferMenu(search_text = '')
  " show loaded buffers on popup menu and open selected buffer
  let s:buf_dict = map(filter(getbufinfo(), 'v:val.listed'), '#{
        \ bufnr: v:val.bufnr,
        \ text: fnamemodify(expand(v:val.name), ":.:~")
        \ }')
  if len(a:search_text)
    " filter buf_dict text with search_text
    call filter(s:buf_dict, 'v:val.text =~ a:search_text')
    if len(s:buf_dict) == 0
      let popup_config = #{
      \   time: 3000,
      \   cursorline: 0,
      \   highlight: 'WarningMsg'
      \ }
      let empty_msg = 'there is no buffer with name matching ' . a:search_text
      call popup_menu(empty_msg, popup_config)
      return
    endif
  endif

  let popup_config = #{
  \   callback: 'LoadBuffer'
  \ }
  call popup_menu(s:buf_dict, popup_config)
endfunction

function! LoadBuffer(id, result)
  let target_buffer = s:buf_dict[a:result - 1]
  execute 'buffer! ' . target_buffer.bufnr
endfunction
```

#### neovim용 lua version
lua에서는 map, filter를 언어차원에서 제공하지 않으므로, util 함수를 먼저 만든다.

```lua
-- ~/.config/nvim/lua/util/lua.lua
local M = {}

function M.filter(tbl, predicate)
  local filtered = {}
  for _, v in ipairs(tbl) do
    if predicate(v) then
      table.insert(filtered, v)
    end
  end
  return filtered
end

function M.filter_dict(tbl, predicate)
  local filtered = {}
  for k, v in pairs(tbl) do
    if predicate(v) then
      filtered[k] = v
    end
  end
  return filtered
end

function M.map(tbl, mapper)
  local mapped = {}
  for _, v in ipairs(tbl) do
    table.insert(mapped, mapper(v))
  end
  return mapped
end

function M.map_dict(tbl, mapper)
  local mapped = {}
  for k, v in pairs(tbl) do
    mapped[k] = mapper(v)
  end
  return mapped
end

function M.apply(tbl, mapper)
  for i, v in ipairs(tbl) do
    tbl[i] = mapper(v)
  end
  return tbl
end

function M.apply_dict(tbl, mapper)
  for k, v in pairs(tbl) do
    tbl[k] = mapper(v)
  end
  return tbl
end

return M
```

단, 이렇게 하면 코드를 python에서의 map, filter처럼 사용해야 하므로, 코드가 지저분해진다. 물론, lua는 coroutine을 이용한 generator를 만들수도 있지만 python에 비해 정의하고 사용하는데 큰 공수가 들어가므로, java의 method chaining처럼 사용할 수 있는 방안을 구상하여 사용했다. 이런 식으로 디폴트 동작을 바꾸는 방법을 사용할 경우 lua의 metatable을 이용하면 된다. 접근방식만 보면 javascript의 prototype에 새로운 메서드를 추가하는 것과 유사하다.

```lua
-- ~/.config/nvim/lua/util/chain.lua
local M  = {}

local Chain = {}
Chain.__index = Chain -- 메타테이블 설정: Chain 테이블에서 메소드를 찾도록 함

-- 생성자 함수 (새로운 Chain 인스턴스를 만듭니다)
function Chain.new(data)
  local self = {
    _data = data or {} -- 내부적으로 데이터를 저장할 필드
  }
  return setmetatable(self, Chain)
end

-- 필터링 메소드
function Chain:filter(predicate)
  local new_data = {}
  for _, v in ipairs(self._data) do
    if predicate(v) then -- filter는 키와 값 모두 받도록 유연하게
      table.insert(new_data, v)
    end
  end
  self._data = new_data -- 필터링된 데이터로 업데이트
  return self           -- 중요: self를 반환하여 체이닝 가능하게 함
end

-- 필터링 메소드
function Chain:filter_dict(predicate)
  local new_data = {}
  for k, v in pairs(self._data) do
    if predicate(v) then -- filter는 키와 값 모두 받도록 유연하게
      new_data[k] = v
    end
  end
  self._data = new_data -- 필터링된 데이터로 업데이트
  return self           -- 중요: self를 반환하여 체이닝 가능하게 함
end

-- 매핑 메소드 (새로운 값을 생성)
function Chain:map(mapper)
  local new_data = {}
  for i, v in ipairs(self._data) do
    new_data[i] = mapper(v) -- map은 값만 받도록 단순하게
  end
  self._data = new_data
  return self
end

-- 매핑 메소드 (새로운 값을 생성)
function Chain:map_dict(mapper)
  local new_data = {}
  for k, v in pairs(self._data) do
    new_data[k] = mapper(v) -- map은 값만 받도록 단순하게
  end
  self._data = new_data
  return self
end

-- 적용 메소드 (데이터를 제자리에서 수정)
function Chain:apply(mapper)
  for i, v in ipairs(self._data) do
    self._data[i] = mapper(v)
  end
  return self
end

-- 적용 메소드 (데이터를 제자리에서 수정)
function Chain:apply_dict(mapper)
  for k, v in pairs(self._data) do
    self._data[k] = mapper(v)
  end
  return self
end

-- 현재 데이터를 가져오는 메소드 (체이닝의 끝)
function Chain:get()
  return self._data
end

function M.from(tbl)
  return Chain.new(tbl)
end

return M
```

neovim에서 제공하는 마음에 드는 기능 중 하나인 floating window를 앞으로도 많이 사용하게 될 것 같으므로 floating window 및 buffer, window를 관리할 buf.lua도 만들었다. 간단하게 두가지 메서드를 담고 있고 각각은 다음과 같다.
- floating_window
  - lines와 field를 받아 floating window에 해당 content를 표출한다(최소 너비 80, 최소 높이 20) 
  - 초기에는 lines에 content를 담은 배열만 전달하도록 설계했으나, object의 list 중에 표출할 field를 설정하는 방식으로 동작하는 것이 더 유연하고, 무엇보다 기존 vimscript의 popup_menu가 `text` 필드를 이용하여 이미 그런식으로 동작하므로 field 를 추가하여 lines가 table인 경우 각 element의 field 를 이용하여 content를 구성하도록 변경했다.
- add_floating_window_callback
  - floating window를 띄우는 것과 그 동작을 제어하는 것은 다른 메서드에 있어야 재사용성이 높아질 것이라는 생각으로 만든 메서드
  - 기본 동작은 엔터키 입력시 버퍼를 닫는 기능을 추가한다. (floating window를 popup menu로 사용하는 workflow를 고려하는 메서드)
  - 초기에는 pre_callback만 넣어두었으나 이후 floating 윈도우가 닫힌 이후에 동작을 추가할 필요성을 느껴 post_callback 함수도 추가했다.
    - pre_callback에서는 floating window가 열려있는 상태에서 `vim.api.nvim_get_current_line()`나 `vim.fn.line('.')` 로 커서가 위치한 라인넘버 / 라인 내용에 접근할 필요가 있었고
    - post_callback에서는 `vim.api.nvim_win_close(win, true)`로 floating window가 닫힌 시점에서 동작을 제어하기 위해 추가했다.
  - 위와 같은 내용을 고려하게 된 이유는 buffer_menu의 메커니즘이 다음과 같이 이루어지기 때문이다.
    1. getbufinfo를 이용하여 버퍼 정보를 load하고 적당히 조작하여 floating window에 표출할 항목을 구성
    2. 해당 항목(`buffers`)를 이용하여 floating window open
    3. floating window 내에서 원하는 버퍼 선택 및 선택지 정보 반환 <- pre_callback
    4. floating close
    5. 원래 버퍼로 돌아와 반환된 선택지 정보에서 버퍼넘버를 추출하여 버퍼이동 <- post_callback
  - 위의 메커니즘을 이용하지 않고 pre_callback만 있다면 floating window 자체를 선택한 버퍼로 변경한 후 닫게 되어 사용자는 버퍼 이동 기능을 사용할 수 없다.
  - post_callback만 있다면 floating window에서 어떤 버퍼를 선택하였는지 정보를 전달할 수 없다. (global 변수나 register로 우회는 가능하겠으나, lua로 다시 작성하면서 마음에 들었던 점이 closure를 이용한 script variable 제거였기 때문에 논외)
  - 기능 요구사항을 충족하고 function signature를 결정하는 데는 다음을 고려하였다.
    - 막상 만들고보니 하나의 함수에서 모두 제어할 수 있는 편이 좋을 것 같아 pre_callback과 post_callback을 optional한 parameter로 두기로 결정
    - 대부분의 경우 post_callback이 필요한 경우는 선택지를 선택한 경우일 것이므로 post_callback에는 pre_callback에서 선택한 item 정보를 parameter로 전달하기로 결정
    - pre_callback에서 선택지를 선택하지 않고 post_callback만 호출하는 경우는 고려되어있지 않은데, 그럴 만한 경우가 생기면 그 때 다시 수정하기로 결정

```lua
-- ~/.config/nvim/lua/util/buf.lua

local M = {}

function M.floating_window(lines, field)
  local max_line_width = 0
  local contents = {}

  -- set content-extracting function
  local get_content = function(line) return line end
  if field and field ~= "" then
    get_content = function(line) return line[field] end
  end

  for _, line in ipairs(lines) do
    local content = get_content(line)
    table.insert(contents, content)
    max_line_width = math.max(max_line_width, vim.fn.strwidth(content))
  end

  local buf = vim.api.nvim_create_buf(false, true)

  vim.api.nvim_buf_set_lines(buf, 0, -1, false, contents)

  local width = math.max(max_line_width + 2, 80)
  local height = math.max(#lines, 20)

  local win = vim.api.nvim_open_win(buf, true, {
    relative = 'editor',
    width = width,
    height = height,
    row = math.floor((vim.o.lines - height) / 2),
    col = math.floor((vim.o.columns - width) / 2),
    style = 'minimal',
    border = 'rounded',
  })

  vim.api.nvim_buf_set_option(buf, 'modifiable', false)
  vim.cmd("setlocal cursorline")

  return win, buf
end

function M.add_floating_window_callback(win, buf, pre_callback, post_callback)
  local item = nil
  vim.keymap.set('n', '<CR>', function()
    if pre_callback then
      item = pre_callback()
    end
    vim.api.nvim_win_close(win, true)
    if post_callback then
      post_callback(item)
    end
  end, { buffer = buf })
end

return M
```


메인로직은 다음과 같다.

```lua
-- ~/.config/nvim/lua/common/init.lua

-- Buffer menu popup
function M.buffer_menu(search_text)
  local buf_listed = function(buf) return buf.listed == 1 end
  local bufnr_relpath = function(buf)
    return {
      bufnr = buf.bufnr,
      path = vim.fn.fnamemodify(buf.name, ":.:~")
    }
  end
  local search_match = function(buf) return buf.path:match(search_text) end
  local buffers = chain.from(vim.fn.getbufinfo())
    :filter(buf_listed)
    :apply(bufnr_relpath)
    :get()

  if search_text and search_text ~= "" then
    buffers = chain.from(buffers)
      :filter(search_match)
      :get()
    if #buffers == 0 then
      local empty_msg = "there is no buffer with name matching <" .. search_text .. ">"
      local win, buf = buf_util.floating_window({empty_msg})
      buf_util.add_floating_window_callback(win, buf)
      return
    end
  end

  local select_buffer = function ()
    return buffers[vim.fn.line(".")]
  end

  local load_buffer = function (item)
    vim.cmd("buffer! " .. item.bufnr)
  end

  local win, buf = buf_util.floating_window(buffers, 'path')
  buf_util.add_floating_window_callback(win, buf, select_buffer, load_buffer)

end
```

그리고 키 바인딩은 이렇게 해서 사용한다.


```lua
-- load scripts
local common = require('common')
vim.keymap.set('n', '<leader><leader><leader>', common.buffer_menu)
vim.api.nvim_create_user_command(
  'BufferMenu',
  function(opts)
    local search_text = opts.fargs[1]
    common.buffer_menu(search_text)
  end,
  {
    nargs = 1,
    desc = "Open buffer menu with optional search text"
  }
)
vim.keymap.set('n', '<leader><leader>s', ':BufferMenu ')
```


잘 동작한다.
