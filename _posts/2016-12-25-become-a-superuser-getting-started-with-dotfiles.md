---
layout: post
comments: true
title: Become a SuperUser&#58; Getting started with Dotfiles
tags:
- Linux
---

![dotfile.png](https://viblo.asia/uploads/images/15710dd85247a62e3aa5a11089b38835729597e9/00ea42cf7ecec30d39fa78d18f9d77a791ee3f00.png)

## Introduction

Đã bao giờ vào 1 ngày đẹp trời, em máy tính của bạn bỗng tự nhiên lăn đùng ra chết. Và sau đó bạn phải cài lại 1 hệ điều hành mới, toàn bộ phần mềm, setting cho các chương trình cũ của bạn đều bị mất? Bạn mất hàng giờ (có khi là cả ngày) chỉ để ngồi cài lại những chương trình đó, sau đó lần mò settings lại? Nếu bạn đã gặp những trường hợp như thế thì DOTFILES chính là giải pháp cho bạn.

## Dotfiles

### 1. Khái niệm
Bạn có thể hiểu Dotfiles đơn giản là tập hợp tất cả các files có dấu chấm ở đầu. =]]] Tuy nhiên, các files đó chứa các config của 1 chương trình hay 1 hệ thống nào đó.

Các files này xuất hiện rất nhiều trong công việc lập trình hàng ngày của bạn. Set biến môi trường thì cài đặt trong `~/.bashrc`, set ignore git thì thêm vào file `gitignore`, ... Cái tên "Dotfiles" cũng bắt đầu từ việc các file này luôn có dấu "." ở đầu. Nó mặc định sẽ bị ẩn đi khi bạn mở thư mục.

### Tại sao nên sử dụng Dotfiles?

Dù không hề biết đến Dotfiles, bạn vẫn có thể sử dụng máy tính bình thường, thế thì cần gì tới dotfiles :v. Hơn nữa, cho dù có phải cài lại máy tính, bạn vẫn có dư thời gian để có thể ngồi cài lại máy tính :v thế thì ta có nhất thiết cần biết tới Dotfiles?

Câu trả lời là không =]]] Tuy nhiên, Dotfile sẽ mang lại cho bạn rất nhiều điều thú vị:

* Automate All The Things!
* You’re The King Of Your Castle!
* Always Learn something new!

Trước đây, có lần mình đọc được 1 bài viết về "How to be a lazy developer?", trong đó có viết về việc 1 anh chàng lập trình viên cố gắng trở nên luời biếng bằng cách viết mọi script cho các công việc gì mà anh đó CÓ THỂ làm đi làm lại và mất nhiều hơn 5 phút. Mình thấy ý tưởng này rất hay và thú vị.

Trong lập trình có 1 nguyên tắc DRY - Don't repeat yourself - Không bao giờ lặp đi lặp lại 1 cái gì đó. Vì thế hãy làm mọi thứ trở thành tự động. Đừng bao giờ gõ đi gõ lại những dòng config, những lệnh cài đặt phần mềm, ... trong khi chúng ta có thể làm việc đó trở nên đơn giản hơn rất nhiều lần. Dotfiles giúp bạn backup, restore hay đồng bộ những đoạn setting giữa các thiết bị 1 cách dễ dàng.

Thêm vào đó, Dotfiles chính là nơi để bạn thỏa sức sáng tạo =]] Bạn có thể đặt vào đó những config mang đậm phong cách cho mình :v Việc tự mình cài đặt, map phím, viết thêm extend functions sẽ tạo ra những chương trình phù hợp vs thói quen, mục đích sử dụng chương trình của bạn. Điều này sẽ mang tới cho bạn cảm giác có thể kiểm soát mọi thứ theo ý của mình, tự mình tạo nên 1 chiếc máy cho chính mình. :D

Trong quá trình cài đặt đó, mình tin chắc rằng bạn sẽ học được rất nhiều thứ, bạn sẽ hiểu hơn về cách 1 ứng dụng sẽ chạy như thế nào, học hỏi được những config thú vị từ repo dotfiles của những người khác, cách nhận ra các vấn đề và hướng giải quyết, ... Việc tự mình giải quyết các vấn đề cho mình là một cách rất tốt để học hỏi điều mới

### Làm thế nào để custom Dotfiles?

Dưới đây là cấu trúc thư mục dotfiles của mình:

```
Dotfile
├── ctags
│   └── ctags
├── fish
│   └── config.fish
├── git
│   ├── gitconfig
│   └── gitignore
├── i3
│   ├── config
│   └── i3status.conf
├── install.sh
├── README.md
├── tmux
│   └── tmux.conf
├── tmuxinator
│   └── project.yml
├── vim
│   ├── ackrc
│   └── vimrc
├── vimperator
│   └── vimperatorrc
└── zsh
    ├── alias.zsh
    ├── fzf.zsh
    └── zshrc
```

Có khá nhiều các file config ở đây, phần lớn các file này đều được lưu ở thư mục ~/ trong ubuntu. Do có khá nhiều dùng Zsh nên mình sẽ tập trung nói về cách xây dựng ~/.zshrc

1. Đọc document của phần mềm hoặc hệ thống

Khi mới bắt đầu với zsh, mình có đọc qua một vài tài liệu như ở [đây](https://en.wikipedia.org/wiki/Z_shell) hoặc ở [đây](http://www.slideshare.net/jaguardesignstudio/why-zsh-is-cooler-than-your-shell-16194692) . Các tài liệu này viết khá hay và đầy đủ, chỉ cho mình cách config một số trick cơ bản vs zsh, những ưu điểm của nó so với các shell khác như bash, sh,..

Một trong những thế mạnh khi xây dựng 1 dotfiles hoàn hảo đó là sử dụng plugin có sẵn. Có rất nhiều plugins hay và thú vị đã được cộng đồng mạng xây dựng. Mình có google thử 1 vài tool để quản lý các plugins cho zsh thì được recommend dùng Oh-my-zsh. Hiện tại mình đang dùng nó để quản lý các plugins.

Đây là nội dung files .zshrc hiện tại của mình:

```
# Path to your oh-my-zsh installation.
export ZSH=/home/$USER/.oh-my-zsh

# Dotfiles folder
export DOTFILE=~/.dotfiles

# Ack like vim
bindkey -v

ZSH_THEME="agnoster"

export UPDATE_ZSH_DAYS=7

# Plugin
plugins=(rails zsh-autosuggestions tmuxinator tmux vi-mode)

# User configuration
export PATH="$PATH:$HOME/.rvm/bin"

source $ZSH/oh-my-zsh.sh

export TERM="xterm-256color"
export EDITOR="vim"

# Alias
source $DOTFILE/zsh/alias.zsh

# FZF functions
source $DOTFILE/zsh/fzf.zsh

[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh

bindkey '^I' fzf-file-widget
bindkey '^O' fzf-history-widget
bindkey '^ ' autosuggest-execute
```

2. Học lỏm từ dotfiles của người khác.

Mỗi dotfiles của một người nào đó có rất nhiều config hay và hữu dụng, đặc trưng cho công việc, thói quen sử dụng của từng người. Mình rất hay vào các repo dotfiles của mọi người trên github để đọc, tìm ra một vài config thú vị, phù hợp cho coder ruby on rails. :v . Trong một lần lượn quanh cách repo đó, mình có đọc được 1 số tricks rất thú vị về [Fzf](https://github.com/junegunn/fzf).

Như mọi người đã thấy ở files ~/.zshrc bên trên, có 1 đoạn mình load file: `fzf.zsh`. Đây là nội dung của file đó:

```
FZF_TMUX_HEIGHT=20

# Open file with Vim
v() {
  local file
  file=$(fzf --query="$1") && vim "$file"
}

# cd to folder
fd() {
  local dir
  dir=$(find ${1:-*} -path '*/\.*' -prune \
                  -o -type d -print 2> /dev/null | fzf +m) &&
  cd "$dir"
}

# show all hidden folder to 'cd'
fda() {
  local dir
  dir=$(find ${1:-.} -type d 2> /dev/null | fzf +m) && cd "$dir"
}

# Search in history
fh() {
  eval $(history | fzf +s | sed 's/ *[0-9]* *//')
}

# Kill a process
fk() {
  ps -ef | sed 1d | fzf -m | awk '{print $2}' | xargs kill -${1:-9}
}

# Checkout a branch
fbr() {
  local branches branch
  branches=$(git branch) &&
  branch=$(echo "$branches" | fzf +s +m) &&
  git checkout $(echo "$branch" | sed "s/.* //")
}

# Checkout a commit
fco() {
  local commits commit
  commits=$(git log --pretty=oneline --abbrev-commit --reverse) &&
  commit=$(echo "$commits" | fzf +s +m -e) &&
  git checkout $(echo "$commit" | sed "s/ .*//")
}

# Search tags
ft() {
  local line
  [ -e tags ] &&
    line=$(grep -v "^!" tags | cut -f1-3 | cut -c1-80 | fzf --nth=1) &&
    $EDITOR $(cut -f2 <<< "$line")
}

# fq1 [QUERY]
# - Immediately select the file when there's only one match.
#   If not, start the fuzzy finder as usual.
fq1() {
  local lines
  lines=$(fzf --filter="$1" --no-sort)
  if [ -z "$lines" ]; then
    return 1
  elif [ $(wc -l <<< "$lines") -eq 1 ]; then
    echo "$lines"
  else
    echo "$lines" | fzf --query="$1"
  fi
}

# fe [QUERY]
# - Open the selected file with the default editor
#   (Bypass fuzzy finder when there's only one match)
fe() {
  local file
  file=$(fq1 "$1") && ${EDITOR:-vim} "$file"
}

```

Trong file này có rất nhiều hàm thú vị, các bạn có thể dễ dàng thực hiện 1 lệnh mà không cần thực hiện các thao tác dài dòng. Tất cả các hàm trên sẽ được gọi dưới dạng 1 alias của zsh. Mình rất hay sử dụng các lệnh này, đặc biệt là lệnh `fk` để kill process không cần dùng đến.

3. Viết thêm những config khác.

Như mình nói ở trên, mỗi 1 repo dotfiles đều mang đặc trưng của người viết ra nó, nên bạn hãy tự tạo ra cho mình những file config phù hợp với bản thân.
Những config đó có thể là một số alias bạn hay sử dụng, hoặc cũng có thể là 1 số đoạn mã hay ho mà bạn nghĩ ra hoặc "chôm" được từ dotfiles của ai đó =)). Thường thì mình để trong file `alias.zsh`. Ví dụ như đoạn mã check thời tiết ngày hôm nay ở Hà Nội:

```
weather() {
  local CITY=${1:-Hanoi}
  curl -4 "wttr.in/$CITY"
}
```
hoặc translate nhanh 1 từ bằng google nếu bạn cài [translate-shell](https://github.com/soimort/translate-shell):

```
# Translate with google
alias t="trans -b :vi"
```
hay tra cứu nhanh trên stackoverfolow từ terminal với [howdoi](https://github.com/gleitz/howdoi).

```
# How do I ....
alias how="howdoi"
```

Ngoài ra, mình có viết thêm 1 vài alias để tiện cho việc dùng Git và chạy rails trong file alias.zsh

```
OTFILE=~/Dropbox/Projects/dotfile

# Alias for Git
alias gd="git diff @~..@"
alias grs="git reset HEAD~1"
alias gst="git status -s"
alias gsta="git add -A; git stash"


run_migrate() {
  if rake db:migrate:status | grep down
  then
    rake db:migrate
    return 0;
  fi
}

d.() {
  curr_branch=$(git symbolic-ref --short -q HEAD);
  git checkout develop;
  git br -D $curr_branch;
}

gpf() {
  git pull framgia develop;
  run_migrate;
}

gf() {
  git add -A;
  git commit -m $1;
  branch_name=$(git symbolic-ref --short -q HEAD);
  if gd | grep "binding.pry"
  then
    echo "Binding pry detected!"
    return 0
  fi
  git push origin $branch_name;
  repo_url=$(git config --get remote.origin.url)
  firefox $repo_url;
}

gff() {
  git add -A;
  git commit --amend;
  branch_name=$(git symbolic-ref --short -q HEAD);
  if gd | grep "binding.pry"
  then
    echo "Binding pry detected!"
    return 0
  fi
  git push origin $branch_name -f;
}

grb() {
  curr_branch=$(git symbolic-ref --short -q HEAD);
  git checkout $1;
  git pull framgia $1;
  git checkout $curr_branch;
  git rebase $1;
}

grc() {
  git add .;
  git rebase --continue;
  branch_name=$(git symbolic-ref --short -q HEAD);
  git push origin $branch_name -f;
}

gcc() {
  git checkout master;
  git checkout -b $1;
}

#\\ System
alias sdown="sudo shutdown -h now"
alias gno="gnome-open"
alias bi="bundle install"
alias tmx="tmuxinator start project $1"
alias serve="python -m SimpleHTTPServer 8000"

alias pls="sudo"
alias rr="rm -rf"
alias q="exit"
alias c="clear"
alias s="source ~/.zshrc"

alias h="history | grep"
alias ps="ps auxf | grep"

alias kill="sudo killall -9"

# ls
alias ll="ls -lah"
alias la="ls -A"
alias l="ls"

# Connect wifi
alias onwifi="nmcli nm wifi on"
alias offwifi="nmcli nm wifi off"
alias showwifi="nmcli device wifi list"
alias rescanwifi="nmcli device wifi rescan"
cnwifi() {
  nmcli device wifi connect $1 password $2
}

# File system tree
alias .='pwd'
alias ..='cd ..'
alias ...='cd ../..'

alias dot="cd $DOTFILE"

mkcd() {
  mkdir "$1"
  cd "$1"
}
search() { find . -iname "*$@*" | less; }

#\\ Application
alias update="sudo apt-get update"
alias install="sudo apt-get install"
alias add="sudo add-apt-repository"
```

# Kết luận
Trên đây, mình đã giới thiệu với các bạn một vài điều cơ bản về Dotfiles. Hi vọng rằng, qua ví dụ về việc xây dựng `~/.zshrc`, bạn có thể tự tạo 1 repo Dotfiles của riêng mình, phục vụ tốt nhất cho công việc và thói quen sử dụng máy tính của bạn. Và đừng quên, share `dotfiles` của bạn lên github để mọi người có thể học hỏi cùng nhé.

Bạn cũng có thể xem qua dotfiles của mình tại [đây](https://github.com/ttuan/dotfile).

Cảm ơn các bạn đã dành thời gian đọc bài viết.

Tài liệu tham khảo: https://medium.com/@webprolific/getting-started-with-dotfiles-43c3602fd789#.hjk1r1vls
