---
layout: post
title: terminal buffer를 통한 pwd sync
subtitle: terminal buffer를 통한 pwd sync
tags: [vim]
comments: true
author: widehyo
---

- vim의 `:terminal` 커맨드를 사용하면서 알게 된 내용
  - vim의 `:terminal` excommand는 buftype이 'terminal'인 buffer를 연다
    - 확인하는 방법은 getbufvar(<bufnr(of terminal buffer)>, '&buftype') == 'terminal'
  - vim의 terminal-buffer는 커맨드라인을 이용할 수 있는 `Terminal-Job` 모드와
    vim-keybinding을 사용할 수 있는 `Terminal-Normal` 모드를 지원한다.
  - `Terminal-Job`모드에서 `Terminal-Normal`모드로 전환은 키를 이용한다. `(:h terminal-typing)`
    - `<C-\><C-N>`
    - `<C-W>N`(CTRL-W 입력 후 그냥 대문자 N을 입력)
  - `:terminal`로 진입하면 기본적으로 `Terminal-Job`모드로 설정된다.
  - `Terminal-Job`모드에서 인터랙티브 셸을 종료하면 해당하는 terminal buffer가 닫힌다.
  - `Terminal-Job`모드는 `tmap`을 이용하여 제어할 수 있다.
    - `tmap jj <C-W>N`은 `imap jj <ESC>` 만큼 유용하다
      - 다만 git log 등 자체적으로 지원하는 `less` 바인딩에서 네비게이팅하기 어려워 별도의 키바인딩을 사용하는게 좋다.
        - 예시) `tnoremap <C-Q> <C-W>N`
    - `tnoremap <C-S-V> <C-W>"+`를 이용하여 `unnamedplus` 레지스터의 내용을 붙여넣는다
  - 터미널 buffer에서 키입력을 보내고 싶다면 `feedkeys` 함수를 이용한다
    - ex: `call feedkeys("i\<C-u>")`
      - 터미널 버퍼에서 terminal 모드로 진입하고(`i`)
      - 입력된 commandline을 지운다(`<C-u>`)
        - `$ bind -p | grep unix-line-discard` `"\C-u": unix-line-discard`
      - 터미널 버퍼로 전환 후 입력가능한 상태로 만들 때 사용한다.
  - vim에서 open한 `terminal buffer`에서는 `terminal-api`를 이용해 vimscript를 호출할 수 있다.(`:h terminal-api`)
    - 보안상의 이유로 모든 vimscript를 호출시킬수는 없고 vimscript 함수의 이름이 `Tapi_`를 prefix로 가지는 함수만 실행 가능하다
    - 위의 `Tapi_` prefix는 `term_sendapi` 함수를 이용해 prefix를 바꿀 수 있다
    - 호출방법이 매우 비직관적이고 예시를 찾기 어려운데, 다음과 같이 호출한다
    - 먼저 vim을 실행시키고 다음 함수를 작성후 `:source %`로 등록한다
```vim
function Tapi_Test(bufnum, arglist)
  echomsg a:bufnum
  echomsg a:arglist
endfunction
```
    - 그 후 `:terminal`로 `Terminal-Job`모드에서 다음을 실행시킨다.
```bash
$ printf '\e]51;["call","Tapi_Test","asdf"]\x07' 
```
    - vim의 `:messages`에서, `bufnum`과 `arglist`가 제대로 출력되었는지 확인한다.
    - 필자는 vim의 `terminal-api`는 이렇게 사용 방법이 복잡하고 8.2 버전 이상부터 지원되었기 때문에 가장 접근성 및 인지도가 떨어지는 기능 중 하나라고 생각한다.
      - 필자가 생각하는 이유는 다음과 같다
        - 애초에 이 기능을 사용하려면 vimscript를 작성 및 활용할 수 있어야 하는데, neovim이 도입된 이후 구태여 vimscript를 다루는 사람의 수는 매우 적다.
        - vimscript를 다룰 줄 아는 사람은 터미널에서 오랜 시간을 보냈을 가능성이 높다
        - vimscript까지 다루며 vim을 사용하는 이유중 하나로는 최대한의 호환성을 확보가 있다
        - `terminal-api`는 사용하는 방법이 매우 복잡하고, `help` 문서에 실사용 예시 지원이 부족하다
        - 그래서 vim 8.2 이상 버전에 추가된 `terminal-api`는 존재를 모르거나 알아도 호환성 등의 이유로 사용하지 않을 가능성이 있다.
    - 그럼에도 터미널 모드에서 vimscript를 호출할 수 있다는 것은 vim의 buffer 및 window와 같은 내장 객체를 사용할 수 있다는 의미로, 활용성이 뛰어나다

- 다음은 terminal-api를 이용하여 terminal-job 모드에서 파일시스템을 네비게이팅하며 terminal-job 모드의 pwd를 vim의 pwd로 sync하는 예시이다
- 동작원리는 다음과 같다
  - 터미널 진입시 터미널 버퍼에 `setbufvar`를 이용하여 `osc7_dir`를 설정하는 terminal-api 함수를 만든다
```vim
function! Tapi_SetOsc7_Dir(bufnum, arglist)
  call setbufvar(a:bufnum, 'osc7_dir', a:arglist)
endfunction
```
  - 해당하는 함수를 호출하는 커맨드를 확인하고 `.bashrc`에 등록한다
```sh
_setosc7dir() {
  printf '\e]51;["call","Tapi_SetOsc7_Dir","%s"]\x07' "$PWD" # for vim terminal api
}
alias setosc7dir="_setosc7dir"
```
  - 터미널 모드에서 파일시스템을 navigating할 때 항상 해당 함수가 호출될 수 있게끔 `$PROMPT_COMMAND`를 설정한다
```sh
export PROMPT_COMMAND='setosc7dir; '"$PROMPT_COMMAND"
```
  - `$PROMPT_COMMAND`는 모든 실행마다 추가적으로 실행되는 커맨드이므로, 무거운 작업을 등록하기 적절하지 않다.
    - 지금 설정한 작업은 그렇게 무거운 작업은 아니지만, $PWD가 변경되지 않아도 vimscript를 호출하는 비효율성이 있다
    - pwd가 변경되었을 때만 vimscript를 호출하도록 셸 내장 변수 `__osc7_prev_pwd`와 비교하는 로직을 추가한다
```sh
__osc7_prev_pwd=""
_setosc7dir() {
  if [[ "$PWD" != "$__osc7_prev_pwd" ]]; then
    printf '\e]51;["call","Tapi_SetOsc7_Dir","%s"]\x07' "$PWD" # for vim terminal api
    __osc7_prev_pwd="$PWD"
  fi
}
```
  - 이제 터미널 모드에서 파일시스템을 navigating 할 때 자동으로 terminal buffer의 `osc7_dir` 변수가 pwd로 갱신된다
  - 나머지는 해당 변수를 이용하여 `cd`만 호출하면 된다. 이 작업은 vim에서만 이루어진다
```vim
function! SyncTerminalPwd()
  let term_bufnr = bufnr()
  let osc7_dir = getbufvar(term_bufnr, 'osc7_dir')
  if isdirectory(osc7_dir)
    echo 'osc7_dir: ' .. osc7_dir
    execute 'cd ' .. osc7_dir
  endif
endfunction
```
  - 동작을 확인했으므로, 해당 함수를 `TerminalOpen` 이벤트 `autocmd`에 `setlocal`을 이용해 키맵을 설정하자

```vim
function! SetupTerminalOpen() abort
  " <leader>cd : osc7_dir으로 pwd 설정
  execute 'nnoremap <buffer> <leader>cd :call SyncTerminalPwd()<CR>'
endfunction

" TermOpen 이벤트에 대한 자동 명령
augroup TerminalKeymaps
  autocmd!
  autocmd TerminalOpen * call SetupTerminalOpen()
augroup END

```

- 다음으로 이 기능을 발전시켜 보자
- 필자는 `:sh`를 이용하여 vim의 내장 shell을 매우 적극적으로 활용했는데, 기본 동작은 vim의 pwd를 기준으로 interactive shell을 열어준다.
- 위의 터미널 버퍼를 이용한 설정에는 vim의 pwd와 sync하는 부분이 빠져있다.
- 기능을 추가하자
- 방법은 터미널 버퍼를 열 때, `feedkeys`를 이용하여 pwd로 cd하는 명령을 보내는 것이다
  - 터미널 버퍼를 처음 열 때는 기본동작이므로, 열려있는 터미널 버퍼를 감지하여 재활용 할 때 사용한다.
  - pwd는 `getcwd`로 얻을 수 있고, 터미널 버퍼에 해당 path로 cd하는 명령만 추가적으로 보내주면 된다
  - 터미널 버퍼에 키를 보낼 때는 `feedkeys`를 활용한다
    - 단, 이미 열려있는 터미널 버퍼에 명령어가 입력되어 있는 경우가 있으므로 먼저 내용을 먼저 지워준다

```vim
function! OpenTerminal()
  for listed_buffer in filter(getbufinfo(), 'v:val.listed')
    let bufnr = listed_buffer.bufnr
    let buftype = getbufvar(bufnr, '&buftype')
    let buftype = (buftype == '' ? 'normal' : buftype)
    if buftype == 'terminal'
      execute 'buffer! ' .. bufnr
      let pwd = getcwd()
      " sync vim pwd
      call feedkeys("i\<C-u>cd " .. pwd .. "\<CR>")
      return
    endif
  endfor
  execute 'terminal!'

endfunction
```

  - 물론 키바인딩도 추가한다
    - 기존에 오랫동안 `nnoremap <C-D> :sh<CR>`를 이용하여 vim->내장 셸, 내장 셸 -> vim을 토글하는 키로 `<C-D>`키를 사용해 왔으므로, 터미널 버퍼로 같은 동작을 하는 키를 `<C-D>`로 하여 대체한다
      - `nnoremap <C-D> :call OpenTerminal()<CR>`
  - 문제는 다음과 같다
    - `<C-D>`키를 이용한 vim-셸 토글이 작동하지 않는다
      - 이는 `Terminal-Job` 모드에서 `<C-D>`를 입력하면 셸이 종료되며 이를 실행하던 버퍼도 같이 닫는 방식으로 동작하기 때문이다
      - 기존의 사용 경험으로는 다시 이전 vim buffer로 돌아오는 것이 편했으므로 다음 키바인딩을 추가한다
        - `execute 'nnoremap <buffer> <C-D> :buffer! #<CR>'`
      - 이 외, 필요하다고 생각하는 터미널 버퍼 local 설정도 추가한다
        - `setlocal hidden`
        - `setlocal nonumber`
        - `setlocal nolist`
    - `execute buffer` 부분에서 언제나 현재 열려있는 버퍼를 터미널 버퍼로 전환하므로, 터미널 버퍼를 다른 윈도우에 분할해서 사용하는 내 사용 방식에서는 불필요하게 두개의 윈도우가 하나의 터미널 버퍼를 연다.
    - 처음으로 터미널 버퍼를 열 때 `setbufvar`로 `winid`를 지정해두고, 터미널 버퍼를 찾으면 먼저 해당 버퍼에 저장된 `winid`를 이용해 윈도우를 전환한 후 `execute buffer`를 실행하여 해결한다

- 문제를 해결한 시점의 vimscript

`~/.vim/autocmd/terminal.vim`

```vim
function! SetupTerminalOpen() abort
  let term_bufnr = bufnr()
  setlocal hidden
  setlocal nonumber
  setlocal nolist
  " <C-D> : 이전 버퍼로 전환
  execute 'nnoremap <buffer> <C-D> :buffer! #<CR>'

  " <leader>cd : osc7_dir으로 pwd 설정
  execute 'nnoremap <buffer> <leader>cd :call SyncTerminalPwd()<CR>'

  call setbufvar(term_bufnr, 'winid', bufwinid(term_bufnr)) " save winid
endfunction

" TermOpen 이벤트에 대한 자동 명령
augroup TerminalKeymaps
  autocmd!
  autocmd TerminalOpen * call SetupTerminalOpen()
augroup END

```

```vim
function! OpenTerminal()
  for listed_buffer in filter(getbufinfo(), 'v:val.listed')
    let bufnr = listed_buffer.bufnr
    let buftype = getbufvar(bufnr, '&buftype')
    let buftype = (buftype == '' ? 'normal' : buftype)
    if buftype == 'terminal'
      let term_winid = getbufvar(bufnr, 'winid')
      if win_id2win(term_winid) != 0
        " terminal buffer window is opened
        " move cursor to the window
        call win_gotoid(term_winid)
      endif

      execute 'buffer! ' .. bufnr
      let pwd = getcwd()
      " sync vim pwd
      call feedkeys("i\<C-u>cd " .. pwd .. "\<CR>")
      return
    endif
  endfor

  execute 'terminal! ++curwin'

endfunction
```

- 이정도 설정으로 잘 활용하고 있었으나, 사용 중
  - 내장 셸을 그래도 활용해야 하거나
  - 터미널로 전환시 항상 pwd를 sync하는 cd 명령을 보낼 필요가 없거나
    - 예를 들어, `psql`이나 `python`등의 REPL 프로그램을 터미널 버퍼에 실행시키고 있는 경우 해당 버퍼로 전환 할 때 마다 `cd path`가 입력되는 것이 불편했다
  - 1회용으로 잠깐 터미널을 열고 빠져나오는 경우
- 가 있다는 것을 알게 되어 추가 설정에 들어갔다

- 
- `g:open_terminal_mode` 글로별 변수와 모드를 전환하는 키맵을 이용해 해결한다
```vim
let g:open_terminal_mode = 0
nnoremap <space><space><space> :call ToggleOpenTerminalMode()<CR>
```

- `g:open_terminal_mode`는 4가지 값을 가질 수 있다
  - 0: `:sh` 사용
  - 1: `:terminal` 사용 (sync pwd)
  - 2: `:terminal` 사용 (sync pwd 용 cd 명령을 보내지 않음, REPL 작업용)
  - 3: `:vsplit`에서 `:terminal` 사용, 1회용으로 잠깐 command를 실행할 때

- 모드 변경 및 동작방식 확인이 필요하므로 다음 help function을 추가한다

```vim
function! PrintOpenTerminalMode()
  let terminal_modes = [':sh', ':terminal (cd pwd)', ':terminal', ':terminal (vs)']
  let mode_repr_list = []
  for idx in range(len(terminal_modes))
    let t_mode = terminal_modes[idx]
    if g:open_terminal_mode ==# idx
      let t_mode = '< ' .. t_mode .. ' >'
    endif
    call add(mode_repr_list, t_mode)
  endfor
  echomsg join(mode_repr_list, ' | ')
endfunction

function! ToggleOpenTerminalMode()
  let g:open_terminal_mode = (g:open_terminal_mode + 1) % 4
  call PrintOpenTerminalMode()
endfunction
```

- 다음은 메인 로직이다

```vim

function! OpenTerminal()
  call PrintOpenTerminalMode()
  if g:open_terminal_mode == 0
    execute ':sh'
    return
  endif

  if g:open_terminal_mode > 0
    for listed_buffer in filter(getbufinfo(), 'v:val.listed')
      let bufnr = listed_buffer.bufnr
      let buftype = getbufvar(bufnr, '&buftype')
      let buftype = (buftype == '' ? 'normal' : buftype)
      if buftype == 'terminal'
        let term_winid = getbufvar(bufnr, 'winid')
        if win_id2win(term_winid) != 0
          " terminal buffer window is opened
          " move cursor to the window
          call win_gotoid(term_winid)
        endif

        if g:open_terminal_mode == 3 && len(getwininfo()) == 1
          execute 'vsplit'
        endif
        execute 'buffer! ' .. bufnr
        if g:open_terminal_mode == 1
          let pwd = getcwd()
          " sync vim pwd
          call feedkeys("i\<C-u>cd " .. pwd .. "\<CR>")
        endif
        if g:open_terminal_mode == 3 && mode() == 'n'
          call feedkeys("i\<C-u>")
        endif
        return
      endif
    endfor

    if g:open_terminal_mode == 3
      execute 'vsplit'
    endif
    execute 'terminal! ++curwin'
  endif

endfunction
```
