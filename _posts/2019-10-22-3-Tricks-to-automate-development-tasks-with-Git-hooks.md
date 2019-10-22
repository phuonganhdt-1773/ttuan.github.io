---
layout: post
title: 3 Tricks to automate development tasks with Git Hooks
tags:
- Dev
- Git
---

![](https://miro.medium.com/max/1090/1*AWV-E_cbn7HOnrqNC2XjXg.jpeg)

## 1. Mở đầu
Nếu bạn đã từng làm việc với các project open source, hoặc làm việc trong 1 team, chắc hẳn bạn đã từng sử dụng qua một số tool để quản lý versions. Version Control System (VCS) đã trở thành một công cụ không thể thiếu cho các project, trong đó, phổ biến nhất là [Git](https://git-scm.com/). Tuy nhiên, khi số lượng member trong team lớn dần lên, sẽ rất khó để kiểm soát được code styles và áp dụng các rules của dự án cho tất cả các contributors. Để giải quyết vấn đề này, Git đã tạo ra một feature rất hay, đó là Git Hooks.

Trong bài viết này chúng ta sẽ cùng nhau đi tìm hiểu về Git hooks và cách áp dụng nó để automate một số công việc trong quá trình phát triển dự án nhé ;)

## 2. Git Hooks
Git Hooks là một tính năng cho phép tự động hoá 1 số hành động, sẽ được thực hiện đi kèm với các Git commands.

Hooks là các chương trình chạy trước hoặc sau 1 Git event. Ví dụ: trước khi hoàn thành 1 git commit bằng lệnh `git commit`, hay sau khi thay đổi branch bằng lệnh `git checkout tên_branch` hay trước khi push code bằng lệnh `git push`.

Hooks được lưu trữ trong thư mục `.git/hooks` của mỗi project. Chúng bắt buộc phải là các scripts hoặc chương trình mà có thể excecute được. (Trên hệ điều hành Unix-like, chúng phải được cấp quyền execute bằng lệnh `chmod +x tên_file`)

Để add 1 hook vào project, chúng ta chỉ cần copy hoặc tạo sym link tới thư mục `.git/hooks`, với tên chương trình là tên các "event" của git, ví dụ: `pre_commit` chẳng hạn. Full command bạn có thể đọc thêm ở [git hooks document](https://git-scm.com/book/uz/v2/Customizing-Git-Git-Hooks).

## 3. Tricks to automate development tasks with Git hooks

### 1. Check quality of the code before committing
Ý tưởng là ta sẽ tạo 1 hooks, được chạy trước để xác nhận lại xem những thay đổi mà chúng ta vừa tạo ra có hữu dụng hay không. Ví dụ ta sẽ chạy lint lại project, trong Rails application ta sẽ chạy 1 số tool như `rspec`, `rubocop`, ... để check lại.


Nếu hook fails hoặc trả về error code, những thay đổi ta vừa tạo ra sẽ không được commit vào.

Ví dụ dưới đây sử dụng package `jslint` của Node.js để check quality cho JavaScript application code trước khi commit:

```sh
#!/bin/bash
# pre-commit.sh

# Save unconfirmed changes
STASH_NAME="pre-commit-$(date +%s)"
git stash save -q --keep-index $STASH_NAME

# Checks and tests
jslint my_application.js || exit 1

# Recover saved changes
STASHES=$(git stash list)
if [[ $STASHES == "$STASH_NAME" ]]; then
  git stash pop -q
fi
```

Dựa vào đoạn code trên, bạn cũng có thể tạo ra file check quality cho riêng mình, dùng để chạy trước khi tạo 1 commit nào đó.

### 2. Generate docummentation as changes are uploaded

Nếu trong dự án của bạn có sử dụng 1 số tool để generate document tự động từ code, chúng ta có thể execute chúng mỗi khi push 1 commit lên Github dựa vào `pre-push` hook.

Đơn giản, chúng ta sẽ chạy các command cần thiết để tổng hợp documentation vào 1 thư mục nào đó, ví dụ `./docs/` và add các file này vào commit mới.

Bạn có thể tham khảo ví dụ dưới đây:

```sh
#!/bin/bash
# pre-push.sh

# Generate the documentation
doxygen Doxyfile

# Another example, with Python:
pydoc3 -w docs/

# Another example, with R:
Rscript -e 'pkgdown::build_site()'

# Add and confirm the changes related to the documentation
git add docs/
git commit -m "Update documentation ($(date +%F@%R))"
```

Công dụng lớn nhất đó là, nếu bạn sử dụng Github Pages hoặc 1 service nào đó để viết documentation, bạn sẽ dễ dàng cập nhật doc mới nhất cho những đoạn code mình viết ra mà không phải mất nhiều công sức làm bằng tay :v

### 3. Check dependencies khi thay đổi branch
Một ứng dụng rất thú vị của hooks đó là update các dependencies mỗi khi chúng ta thay đổi branch trong 1 project.

Nếu như bạn sử dụng package managers và dependencies tool (Ví dụ như Pip cho Python, npm cho Node.js, Nuget cho .NET, Bundler trong Ruby, Cargo trong Rust, ...), thì sử dụng Git hook này sẽ rất tiện lợi.

Ý tưởng là mỗi khi chúng ta checkout 1 branch, git hook này sẽ check xem giữa branch cũ và branch ta vừa checkout có khác nhau về dependencies hay không. (dự vào hash code của file quản lý dependencies - Gèmile trong Ruby hay package.json trong Node.js), nếu có thay đổi, nó sẽ executes lệnh để cài đặt dependencies mới.

```
#!/bin/bash
# post-checkout.sh

if [ $1 = 0000000000000000000000000000000000000000 ]; then
  # If we are in a recently cloned project, compare with the empty directory
  old=$(git hash-object -t tree /dev/null)
else
  # The first argument is the hash of the previous HEAD
  old=$1
fi
if [ -f Gemfile ] &&
  git diff --name-only $old $2 | egrep -q '^Gemfile|\.gemspec$'
then
  # Empty $ GIT DIR prevents problems if bundle calls git
  (unset GIT_DIR; exec bundle)
  # Checkout is completed even if the bundler fails
  true
fi
```

Ví dụ trên đây là dùng cho Gemfile trong Ruby on Rails project. Bạn cũng có thể thay đổi Gèmile thành `package.json` hoặc `requirements.txt`, thay lệnh `bundle` thành lệnh tương ứng như `npm install` hoặc `pip install -r requirements.text` tuỳ vào dự án của bạn ;)

## Kết luận
Bài viết là những kiến thức và ý tưởng rất cơ bản về Git Hooks. Bạn có thể Google thêm về những tricks, thủ thuật của các developers khác, sử dụng Githooks để tối ưu hoá Workflow của họ.

Hoặc chỉ cần bạn có ý tưởng, sử dụng Git Hooks sẽ làm việc coding trở nên nhẹ nhàng hơn rất nhiều ;)

Cám ơn các bạn đã đọc bài viết.

Nguồn: [https://dev.to/shameemreza/3-tricks-to-automate-development-tasks-with-git-hooks-2dah](https://dev.to/shameemreza/3-tricks-to-automate-development-tasks-with-git-hooks-2dah)



















