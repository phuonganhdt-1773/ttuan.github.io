---
layout: post
title: Mouse Free&#58; Một vài kinh nghiệm khi làm việc với VIM
comment: true
tags:
- Mouse Free
- Vim
---

![VIM](https://cdn-images-1.medium.com/max/1200/1*OVSMUYjGFQg2_xlV5Q2S8w.png)

Đây là một bài viết về Vim mà mình đã muốn viết từ rất lâu rồi nhưng (vâng, lại là "nhưng") vẫn chưa có cơ hội =]] Bài viết này mình sẽ tổng hợp một vài kinh nghiệm, lời khuyên hoặc 1 vài tricks mà mình đang dùng khi code Vim.

Mình rất thích VIM. Lý do tại sao thì bạn có thể đọc ở bài viết [này](https://ttuan.github.io/2016/07/26/mouse-free-how-i-met-Vim/). Bạn cũng có thể tham khảo một vài tricks khi làm việc với VIM và Tmux ở bài viết [này](https://ttuan.github.io/2017/10/16/Tips-tricks-khi-su-dung-Vim-va-Tmux/)

## 0. Không bao giờ đặt 1 dòng trong file .vimrc mà không hiểu ý nghĩa
Đây là 1 rule được khá nhiều cao thủ đặt lên hàng đầu. Tại sao vậy?

Nếu bạn là người mới bắt đầu đến với Vim, hãy dùng mặc định. Việc dùng mọi config mặc định của Vim sẽ giúp bạn làm quen, hiểu về "ngôn ngữ" của Vim. Còn khi bạn nhìn ra vấn đề của mình - thứ mà config mặc định k giải quyết được 1 cách dễ dàng - thì đó là lúc bạn cần thêm 1 dòng nào đó vào file `~/.vimrc`. Dòng đó có thể đi copy từ file `~/.vimrc` của người khác, nhưng nó **giải quyết vấn đề của bạn**. Nếu bạn đặt vào đó mà k hiểu nó để làm gì, khi sang dùng 1 máy tính khác, bạn cũng sẽ k biết nên thêm gì vào đó để có thể thao tác giống với máy tính cũ của mình. :(

## 1. Quản lý file .vimrc
File `~/.vimrc` là file chứa mọi config cho VIM của bạn, nên hãy giữ nó 1 cách **clean** nhất có thể. Mình chia file ra làm 5 phần chính:
* General Config: Chứa các config cơ bản cho Vim như: set line number, encoding, shell, mouse, ...
* Apperance: Chứa các config liên quan tới giao diện: font, color, ..
* Plugins: Bao gồm các plugin mình hay dùng + config của mỗi plugin
* Mapping: Map các tổ hợp phím/ action thành các thao tác thuận tiện cho mỗi người.
* Behavior && extend: Chứa các config liên quan tới behavior như: swap file, autosave, backup, custom functions, ...

Với mỗi 1 dòng config, hãy đặt thêm comments cho nó. Vừa để bạn nhớ hơn khi đọc lại, vừa có thể giúp ích cho người khác khi họ tham khảo `~/.vimrc` của bạn.

## 2. Map Leader is your Space
Đây là điều mình thấy rất thuận tiện, mặc dù config cho nó khá là đơn giản. `Space` là phím to nhất trên bàn phím -> dễ bấm nhất, bạn có thể press phím đó bằng 2 ngón cái, ngón nào cũng được :v

```
let mapleader=" " " Set Space for Leader key
```

## 3. Map CapsLock to Esc
`Esc` là 1 phím được bấm khá nhiều lần để exit 1 mode nào đó, chuyển qua Normal mode.

Thông thường, bạn rất ít khi dùng phím `CapsLock`, mỗi lần muốn in hoa chữ cái gì thì bạn có thể dùng phím `Shift`. Thêm vào đó, phím `CapsLock` rất gần ngón út của bạn, có thể gõ dễ dàng hơn là phím `Esc`. Vì thế, bạn hãy thử map phím như vậy và xem thử xem có thuận tiện hơn không nhé.

Trên Mac bạn có thể vào mục Keyboard => Modifier Keys để map. Còn trên Ubuntu bạn có thể dùng Gnome Tweak Tool để thay đổi.

## 4. Edit config file quickly
Các file `~/.vimrc`, `~/.zshrc` đều là những file sẽ được chỉnh sửa khá nhiều, như thêm config, thêm biến môi trường, ... Vậy thì còn chờ gì nữa mà không map `:e ~/.vimrc` hay `:e ~/.zshrc` thành các tổ hợp phím thuận tiện với bạn? Đây là config của mình.

```
" ~/.vimrc
nmap <Leader>v :e ~/.vimrc<CR> 		           " Quickly edit .vimrc file
noremap <Leader><Leader>s :so ~/.vimrc<CR>     " Source .vimrc file
```

## 5. Vim Hardtime
Mình biết được từ khoá `Vim hardtime` sau khi đọc được 1 bài article [Habit breaking, habit making](http://vimcasts.org/blog/2013/02/habit-breaking-habit-making/). Đây thực sự là một bài viết hay, nó đã thay đổi cách dùng Vim của mình khá nhiều. Ý tưởng của bài viết là:

> Ban đầu, khi chưa dùng Vim, chúng ta quen với việc sử dụng 4 phím mũi tên, lên/xuống/sang trái/phải. Khi bắt đầu làm quen với VIM, chúng ta quen với việc dùng `h,j,k,l`. Tuy nhiên, tại sao chúng ta vẫn tiếp tục sử dụng `h,j,k,l` trong khi VIM cung cấp rất nhiều "dozens of motions" khiến bạn có thể moving nhanh hơn rất rất nhiều. Hãy tham khảo [Moving Around](http://vim.wikia.com/wiki/Moving_around). Hãy dùng `Wordwise` thay cho h, l; Dùng `f, F, t, T, ;, ,` để di chuyển nhanh hơn; Dùng `Ctrl-d, Ctrl-u` để di chuyển page up, page down, `g, G` di chuyển đến đầu, cuối trang; Dùng `10j, 10k` để di chuyển 10 dòng lên/ xuống,....

Để làm được việc đó. Bạn hãy cài plugin [Vim hardtime](https://github.com/takac/vim-hardtime)

```
" Vim HardTime
Plug 'takac/vim-hardtime'
```

Hãy tập làm quen với việc di chuyển như vậy. Mình chắc chắn với bạn rằng, chỉ sau khoảng 1 tuần, thói quen Moving around của bạn sẽ hoàn toàn thay đổi.

Ngoài ra, mình có suggest một cách khác, có thể giúp việc di chuyển khi thao tác với file đó là 1 plugin tên là [Vim easy motion](https://github.com/easymotion/vim-easymotion). Sau khi đã cài xong plugin này, bạn có thể config nó sao cho thuận tiện.

```
"\\ Easy Motion
let g:EasyMotion_do_mapping = 0
map <Leader> <Plug>(easymotion-prefix)
map <Leader>L <Plug>(easymotion-bd-jk)
nmap <Leader>L <Plug>(easymotion-overwin-line)
map  <Leader>w <Plug>(easymotion-bd-w)
nmap <Leader>W <Plug>(easymotion-overwin-w)
map <Leader>l <Plug>(easymotion-lineforward)
map <Leader>h <Plug>(easymotion-linebackward)
```

## 6. Search is your friend
Trong Vim, việc bạn search 1 từ khoá trong 1 file rất dễ dàng. Hãy tận dụng nó.

Cú pháp chỉ đơn giản là: `/keyword`. Ví dụ bạn muốn nhảy tới từ `friend` trong file này, chỉ cần gõ `/friend` thì có thể nhảy ngay tới rồi. Để tới từ match tiếp theo hay từ match trước đó, hãy dùng `n, N`.

Ngoài ra, trong Vim, bạn có thể sử dụng `*, #` để tìm tất cả các từ matching trong file. cũng rất thuận tiện ;)

## 7. Mạnh dạn add thêm các snippets
Các editor thường hỗ trợ bộ snippets rất mạnh, cho phép bạn có thể thêm 1 `alias` cho 1 đoạn code, từ khoá, ... dài. Điều này cho phép bạn thu gọn các đoạn code lặp đi lặp lại thành 1 từ rất ngắn.

Nếu bạn có cài plugin `vim-snippet`, bạn có thể tìm đến thư mục lưu code của `vim-snippet` trong `~/.vim` và thêm vào đó các config của mình. Ví dụ như dưới đây mình có thêm snippet cho Ruby:

```
" ~/.vim/plugged/vim-snippets/snippets/ruby.snippets
snippet bd
	binding.pry
```

Như vậy là mình có thể gõ `bd + Tab` thay cho `binding.pry` khi đang ở trong file `.rb` rồi.

## 8. Vim operator và text objects
Trong bài viết [How I met Vim](https://ttuan.github.io/2016/07/26/mouse-free-how-i-met-Vim/), mình có nhấn mạnh 1 điều là: Cấu trúc 1 câu lệnh của Vim luôn là:

> [number][operator][motion/ text object]

Hôm nay, mình sẽ giới thiệu với các bạn 1 số operator và text objects mở rộng mà cộng đồng Vim đã viết thêm để có thể giúp ích cho những trường hợp hay sử dụng.

### 8.1 Text objects
Bạn có thể tham khảo một số textobject ở repo [Vim-textobj-user](https://github.com/kana/vim-textobj-user/wiki). Dưới đây mình chỉ giới thiệu 1 số textobject mình hay dùng:

* [Indent Object](https://github.com/michaeljsmith/vim-indent-object) cung cấp text object dựa trên indentation level. Text object này rất hữu ích khi bạn edit các file code Python, Ruby, CoffeeScript, ... Bạn có thể dùng `ai` hoặc `ii` để thao tác với các text có cùng indent. Ví dụ: `dai` sẽ xoá các đoạn text có cùng indent với dòng mà cusor đang trỏ.
* [Ruby Block](https://github.com/nelstrom/vim-textobj-rubyblock) cung cấp text object dựa trên Ruby block, bao gồm tất cả các expression mà close với keyword `end`. Bạn có thể xoá các dòng text trong 1 block Ruby bằng cách gõ: `dir` hoặc `dar` để xoá cả block Ruby đó đi.
* [Line Object](https://github.com/kana/vim-textobj-line): Mỗi dòng là 1 object. Bạn có thể xoá 1 dòng bằng cách gõ `dd` trong Vim, nhưng mình thích theo đúng cú pháp của Vim là: `dal` hơn =]]
* [erb Object](https://github.com/whatyouhide/vim-textobj-erb): thao tác với `<%= %>` hoặc `<% %>` trong file erb. Bạn có thể dùng `ciE` hoặc `daE` để thao tác với các thẻ tag trên.
* [Vim textobj Delimited](https://github.com/machakann/vim-textobj-delimited): thao tác với các text cách nhau bởi dấu `_` (ví dụ như 'foo_bar_bar', khi con trỏ đang ở chữ `foo`, bạn gõ `did` thì đoạn text sẽ là: '_bar_bar')

Còn rất nhiều text object khác mà mình đã để link ở repo trên kia, bạn hãy vào, tìm textobject mà mình hay dùng, sau đó cài và trải nghiệm nhé ;)

### 8.2 Operator
Trong cú pháp câu của Vim, có 1 phần rất quan trọng đó là `Operator`. Nó như là 1 động từ trong 1 câu vậy. Dưới đây mình sẽ giới thiệu 1 vài `operator` mà mình hay dùng (ngoài `d, c, y,..` mặc định của Vim)

* [Replace with Register](https://github.com/vim-scripts/ReplaceWithRegister): Bạn dùng `gr` để replace text object chỉ định bằng đoạn text được copied trong clipboard. Ví dụ bạn copy 1 dòng, sau đó gõ `gri"` thì tất cả đoạn text trong dấu `" "` bằng dòng đang trong clipboard.
* [Vim Surround](https://github.com/tpope/vim-surround): Bạn dùng `cs, ds,..` để thay đổi các thẻ, dấu xung quanh 1 text object. Ví dụ, bạn có đoạn text `"test word"`, bạn có thể gõ `cs"'` để biến dấu "" xung quanh từ `test word` thành dấu ''. Để biết chi tiết hơn, bạn có thể tham khảo trong link repo này nhé.
* [Vim Commentary](https://github.com/tpope/vim-commentary): Bạn dùng `gc` để comment 1 text object nào đó. Riêng mình mình map `cm` = `gc`. Như vậy, để comment 1 dòng mình gõ `cmal` - Comment a line, `cmap` - Comment a paragraph, ...

Còn rất nhiều operator khác nữa, bạn có thể google để tìm ra operator cần thiết cho mình. Hoặc bạn cũng có thể tham khảo để tìm cách để viết 1 operator mới =]]

## 9. Plugins hữu ích :D
Khi bạn bắt đầu sử dụng Vim, có lẽ bạn sẽ nhận được rất nhiều lời khuyên là k nên sử dụng Plugin, hãy dùng mặc định. Nhưng cá nhân mình thấy nếu plugins tiện lợi cho mình -> hãy dùng =]]

Dưới đây mình list ra 1 số plugins tiện dùng mà mình hay dùng.

### 9.1 Common plugins
```
Plug 'Valloric/YouCompleteMe'    # Plugin giúp autocomplete/ show gợi ý code
Plug 'rking/ag.vim'					# Plugin giúp search multiple file nhanh hơn
Plug 'tpope/vim-repeat'				# Hỗ trợ dùng "." cho 1 số custom operator
Plug 'kien/ctrlp.vim'				# Plugin của 1 anh người Việt, bring fuzzy search file
Plug 'mattn/emmet-vim'				# Emmet for Vim
Plug 'wakatime/vim-wakatime'		# Wakatime, thống kê thời gian code của bạn trên Vim
Plug 'SirVer/ultisnips'
Plug 'honza/vim-snippets'			# Hỗ trợ viết snippets
Plug 'scrooloose/nerdtree'			# File bar
```

### 9.2
Plugins for Ruby on Rails dev
```
Plug 'tpope/vim-rails'				# Awesome plugin for Rails dev. Đây có lẽ là plugin mình dùng nhiều nhất =]]
Plug 'tpope/vim-fugitive'			# Git for Vim. Mình chủ yếu dùng mapping <Leader>bl :Gblame<CR>
Plug 'airblade/vim-gitgutter'		# Để biết bạn thêm/ xoá, sửa dòng nào trong git project
Plug 'vim-ruby/vim-ruby'
Plug 'tpope/vim-endwise'			# Auto add `end` key for `if, do, def, ...`
Plug 'tpope/vim-bundler'			# Bundler quickly in Vim
Plug 'thoughtbot/vim-rspec'		# Run spec quickly. Minh hay dùng `nnoremap <Leader>rs :call RunCurrentSpecFile()<CR>` mapping để chạy test current file
```

Đây là 1 số plugins mình hay dùng :v Và quả thực nó giúp ích mình rất nhiều trong quá trình code hàng ngày (Đặc biệt là plugin `vim-rails` với `gf`, `:Eview/model/controller/migration`,...). Hy vọng có thể giúp ích được cho các Rails developer ;)

## 10. Một vài câu lệnh/ custom functions hay dùng
### 10.1 Select the pasted text
Mình thường dùng thao tác này khi copy code ở 1 file A rồi dán vào file B. Nếu như bị sai indent thì mình cần phải chọn đoạn text vừa paste, sau đó dùng `>` để chỉnh lại indent. Đây là config của mình:

```
"\\ Quickly select text you just pasted
noremap gV `[v`]
```

### 10.2 Swap last 2 files
Bạn vừa chỉnh file A, sau đó mở file B lên. Sau khi edit xong file B, bạn muốn quay trở lại file A thì dùng cách nào?

```
"\\ Switch between files
nnoremap <tab> :bp<CR> " Previous buffer file
nnoremap <S-tab> :bn<CR> " Next buffer file
nnoremap <Leader><Leader>c <c-^> " The last two files
```
Swap 2 files bằng cách bấm `<Space><Space>c` với Space là Leader key của mình.

### 10.3 Open file in same directory
Bạn đang edit 1 file `a.txt`. Giờ bạn muốn thêm/ edit 1 file `b.txt` cùng thư mục với file `a.txt`, bạn có thể dùng config dưới đây để có thể thực hiện nhanh:

```
"\\ Open file in same directory
nnoremap ,e :e <C-R>=expand('%:p:h') . '/'<CR>
nnoremap ,t :tabe <C-R>=expand("%:p:h") . "/" <CR>
nnoremap ,s :split <C-R>=expand("%:p:h") . "/" <CR>
```

### 10.4 Force save file when you forgot run "sudo vim file_name"
Đôi khi bạn edit file trong các thư mục nằm ngoài `~`, bạn cần phải chạy `sudo` nhưng lại quên mất. (do thói quen, mình hay gõ: `vim nginx.conf` luôn mà quên gọi `sudo vim`). Đó là khi config dưới đây tỏ ra rất hữu dụng:

```
"\\ Force save file when I forgot run 'sudo vim file'
"\\ With Great Power Comes Great Responsibility
cmap w!! %!sudo tee > /dev/null %
```
Bạn gõ: `:w!!` để force save file nhé.

### 10.5 Rename current file
Không chỉ edit text trong file, bạn cũng có thể edit được tên file đang thao tác bằng cách sử dụng custom function sau:

```
" Rename current file
function! RenameFile()
  let old_name = expand('%')
  let new_name = input('New file name: ', expand('%'), 'file')
  if new_name != '' && new_name != old_name
    exec ':saveas ' . new_name
    exec ':silent !rm ' . old_name
    redraw!
  endif
endfunction
map <Leader>rnf :call RenameFile()<cr>
```

Hầy. Bài viết khá dài. Cám ơn các bạn đã đọc tới tận đây. Mình hy vọng những kinh nghiệm khi làm việc với Vim của mình sẽ giúp ích được cho các bạn mới dùng Vim hoặc dùng Vim ở mức cơ bản :D Nếu có thắc mắc gì, bạn có thể để lại comment ở dưới bài viết nhé ;)

Happy coding!!!!!!!!
