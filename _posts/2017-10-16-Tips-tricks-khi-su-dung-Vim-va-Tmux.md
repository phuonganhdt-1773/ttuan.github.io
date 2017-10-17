---
layout: post
comments: true
title: Tips, tricks khi làm việc với Vim và Tmux
tags:
- Vim
- Tmux
- Mouse Free
---


![Tmux and Vim](https://www.slothparadise.com/wp-content/uploads/2015/07/vimandtmux.jpg)

Đây là series mình tổng hợp lại các tips, tricks khi mình sử dụng Vim và Tmux trong quá trình code. Trong loạt bài viết này, mình sẽ không đánh giá tip nào hay/ không hay mà sẽ tập trung để viết nó theo **work flow**, nêu vấn đề gặp phải và cách giải quyết để mọi người tiện theo dõi ^^.

# Phần 1: Vim, Tmux and Multiple computers

## 1. Bật tất cả các tmux windows cùng 1 lần
* **Vấn đề**: Giả sử bạn code Rails, bạn sẽ phải mở 1 cửa sổ `Vim`, 1 cửa sổ chạy `rails c`, 1 cửa sổ chạy `rails s`, 1 cửa sổ chạy `guard`, ... Rất bất tiện mỗi lần bắt đầu code đúng không?
* **Giải pháp**: Sử dụng 1 gem tên là [tmuxinator](https://github.com/tmuxinator/tmuxinator)

Cách install gem thì bạn thảm khảo trong repo Github. Config thì bạn dùng file này:

```
# ~/.tmuxinator/project.yml

name: project
root: ~/Dropbox/Projects/<%= @args[0] %>

# Optional tmux socket
# socket_name: foo

# Runs before everything. Use it to start daemons etc.
# pre: sudo /etc/rc.d/mysqld start

# Runs in each window and pane before window/pane specific commands. Useful for setting up interpreter versions.
# pre_window: rbenv shell 2.0.0-p247

# Pass command line options to tmux. Useful for specifying a different tmux.conf.
# tmux_options: -f ~/.tmux.mac.conf

# Change the command to call tmux.  This can be used by derivatives/wrappers like byobu.
# tmux_command: byobu

# Specifies (by name or index) which window will be selected on project startup. If not set, the first window is used.
# startup_window: logs

# Controls whether the tmux session should be attached to automatically. Defaults to true.
# attach: false

# Runs after everything. Use it to attach to tmux with custom options etc.
# post: tmux -CC attach -t project
#
# To find Layout, just type tmux list-windows

windows:
  - editor:
      layout: b7a8,239x61,0,0[239x47,0,0,0,239x13,0,48,4]
      panes:
        - vim Gemfile
        - bundle exec rails s
  - railsc: bundle exec rails c
  - console: weather
```

```
# ~/.zshrc
alias tmx="tmuxinator start project $1"
```

Với config này, mỗi lần mở terminal, mình chỉ cần gõ `tmx project_name` (với `tmx` là alias của `tmuxinator`, project_name là tên project được đặt trong thư mục `~/Dropbox/Projects/project_name`), các cửa sổ `Vim`, `railsc` và `console` sẽ được bật lên cùng 1 lúc.

## 2. Di chuyển nhanh giữa Vim split và tmux panel
* **Vấn đề**: Khi 1 tmux window bạn split làm 2 phần, 1 để chạy Vim, 1 để chạy `rails s` chẳng hạn. Nhưng trong panel Vim, bạn lại split nó ra làm 2, 3 splits nhỏ hơn. Thế thì việc di chuyển giữa các splits và tmux panel khá là bất tiện. (Vì di chuyển ở Vim 1 kiểu, chuyển qua Tmux lại 1 kiểu khác)
* **Giải pháp**: [Mislav Marohnic](http://mislav.uniqpath.com/) có 1 giải pháp cho phép chúng ta sử dụng cùng 1 key strokes để switch giữa Vim và Tmux. Với giải pháp này, mình có thể dùng <C-h/j/k/l> để di chuyển giữa tất cả các splits.

From `.vimrc`, install [vim-tmux-navigator](https://github.com/christoomey/vim-tmux-navigator)

```
Bundle 'christoomey/vim-tmux-navigator'
```
In `~/.tmux.conf`, thêm đoạn config sau:

```
# smart pane switching with awareness of vim splits
bind -n C-h run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys C-h) || tmux select-pane -L"
bind -n C-j run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys C-j) || tmux select-pane -D"
bind -n C-k run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys C-k) || tmux select-pane -U"
bind -n C-l run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys C-l) || tmux select-pane -R"
bind -n C-\ run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys 'C-\\') || tmux select-pane -l"
```

## 3. Enable mouse on Tmux
WTH? Bạn muốn dùng Vim và Tmux vì muốn **Mouse Free**, sao giờ lại cần enable mouse trên Tmux???

Đây chỉ là vì cá nhân mình thấy khá tiện khi copy, resize, ... các tmux panel bằng chuột :v Nếu bạn k thích, hãy next qua trick này ;)

```
# ~/.tmux.conf

# Make mouse useful in copy mode
setw -g mode-mouse on

# Allow mouse to select which pane to use
set -g mouse-select-pane on

# Allow mouse dragging to resize panes
set -g mouse-resize-pane on

# Allow mouse to select windows
set -g mouse-select-window on
```

## 4. Đồng bộ clipboard giữa Vim, Tmux và System OS
* **Vấn đề**: Khi bạn copy 1 đoạn text nào đó ở Vim, Tmux hoặc System OS, sẽ khó để có thể paste nó sang 1 trong 2 cái còn lại. Tại sao k đồng bộ chúng lại, chỉ dùng 1 clipboard thôi?
* **Giải pháp**:

1. Install xclip:

```
sudo apt-get install xclip
```
2. Config in `~/.tmux.conf`

```
#\\ Copy and paste with OS clipboard
bind C-p run "tmux set-buffer \"$(xclip -o)\"; tmux paste-buffer"
bind C-y run "tmux save-buffer - | xclip -i"
```

3. In `~/.vimrc`

```
set clipboard^=unnamed,unnamedplus       " Share clipboard between Vim and OS, across platform
```

## 5. Đồng bộ các file config
* **Vấn đề**: Bạn sử dụng nhiều máy tính, mỗi lần thêm 1 config nào đó vào file `.vimrc` hoặc `.tmux.conf` thì lại phải thêm vào các máy khác (hoặc git add rồi pull về trên các máy còn lại) -> Mất công đúng không =]]
* **Giải pháp**: Hãy để Dropbox giúp bạn.

Tạo 1 folder `~/Dropbox/Projects/dotfiles`, đặt config của file `.vimrc` và `.tmux.conf` vào đó. Như vậy thì mỗi khi có thay đổi gì, nó sẽ được syn về các máy khác. Giờ vấn đề chỉ còn là cho tmux và vim nhận file này thôi.

Để làm đc điều này, bạn cần dùng soft link của linux:

```
ln -s ~/Dropbox/Projects/dotfiles/vim/.vimrc ~/.vimrc
```
Với thủ thuật này, config của bạn đã được đồng bộ giữa các máy rồi đó ;)


Phần này mình sẽ chỉ tạm giới thiệu qua 1 số tricks khi làm việc với cả Vim và Tmux. Phần sau mình sẽ tập trung nói về các tricks, plugin, custom functions cho Vim để bạn có thể code, làm việc với Vim 1 cách thuận tiện và dễ dàng hơn. ;)

Happy coding!!!
