---
layout: post
title: 빔 터미널 API를 이용해보자
subtitle: vim-terminal API to sync terminal buffer pwd to vim's pwd
tags: [vim,terminal,shell]
comments: true
author: widehyo
---

## vim의 terminal buffer 확인방법
`:echo &buftype`이 terminal인지 확인한다.

```vim
function! ShowBufType()
  for listed_buffer in filter(getbufinfo(), 'v:val.listed')
    let bufnr = listed_buffer.bufnr
    let buftype = getbufvar(bufnr, '&buftype')
    let buftype = (buftype == '' ? 'normal' : buftype)
    echo buftype
  endfor
endfunction

call ShowBufType()
```

```vim
function! OpenTerminal()
  for listed_buffer in filter(getbufinfo(), 'v:val.listed')
    let bufnr = listed_buffer.bufnr
    let buftype = getbufvar(bufnr, '&buftype')
    let buftype = (buftype == '' ? 'normal' : buftype)
    if buftype == 'terminal'
      execute 'buffer! ' .. bufnr
      return
    endif
  endfor

  execute 'terminal! ++curwin'
endfunction
```


## terminal-api
- vim의 terminal buffer에서 terminal-api를 이용하여 해당 vim 프로세스에 load된 vimscript
함수를 호출 할 수 있다. `:h terminal-api`
  - 사용하는 방법은 다음과 같다.
    1. `:term`을 이용하여 terminal buffer를 연다
    2. `$ printf '\e]51;["call","$VIMFUNC","$ARGLIST"]\x07'`를 이용해 vim function을
    호출한다. 호출 예:
```bash
$ printf '\e]51;["call","Tapi_test",[]]\007'
$ printf '\e]51;["call","Tapi_test",["hello", 123]]\007'
$ printf '\e]51;["call","Tapi_test","arg"]\007'
```

```vim
function! Tapi_test(argcnt, dir)
  echomsg a:argcnt
  echomsg a:dir
  call system('touch ~/api_occured')
  for item in a:dir
    echomsg item
  endfor
  execute 'cd ' .. a:dir
endfunction
```
  - vim과 shell에 있어서 다음을 확인하였다.
    1. terminal-api는 어떻게 호출하는가?
      - vim에서 :term으로 연 terminal buffer에서 특정 문자열을 printf로 실행한다
      - vim에서 :sh로 이동 연 shell에서는 *해당 명령*을 호출해도 vim 함수를 실행하지 않는다.
      - vim을 실행하지 않은 상태에서 *해당 명령*을 실행해도 vim 함수를 실행하지 않는다.
      - 두 개의 터미널을 각각 열고 각 터미널에서 vim > :term으로 진입한 후 하나의 터미널
      에서 *해당 명령*을 실행해도 그 터미널에서 열린 vim 함수만 실행된다.
      (해당 명령: `$ printf '\e]51;["call","Tapi_test","arg"]\007'`)
  - 이를 이용하여 `:terminal`로 연 터미널 버퍼에서의 pwd를 이용해 vim의 pwd를 sync하는
    함수를 정의하였다.
  - 사용 방법은 터미널버퍼에서 syncpwd를 입력하면 된다.

```vim
function! Tapi_SyncTerminalPwd(param, dir)
  execute 'cd ' .. a:dir
endfunction
```

`~/.bashrc`
```sh
_syncpwd() {
  printf '\e]51;["call","Tapi_SyncTerminalPwd","%s"]\x07' "$PWD"
}
alias syncpwd="_syncpwd"
```



## 최종 파일

`~/.vimrc`
```vim
source ~/.vim/util/common.vim
source ~/.vim/autocmd/terminal.vim
nnoremap <C-D> :call OpenTerminal()<CR>
```

`~/.vim/autocmd/terminal.vim`
```vim
" TermOpen 이벤트에 대한 자동 명령
augroup TerminalKeymaps
  autocmd!
  autocmd TerminalOpen * call s:SetupTerminalKeymaps()
augroup END

function! s:SetupTerminalKeymaps() abort
  " <C-D> : 이전 버퍼로 전환
  execute 'nnoremap <buffer> <C-D> :buffer! #<CR>'

  " gf : 새 윈도우에서 파일 열기
  execute 'nnoremap <buffer> gf <C-W><C-F>'

  " 터미널 모드에서 <C-Q> → <C-\><C-n> : Normal 모드로 진입
  execute 'tnoremap <buffer> <C-Q> <C-\><C-n>'

endfunction
```

`~/.vim/util/common.vim`
```vim
function! Tapi_SyncTerminalPwd(param, dir)
  execute 'cd ' .. a:dir
endfunction
```

`~/.bashrc`
```sh
_syncpwd() {
  printf '\e]51;["call","Tapi_SyncTerminalPwd","%s"]\x07' "$PWD"
}
alias syncpwd="_syncpwd"
```
