---
layout: post
title: 필요한 스크립트를 직접 만들어보자
subtitle: make your own script
tags: [shell]
comments: true
author: widehyo
---

## 기능과 접근성
터미널 환경에서 자주 느끼는 점은 좋은 기능이 있어도 해당 기능에 접근하기 위한 절차가
복잡하면 해당 기능의 존재를 모르거나 사용성이 매우 떨어진다는 점이다. 대표적으로 vim에서
윈도우를 다루는 기능, 소위 <C-W>를 prefix로 하는 여러 기능들을 `nnoremap <C-H> <C-W>h`
등으로 매핑하기 전까지는 거의 사용하지 않았고, 이외 윈도우를 split하는
명령이나(`<C-W><C-S>`, `<C-W><C-V>`) 새로운 버퍼를 이용하여 split하는 명령(`:new<CR>`,
`:vnew<CR>`) 등이 매우 사용성이 높지만 직접 사용하기에는 접근성이 떨어지는 명령이었고,
다음 키매핑을 넣기 전까지는 거의 사용하지 않던 기능이었다.

```vim
" window
nnoremap <C-H> <C-W>h
nnoremap <C-L> <C-W>l
nnoremap <C-J> <C-W>j
nnoremap <C-K> <C-W>k

" window split
nnoremap <leader>ws <C-W><C-S>
nnoremap <leader>wv <C-W><C-V>

" window split new
nnoremap <leader>wns :new<CR>
nnoremap <leader>wnv :vnew<CR>

" window resize
nnoremap <leader>wwr <C-W>=
nnoremap <leader>wwv <C-W>_
nnoremap <leader>wwh <C-W><bar>
```

```lua
-- window
vim.api.nvim_set_keymap('n', '<C-H>', '<C-W>h', silent_noremap)
vim.api.nvim_set_keymap('n', '<C-L>', '<C-W>l', silent_noremap)
vim.api.nvim_set_keymap('n', '<C-J>', '<C-W>j', silent_noremap)
vim.api.nvim_set_keymap('n', '<C-K>', '<C-W>k', silent_noremap)
vim.api.nvim_set_keymap('n', '<leader>ws', '<C-W><C-S>', silent_noremap)
vim.api.nvim_set_keymap('n', '<leader>wv', '<C-W><C-V>', silent_noremap)
vim.api.nvim_set_keymap('n', '<leader>wns', ':new<CR>', silent_noremap)
vim.api.nvim_set_keymap('n', '<leader>wnv', ':vnew<CR>', silent_noremap)
vim.api.nvim_set_keymap('n', '<leader>wwr', '<C-W>=', silent_noremap)
vim.api.nvim_set_keymap('n', '<leader>wwv', '<C-W>_', silent_noremap)
vim.api.nvim_set_keymap('n', '<leader>wwh', '<C-W><bar>', silent_noremap)
```

좋은 기능의 접근성을 높이는 방법은 적어도 터미널 환경에서는 단축키를 지정하는 것 만한
것이 없고, 그래서 필자의 `.bashrc` 및 `.vimrc`, `init.lua` 등의 파일에는 다양한 단축키가
매핑되어 있다.

## 왜 개인 스크립트를 만들게 되었는가
터미널을 사용하는 데에 있어서 여러 층위의 사용자가 있다.

1. 리눅스에 익숙하지 않은 사용자
  - ls, cd, cat 등의 기본 명령어를 이용해 linux 파일시스템을 navigating 할 수 있음
  - ps -ef나 netstat 등의 명령은 사용해본 적이 있으나 그 의미를 알고 사용하지는 않고,
    사용하게 된 이유는 단순히 책/강의에서 보았거나, 현업에서 매뉴얼 혹은 사수 등 인력의
    지시에 의함
  - 기본적인 파일시스템 명령어를 사용할 수 있음(mv, rm, cp, touch, echo)
2. 리눅스에 조금 익숙해진 사용자
  - 리눅스 커맨드라인에서 파일시스템을 자유롭게 탐색 할 수 있음
  - find, grep 등을 이용하여 파일명으로 파일 찾기, 특정 내용을 지닌 파일 찾기를 할 수 있음
  - stdin, stdout, redirection을 알고 있고 활용할 수 있음
  - 파이프를 이용하여 각 명령의 결과를 다음 명령의 input으로 활용할 수 있음
  - echo, env를 이용해 환경변수를 활용할 수 있음
3. 커맨드라인에 익숙해진 사용자
  - cat, sort, find, grep, cut, head, tail, sed, awk 등 커맨드라인 도구에 익숙함
  - du, df, ps, netstat(ss), mount 등을 이용하여 시스템의 자원을 파악할 수 있음
  - 자신의 dot files를 관리하기 시작함

필자는 3번 수준에 머무르다가 최근 bash를 사용할 일이 많아지면서 조금 더 복잡한 명령을
간단하기 실행시킬 준비가 되었다고 느끼게 되었다. 그러면서 bashrc에 alias로 one liner
수준의 명령 뿐 아니라 하나의 shell 파일을 실행시키고 싶다는 생각이 들었고 작업에 착수했다.

방법은 간단하다. 개인 스크립트를 모아둘 디렉터리를 만들고 `$PATH`에 지정하고
`chmod +x`로 실행 권한을 주면 된다.

```sh
mkdir -p $HOME/.cli/bin
# echo 'export PATH=$PATH:$HOME/.cli/bin' >> ~/.bashrc
source ~./bashrc
```

지금까지 가장 잘 활용하고 있는 단축키 중 하나는 날짜에 맞는 md파일을 생성하고 편집하는
커맨드였고, 실제로 가장 많이 사용하고 있는 단축키중 하나이다.

```sh
export PLAY="$HOME/gitclone/playground"
export TODAY="$PLAY/`date '+%Y'`/`date '+%m'`/`date '+%d'`"
export TODAYMD="$TODAY/`date '+%Y%m%d'`.md"

alias cdtoday="cd $TODAY"
alias startplay="mkdir -p $TODAY/ && touch $TODAYMD"
```

```vim
nnoremap <leader>md :e $TODAYMD<CR>
```

일단 터미널을 켜고 아무생각 없이 `startplay`를 먼저 입력하고 `cdtoday`로 이동 후 vim이나
neovim으로 today markdown 파일을 열어 작업 중 기록할 만한 모든 것을 기록한다.

그러다 한 가지 느끼게 된 것이 이렇게 todaymd 파일은 늘었는데, 정작 블로깅할 글을 쓰는
경우은 줄어들었다. 분명 블로깅할 내용은 많이 있는데도 글을 올리지 않는 이유는 낮은
접근성이 가장 큰 원인이라는 생각이 들었고 (post에 맞는 형식 작성, 특정 path에 특정
제목규칙을 가진 파일 작성) 그래서 만든 것이 포스팅용 스크립트이다.

먼저 `.env`파일을 먼저 생성했고 해당 환경설정 파일을 잘 불러오는지 먼저 확인했다.

`touch $HOME/.cli/bin/touchdaily`

```sh
#!/bin/bash


# Print usage message
usage() {
    echo "Usage: touchdaily TITLE" >&2
    exit 1
}

echo $0
echo $1
echo $#

if [[ $# -gt 1 ]]; then
    usage
fi

bindir=$(dirname $0)
echo $bindir
clidir=$(dirname $bindir)
echo $clidir
envdir="$clidir/env"
echo $envdir
envfile="$(dirname $(dirname $0))/env/.env"
echo $envfile

if [[ -f $envfile ]]; then
    echo "-f envfile"
    ls -l $envfile
else
    echo "! -f envfile"
fi
```

이제 envfile을 제대로 찾은 것을 확인하고 셀을 실행시키면서 제대로 작동하는지 확인하면서
작성한 파일은 다음과 같다.

```sh
#!/bin/bash

envfile="$(dirname $(dirname $0))/env/.env"
if [[ -f $envfile ]]; then
    source $envfile
else
    echo "$envfile does not exist" >&2
    exit 1
fi

# Print usage message
usage() {
    echo "Usage: touchdaily TITLE" >&2
    exit 1
}

read -r -p "input post filename: " postfile
read -r -p "input title: " title
read -r -p "input subtitle: " subtitle
read -r -p "input tags: " tags

postfile=${postfile:-default_filename}
title=${title:-default_title}
subtitle=${subtitle:-default_subtitle}
tags=${tags:-}

postfile="$TODAYPOST-$postfile.md"

cat <<EOF >$postfile
---
layout: post
title: $title
subtitle: $subtitle
tags: [$tags]
comments: true
author: widehyo
---
EOF
```

정상 작동을 확인했으므로 더 쉽게 호출할 수 있게 다음을 설정한다.
```sh
alias +x='chmod +x'
alias td='touchdaily'
```

이제 포스팅 글을 많이 올리게 될 것 같다.
