# vim

## .vimrc

```bash
set number
set tabstop=4
set shiftwidth=4
set expandtab
set ai
syntax on
set backspace=indent,start,eol
nnoremap <C-J> <C-W><C-J>
nnoremap <C-K> <C-W><C-K>
nnoremap <C-L> <C-W><C-L>
nnoremap <C-H> <C-W><C-H>

autocmd BufNewFile *.sh exec ":call SetTitle()"

func SetTitle()
    if expand("%:e") == 'sh'
        call setline(1, "#!/bin/bash")
        call setline(2, "#")
        call setline(3, "#*******************************************************************")
        call setline(4, "# Author:                luxinyu")
        call setline(5, "# Data:                  ".strftime("%Y-%m-%d"))
        call setline(6, "# FileName:              ".expand("%"))
        call setline(7, "# Description: ")
        call setline(8, "# Copyright (C):         ".strftime("%Y")." All rights reserved")
        call setline(9, "#*******************************************************************")
        call setline(10, "")
    endif
endfunc
autocmd BufNewFile * normal G
```

