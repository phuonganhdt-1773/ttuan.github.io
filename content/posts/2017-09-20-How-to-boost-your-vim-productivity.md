---
layout: post
title: Mouse Free - How to boost your Vim productivity
categories:
- Rails
tags:
- Mouse Free
- Vim
---

# Mở đầu
Khi bạn sử dụng Vim (dù mới bắt đầu hay đã làm quen một thời gian), có lẽ bạn đều gặp các vấn đề, khó khăn hay một vài điều khiến bạn cảm thấy không được thuận tiện cho lắm. Hay sử dụng Vim không thực sự làm tăng được tốc độ edit text cua ban? Nếu bạn đang gặp những vấn đề như vậy, bài viết dưới đây là một *gợi ý* cho bạn để bạn có thể cải thiện hiệu năng khi làm việc với Vim ;)

# Boost your Vim Productivity
## I. Space is your Leader
`Leader` là một ý tưởng rất tuyệt vời. Nó cho phép chúng ta thực hiện được những action bằng một *chuỗi* các key, thay vì sử dụng *tổ hợp* các phím. Vì mỗi khi mình sử dụng chúng, mình bắt buộc phải gõ tổ hợp phím `Ctrl-something` mới có thể làm được.

Trong một thời gian dài, mình sử dụng `,` để làm `Leader` key. Sau đó, mình nhận ra răng `Space` là một lựa chọn tốt hơn rất nhiều.

```
# .vimrc
let mapleader = "\<Space>"
```

Việc mapping này đã làm mình có rất nhiều trải nghiệm mới với Vim. Bây giờ, mình có thể bấm `Leader` bằng ngón cái, và các ngón còn lại có thể thoải mái đặt ở hàng home. `Leader` trở nên dễ sử dụng hơn rất nhiều.

## II. Map your most frequent actions to Leader

Đầu tiên, bạn hãy nghĩ tới những actions mà bạn hay dử dụng nhất khi làm việc với Vim. Sau đó, hãy map chúng, sử dụng `Leader` key. Dưới đây là một số mapping của mình.
#### Type `<Space>o` để mở một file mới.

```
nnoremap <Leader>o :CtrlP<CR>
```
#### Type `<Space>w` để lưu file (sẽ nhanh hơn rất nhiều so với `:w<Enter>`)

```
nnoremap <Leader>w :w<CR>
```

#### Copy and paste vào system clipboard với `<Space>p` và `<Space>y`

```
vmap <Leader>y "+y
vmap <Leader>d "+d
nmap <Leader>p "+p
nmap <Leader>P "+P
vmap <Leader>p "+p
vmap <Leader>P "+P
```
...
Việc map các actions phải sử dụng thường xuyên sẽ giúp tiết kiệm rất nhiều thời gian cho bạn ;)

## III. Use region expanding

Mình sử dụng [terryma/vim-expand-region](https://github.com/terryma/vim-expand-region) với một vài mapping như sau:

```
vmap v <Plug>(expand_region_expand)
vmap <C-v> <Plug>(expand_region_shrink)
```
Nó cho phép mình thực hiện được các thao tác:
* Gõ `v` để select 1 kí tự.
* Gõ `v` tiếp 1 lần nữa để expand selection thành 1 word.
* Gõ `v` tiếp 1 lần nữa để expand thành 1 đoạn văn.
...
* Gõ `<C-v>` để trở về selection trước đó nếu bạn gõ quá tay =)

Có vẻ việc gõ `vvv` sẽ chậm hơn `vp`. Tuy nhiên, trong thực tế, việc bạn gõ `vvv` sẽ nhanh hơn, vì bạn không cần mất thời gian suy nghĩ trước khi gõ, suy nghĩ xem sẽ chọn gì (1 từ hay 1 đoạn văn hay 1 dòng, ...), và sẽ phải sử dụng tổ hợp phím nào.

Bằng cách này, bạn gõ `v` thay thế cho `viw`, `vaw`, `vi"`, `vi(`, `va(`, `vi[`, `vi{`, `va{`, `vat`, ...... Rất tuyệt đúng không :D

## IV. Discover text search object

Mình chưa bao giờ thực sự thích cách search-and-replace trong Vim cho tới khi mình tìm được 1 snippet trong [Vim wiki](http://vim.wikia.com/wiki/Copy_or_change_search_hit):

```
vnoremap <silent> s //e<C-r>=&selection=='exclusive'?'+1':''<CR><CR>
    \:<C-u>call histdel('search',-1)<Bar>let @/=histget('search',-1)<CR>gv
omap s :normal vs<CR>
```

Nó cho phép mình sử dụng flow dưới đây để search-and-replace:
* Search từ, sử dụng `/something`
* Gõ `cs`, replace first match, gõ `<Esc>`
* Gõ `n.n.n.n.n.` để review và replace tất cả các matches

P.S. Có 1 cách khác đó là dùng `cgn` [from Vim 7.4](http://vimcasts.org/episodes/operating-on-search-matches-using-gn/)

## V. Invent more awesome key mappings

Mình sử dụng những shortcuts dưới đây hàng ngày. Chúng đã giúp mình tiết kiệm được rất nhiều thời gian.

**Tự động nhảy tới cuối đoạn text bạn vừa paste:**

Mình có thể paste nhiều dòng nhiều lần với `ppppp`.

```
vnoremap <silent> y y`]
vnoremap <silent> p p`]
nnoremap <silent> p p`]
```

**Ngăn cản việc thay thế paste buffer khi paste:**
Mình có thể lựa chọn đoạn text và paste chúng mà không phải lo lắng gì nếu paste buffer bị thay thế bằng đoạn text vừa mới removed đi. (Đoạn này nên đặt cuối của file `~/.vỉmc`)

```
" vp doesn't replace paste buffer
function! RestoreRegister()
  let @" = s:restore_reg
  return ''
endfunction
function! s:Repl()
  let s:restore_reg = @"
  return "p@=RestoreRegister()\<cr>"
endfunction
vmap <silent> <expr> p <sid>Repl()
```

**Nhanh chóng di chuyển tới đầu/cuối dòng**

* Gõ `12<Enter>` để tới dòng 12
* Go Enter để tới cuối dòng
* Gõ Backspace để tới đầu dòng.

```
nnoremap <CR> G
nnoremap <BS> gg
```

**Select nhanh đoạn text bạn vừa paste**

```
noremap gV `[v`]
```

## VI. Make your unit testing experience seamless

Mình sử dụng [vim-vroom](https://github.com/skalnik/vim-vroom) và config tmux để chạy tests.

Vì `vim-room` sử dung `<Leader>r` để chạy các test suite, và mình sử dụng `<Space>` làm `Leader` key nên mình chỉ cần gõ `<Space>r` để chạy test.

Các test sẽ chạy trên tmux *split* nên mình có thể nhìn code, chạy test trogn khi vẫn có thể code tiếp từng phần một. ;)

## VII. Use Ctrl-Z to switch back to Vim

Mình thường phải chạy lệnh trên shell. Để có thể thực hiện được điều này, mình pause Vim bằng cách bấm `Ctrl-z`, gõ command và sử dung `fg<Enter>` để switch back về Vim.

Tuy nhiên, mình thấy `fg` không thuận tiện lắm. Mình muốn gõ `Ctrl-z` 1 lần nữa để quay lại Vim. Do không tìm đươc giải pháp nào thích hợp, nên mình có viết đoạn code này để có thể thực hiện với ZSH:

```
fancy-ctrl-z () {
  if [[ $#BUFFER -eq 0 ]]; then
    BUFFER="fg"
    zle accept-line
  else
    zle push-input
    zle clear-screen
  fi
}
zle -N fancy-ctrl-z
bindkey '^Z' fancy-ctrl-z
```

Bạn hãy đặt đoạn code này trong `~/.zshrc`, `source` lại file này và trải nghiệm nhé.

## VIII. Setup Tmux the Right Way

Tmux + OS X + Vim là một tổ hợp khá khó sử dụng vì:

* poor system clipboard handling.
* Khó khăn trong việc di chuyển giữa cửa sổ Vim và Tmux
* Khó thực hiện tmux commands (`C-b`)
* Khó để sử dụng copy mode trong Tmux

Mình đã dành khá nheiefu thời gian để giải quyết các vấn đề này. :3

#### Sử dung `<C-Space>` làm tmux prefix.

Một vài người sử dụng `<C-a>` làm prefix, nhưng mình thấy sử dụng `<C-Space>` tạo nhiều thuận tiện hơn.

```
unbind C-b
set -g prefix C-Space
bind Space send-prefix
```

#### Sử dung `<Space>` để vào copy mode

Hãy nghĩ về việc sử dụng `<C-Space><Space>` để bạn có thể bắt đầu copy mode trong tmux.

```
bind Space copy-mode
bind C-Space copy-mode
```

#### Sử dụng `y` và `reattach-to-user-namespace` (trong OSX)

Để copy vào system clipboard, bạn sẽ phải chạy lệnh `brew install reattach-to-user-namespace` trước :D

```
bind-key -t vi-copy y \
  copy-pipe "reattach-to-user-namespace pbcopy"
```

Từ giờ, bạn có thể switch giữa các cửa sổ của vim và tmux bằng cách sử dụng `<C-h>`, `<C-j>`, `<C-k>`, `<C-l>`.

Mình khuyên các bạn nên sử dụng keybinding dưới đây để split tmux ưindows với `<C-Space>l` và `<C-Space>j` sẽ nhanh hơn bạn sử dụng `<C-Space>%` và `<C-Space>|`.

```
bind j split-window -v
bind C-j split-window -v

bind l split-window -h
bind C-l split-window -h
```

See more [tmux.conf](https://github.com/sheerun/dotfiles/blob/master/tmux.conf)

## IX. Make Ctrl-P plugin a lot faster for Git projects

Đặt đoạn config dưới đây trong file `.vimrc` sẽ khiến cho `Ctrl-P` của bạn chạy nhanh hơn rất nhiều. (config để CtrlP sử dụng `git` hoặc silver searcher cho việc autocompletion)

```
let g:ctrlp_use_caching = 0
if executable('ag')
    set grepprg=ag\ --nogroup\ --nocolor

    let g:ctrlp_user_command = 'ag %s -l --nocolor -g ""'
else
  let g:ctrlp_user_command = ['.git', 'cd %s && git ls-files . -co --exclude-standard', 'find %s -type f']
  let g:ctrlp_prompt_mappings = {
    \ 'AcceptSelection("e")': ['<space>', '<cr>', '<2-LeftMouse>'],
    \ }
endif
```
Mình có recommend sử dung [vim-scripts/gitignore](https://github.com/vim-scripts/gitignore)

## X. Use package manager

[neobundle.vim](https://github.com/Shougo/neobundle.vim) là một plugin tuyệt vời để quản lý Vim plugin.

Tuy nhiên, bạn cũng có thể sử dụng các package manager khác như Vundle, Vim Plug, ....

## XI. Take advantage of Vim plugins

Dưới đây là 1 số plugins mình hay sử dụng:

```
Plug 'Valloric/YouCompleteMe'
Plug 'tpope/vim-commentary'
Plug 'tpope/vim-repeat'
Plug 'kien/ctrlp.vim'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'christoomey/vim-tmux-navigator'

"\\ For Rails developer
Plug 'tpope/vim-rails'
Plug 'tpope/vim-fugitive'
Plug 'airblade/vim-gitgutter'
Plug 'thoughtbot/vim-rspec'
```

## XII. Speed-up setup of Vim on your server
Mình thường phải sử dụng Vim ở trên server. Thật k may là Vim thường bị miss 1 số config (nếu mình thay đổi/ config thêm 1 vài thứ ở các máy local mà mình làm việc).

MÌnh có thể sử dụng [vim-sensible](https://github.com/tpope/vim-sensible) nhưng điều đó là chưa đủ. Mình có phát triển [vimrc](https://github.com/sheerun/vimrc) plugin với nhiều default option, biến file `~/.vimrc` thành một single source khi config Vim. Trong đó cũng bao gồm rất nhiều config cho theme, package manager và support nhiều language syntax.

Điều đó có nghĩa là mình sẽ không cần phải manage `~/.vim` folder trên server nữa. Để cài đặt, config Vim trên server mình chỉ cần gõ:
```
git clone --recursive https://github.com/sheerun/vimrc.git ~/.vim
```

Hoặc bạn có thể tạo 1 repo [dotfiles](https://github.com/sheerun/dotfiles). Tất cả môi trường làm việc sẽ được set up trong 1 vài giây.

# Kết luận
Để có thể cải thiện hiệu năng làm việc với Vim, hãy tự tìm/ nhận ra vấn đề mà mình đang gặp phải trong quá trình phát triển và giải quyết chúng.

Cách giải quyết có thể là mapping vào trong file `.vimrc`, google để tìm solution hoặc hỏi trên các kênh IRC, hỏi người có kinh nghiệm, ...

Và nếu bạn có thêm những cách khác để cải thiện hiệu năng làm việc với Vim, hãy cùng chia sẻ cho mọi người nhé.
Cám ơn bạn đã đọc bài viết.

Bài viết được dịch từ trang: https://sheerun.net/2014/03/21/how-to-boost-your-vim-productivity/
