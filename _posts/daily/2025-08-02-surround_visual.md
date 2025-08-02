---
layout: post
title: 네오빔 환경설정 구성기록(2)
subtitle: surround.vim
tags: [vim, neovim]
comments: true
author: widehyo
---

## 08/02
### dot_files 연동
- neovim과 관련된 dot_files를 [dot_files repository](https://github.com/widehyo1/widehyo1.github.io.git)를 이용해서 관리하고 있다.
- tar의 옵션 중 --from-files를 이용한다.
  - tar --from-files는 압축할 파일/디렉터리를 커맨드라인으로 넘기는 대신, 해당 목록이 적힌 파일 디스크립터를 전달하는 옵션이다.
    - 사용예: `tar -czf a.tgz 1.txt 2.txt` `==` `tar -cvf a.tgz --from-files list.txt`
    - where `echo 1.txt > list.txt` `echo 2.txt >> list.txt` 
  - 상대경로를 이용하여 파일을 압축하고 다른 디렉터리에서 해당 압축파일을 풀면, 해당 디렉터리를 기준으로 압축이 풀린다.
    - 이를 이용하면 압축을 이용한 파일 시스템 복사-붙여넣기를 할 수 있다.
    - `git status` `git log --oneline --name-only` 혹은 `find`와 `grep`을 이용해 얻은 대상 파일 목록을 `list.txt`에 잠시 저장한다.
      - 혹은 추가적인 작업이 필요하지 않다면 바로 `--from-files -`을 통해 stdin을 그대로 사용할 수 도 있다.
    - 원하는 형식이 아니거나 수정이 필요하면 `list.txt`를 vim으로 편집하여 원하는 파일 목록을 얻는다.
      - 이 방법으로 파일시스템을 복사할 때 주의할 점은 압축 해제가 새로운 디렉터리에서 제대로 실행되도록 상대경로를 사용해야한다는 것이다.
    - `tar -czvf transfer.tgz --from-files list.txt`
    - `cd $TARGET_PATH && cp $OLDPWD/transfer.tgz . && tar -xzvf transfer.tgz`
    - 원하는 경로로 이동하고, 상대경로를 이용한 압축파일을 가져와서 압축을 푼다.
- 아래 예시에서는 절대경로를 그대로 사용하지만, 원하는 경우 환경변수를 이용하면 보다 유연하게 사용 가능하다.
- `cddot && git pull && bash dot_files/dot_cli/shell/apply_nvimconf.sh` 로 현재 작업한 neovim 설정 파일의 `.vim`, `.lua`를 dot_files repository에 적용한다.
- `cddot bash dot_files/dot_cli/shell/apply_nvimconf_from_dot.sh` 로 dot_files repository에 설정한 neovim 설정 내용을 현재 머신에 적용한다.

`dot_files/dot_cli/shell/apply_nvimconf.sh`
```bash
!#/bin/bash
echo find /home/widehyo/.config/nvim -type f | grep -E ".lua$|.vim$" | cut -c28- | tar -czvf /home/widehyo/gitclone/dot_files/dot_config/nvim/nvimconf.tgz --directory /home/widehyo/.config/nvim/ --files-from -
find /home/widehyo/.config/nvim -type f | grep -E ".lua$|.vim$" | cut -c28- | tar -czvf /home/widehyo/gitclone/dot_files/dot_config/nvim/nvimconf.tgz --directory /home/widehyo/.config/nvim/ --files-from -
echo tar -xzvf /home/widehyo/gitclone/dot_files/dot_config/nvim/nvimconf.tgz --directory /home/widehyo/gitclone/dot_files/dot_config/nvim
tar -xzvf /home/widehyo/gitclone/dot_files/dot_config/nvim/nvimconf.tgz --directory /home/widehyo/gitclone/dot_files/dot_config/nvim
echo rm /home/widehyo/gitclone/dot_files/dot_config/nvim/nvimconf.tgz
rm /home/widehyo/gitclone/dot_files/dot_config/nvim/nvimconf.tgz
```

`dot_files/dot_cli/shell/apply_nvimconf_from_dot.sh`
```bash
!#/bin/bash
echo find /home/widehyo/gitclone/dot_files/dot_config/nvim -type f | grep -E ".lua$|.vim$" | cut -c50- | tar -czvf /home/widehyo/.config/nvim/nvimconf.tgz --directory /home/widehyo/gitclone/dot_files/dot_config/nvim/ --files-from -
find /home/widehyo/gitclone/dot_files/dot_config/nvim -type f | grep -E ".lua$|.vim$" | cut -c50- | tar -czvf /home/widehyo/.config/nvim/nvimconf.tgz --directory /home/widehyo/gitclone/dot_files/dot_config/nvim/ --files-from -
echo tar -xzvf /home/widehyo/.config/nvim/nvimconf.tgz --directory /home/widehyo/.config/nvim
tar -xzvf /home/widehyo/.config/nvim/nvimconf.tgz --directory /home/widehyo/.config/nvim
echo rm /home/widehyo/.config/nvim/nvimconf.tgz
rm /home/widehyo/.config/nvim/nvimconf.tgz
```

### markdown info_string paste
- 대부분의 노트는 markdown 형식으로 기록한다.
- 모든 내용을 타이핑하기 보다는 터미널의 출력이나 시스템 클립보드에 복사한 markdown에 붙여넣는 경우가 많다.
- 이때, 복사한 내용을 markdown의 info_string (`bash`, `lua`, `vim` 등)으로 감싸진 code block에 붙여넣는 경우가 많다.
- 붙여넣을 때 info_string을 입력하면, 해당 info_string을 가진 code block을 만들고 그 안에 붙여넣는 기능을 만들었다.
- 복사할 내용 뿐 아니라 info_string을 사용자 입력으로 받아야 하기 때문에 `vim.api.nvim_create_user_command`의 `nargs = "?"`를 이용하여 info_string을 입력받는다.

`ftplugin/markdown.lua`
```lua
function paste_code_block(info_string)
  vim.fn.setline('.', '```' .. (info_string or ''))
  vim.cmd('put')
  vim.fn.setline(vim.fn.line('.') + 1, '```')
end

vim.api.nvim_create_user_command(
  'PasteCodeBlock',
  function(opts)
    local info_string = opts.fargs[1]
    paste_code_block(info_string)
  end,
  {
    nargs = "?",
    desc = "Paste codeblock with infomation string"
  }
)

vim.keymap.set('n', '<space>p', ':PasteCodeBlock ')
```

### surround-visual 포팅
- 진행 중
