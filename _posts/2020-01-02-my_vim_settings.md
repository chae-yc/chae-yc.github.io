---
title: "My Vim Settings"
categories:
  - Application
tags:
  - vim
  - settings
last_modified_at: 2020-01-02T01:21:00+09:00
toc: false
---

현재 맥에서 사용중인 vim 설정(`~/.vimrc`)이다.

```vim
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim' "required
Plugin 'tpope/vim-fugitive' "required
call vundle#end()
filetype plugin indent on " Put your non-Plugin stuff after this line

Plugin 'nathanaelkane/vim-indent-guides'
Plugin 'scrooloose/syntastic'

"Syntax
if has("syntax")
    syntax on
endif

set autoindent
set cindent

set nu

set ts=4 " Tab 너비
set shiftwidth=4 " 자동 인덴트할 때 너비
set expandtab

" 마지막으로 수정된 곳에 커서를 위치함
au BufReadPost *
\ if line("'\"") > 0 && line("'\"") <= line("$") |
\ exe "norm g`\"" |
\ endif

" colorscheme Base2Tone_EveningDark
```

colorscheme 을 사용하고 싶으면 아래의 파일을 받아서 `~/.vim/colors` 에 저장한 후 colorscheme 앞에 있는 주석(`"`)을 해제한다.

[Base2Tone Git /Theme](https://github.com/atelierbram/Base2Tone-vim/blob/master/autoload/airline/themes/Base2Tone_EveningDark.vim)
