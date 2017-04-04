---
layout: post
comments: true
title: Easy Life&#58; Duyệt web cùng Vimperator
tags:
- Easy Life
- Vim
---

![](https://viblo.asia/uploads/7a781581-4c47-4257-9e56-170ee2265a5f.jpeg)

## Mở đầu
Trong kỷ nguyên của công nghệ số như hiện nay, mọi thứ đều được người ta đưa lên nền web. Mỗi ngày, chúng ta dành nhiều giờ liền cho việc tìm kiếm thông tin, lướt qua các trang báo, chat, mạng xã hội,... Nếu bạn muốn có những trải nghiệm mới mẻ hơn trong việc duyệt web thì Vimperator chính là một lựa chọn tuyệt vời cho bạn.

## Vimperator là gì?
[Vimperator](http://vimperator.org/vimperator) là một extension trên Firefox, được lấy cảm hứng từ editor nổi tiếng [Vim](http://www.vim.org/) với mục đích ban đầu là làm cho việc duyệt web được trở lên nhanh và dễ dàng hơn bằng cách map các action của trình duyệt thành các command, mỗi command lại được map thành các tổ hợp phím tắt. Khi đã làm quen với các tổ hợp phím tắt này rồi, việc duyệt web của bạn sẽ trở nên nhẹ nhàng và cực kì hiệu quả.

Vimperator không chỉ đơn giản mang đến một command interface cho Firefox, nó còn mang tới cho bạn một môi trường phát triển tuyệt vời. Nếu bạn là một developer và muốn mở rộng khả năng ứng dụng của Vimperator, bạn có thể viết thêm các đoạn script bằng Javascript và sử dụng chúng như những command mới cho trình duyệt.

Bạn có thể cài đặt Vimperator cho Firefox ở [đây](https://addons.mozilla.org/en-US/firefox/addon/vimperator/).
Bạn cũng có thể thử qua [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb) trên Chrome hoặc [Vimari](https://github.com/guyht/vimari) trên Safari.

## Tại sao nên dùng Vimperator
### 1. Duyệt web thuận tiện hơn
Khi bạn duyệt web, bạn sẽ phải di chuyển qua chuyển lại giữa bàn phím và chuột. Ví dụ như click vào 1 đường link, click vào 1 ô text box, click vào 1 button nào đó,... Việc này diễn ra khá thường xuyên và có thể ngún khá nhiều thời gian của bạn. Tuy nhiên, với Vimperator, bạn chỉ cần đặt tay trên bàn phím, k cần di chuyển quá nhiều. Các phím tắt, command có thể giúp bạn!
### 2. Học hỏi được nhiều từ việc custom
Vimperator rất mở cho developer. Bạn có thể tự mở rộng, custom nó theo ý của riêng mình. Ngoài việc có thể remap lại các phím, bạn cũng có thể add thêm các function mới bằng cách viết các script (sử dụng Javascript hoặc Vimscript). Chỉ cần bạn có một ý tưởng mới, bạn hoàn toàn có thể viết code để thực hiện nó với Vimperator.

## Sử dụng Vimperator
Khi đã cài Vimperator, bạn có thể xem các config mặc định, các cách custom bằng cách bấm phim F1, một trang help sẽ được hiển thị, hướng dẫn các bạn mọi thứ để có thể sử dụng Vimperator một cách tốt nhất.

Cá nhân mình thấy tài liệu hướng dẫn này khá dài, đối với người mới bắt đầu thì không cần thiết đọc hết từng đó. Bạn chỉ cần tham khảo qua [cheatsheet](http://sheet.shiar.nl/vimperator) này. Biết cách dùng cơ bản rồi sau đó khúc mắc gì có thể google thêm.

Giống như `Vim`, Vimperator có nhiều mode như Insert, Command, Visual, Caret, ... Hầu hết các keybinding, hint, ... đều được gọi ở chế độ Command.

* Di chuyển lên/ xuống/ sang trái, phải bằng các keybinding như Vim:

Keybinding     | Feature   |
------------- | :-------------:|
h, j, k, l    | Sang trái, lên trên, xuống dưới, sang phải |
gg, G      | Lên đầu, xuống cuối trang      |
C-f C-b C-d C-u | Xuống, lên 1 page, xuống, lên nửa page     |
]], [[      | Next/ previous page(dành cho những trang có phân trang)      |
zi, zo    | Zoom in, zoom out      |
d, u    | Close tab or undo tab recent closed      |
o + url     | Mở url trong tab hiện tại      |
t + url     | Mở url trong tab mới      |
H, L     | Trở về/ tiến tới các trang trước, sau trong history      |
r, R     | Reload page hoặc reload page without cache     |
gi     | Focus vào textarea mà bạn vừa chỉnh sửa      |
gt / gT     | Next/ previous tab      |
g0 / g$     | Đi tới tab đầu / cuối      |
/ + keyword     | Search keyword trong trang. Dùng "n/ N" để di chuyển giữa các kết quả      |

và còn rất nhiều các keybinding thú vị khác nữa cho bạn. :D

* Ex command

Vimperator giúp bạn gọi được hầu hết các thao tác lệnh với firefox. Như `:quit, :restart, :addons, :reload, :quitall, :winopen, ...`

* Hint mode

Đây là tính năng mình thấy thú vị nhất của Vimperator. Khi bạn đang ở trong chế độ `Command`, bạn gõ phím `f` hoặc `F`, các thẻ \<a> sẽ được bôi vàng như hình dưới.

![](https://viblo.asia/uploads/9f337775-b768-4c6b-9964-8a9aa9f346a9.png)

Khi đó, muốn click vào đường link nào, bạn chỉ cần gõ kí tự gợi ý ở phía trên link là được.

* Firefox hỗ trợ người dùng add thêm các Search Engine khác nhau. Bạn có thể cài đặt ở url: `about:preferences#search`. Bạn có thể cài đặt thêm các Search Engine: Youtube (set keyword là chữ `y`), Google Translate (set keyword là chữ `t`), ...  Khi đó, nếu bạn muốn tra từ điển từ "command", bạn chỉ cần bấm `t + t + command` (tabopen translate "command") -> Trang Google Translate sẽ tra từ đó giúp bạn :D

Ngoài ra, Vimperator còn hỗ trợ rất nhiều tính năng hữu ích khác như mark support, recording keys, custom GUI, ...

## Custom Vimperator
### Config
Vimperator cung cấp rất nhiều option để bạn có thể tự mình custom lại extension này. Giống như Vim, để custom lại nó, bạn sẽ phải tạo một file `.vimperatorrc` trong thư mục Home trên Linux hoặc MacOS hoặc file `_vimperatorrc` trong thư mục `C:/Users/%USERNAME%/` trên Windows. Nếu bạn chưa có file này, bạn có thể gọi lệnh `mkvimperatorrc` bằng chế độ vimperator command trên Firefox để tạo file.

Cú pháp config của Vimperator khá giống với Vim. Khi bắt đầu học cách config Vimperator, mình thường google để tìm các hướng dẫn config. Tuy nhiên sau này, mình thấy cách này không hay bằng việc đi học lỏm người khác 😝 Chỉ cần lân la ở các thư mục `dotfiles` trên github, bạn có thể tìm ra cả đống config rất hay, lạ và tiện lợi trên này.

Đây là nội dung file [~/.vimperator](https://github.com/ttuan/dotfile/blob/master/vimperator/vimperatorrc) của mình.

### Theme
Để thêm/ thay đổi theme cho vimperator, bạn cần đặt các file theme tại thư mục: `~/.vimperator/colors`. Sau đó trong Firefox, bạn chạy lệnh `:colorscheme theme-name` để thay đổi theme.

Các bạn có thể tham khảo các theme có sẵn ở repo [vimperator colors](https://github.com/vimpr/vimperator-colors). Từ đó có thể tạo theme riêng cho mình :D

### Plugin
Cộng đồng những người sử dụng Vimperator có viết thêm rất nhiều plugin để có thể mang đến những trải nghiệm mới cho người dùng khi duyệt Web. Bạn có thể tham khảo list các Plugin ở [đây](https://vimpr.github.io/plugins-en.html). Cá nhân mình thấy trong list này có khá nhiều plugin hay, đặc biệt là plugin [goo.gl](http://github.com/vimpr/vimperator-plugins/blob/master/goo.gl.js) cho phép bạn lấy shorten URL một cách nhanh chóng. Ví dụ: để get shortlink của url hiện tại, bạn chỉ cần map phím và gõ: `<Space>sl` là shorten url đã được copy vào clipboard. Rất tiện lợi đúng không :D

Ngoài ra, trên github có có rất nhiều plugin của các developer khác, chỉ cần google với từ khoá `vimperator plugins github` bạn sẽ thấy rất nhiều. Hoặc bạn cũng có thể tự viết plugin cho riêng mình dựa vào cú pháp của các plugin có sẵn, ví dụ như plugin [YouTubePlayer](https://github.com/ttuan/dotfile/blob/master/vimperator/plugin/YoutubePlayer%20-%20Copy.js) này.

Để add thêm Plugin cho Vimperator, bạn cần đặt plugins vào trong thư mục `~/.vimperator/plugin`, sau đó gõ lệnh `loadplugins` trong Firefox là có thể sử dụng được.
## Kết luận
Mặc dù lý do ban đầu mình đến với Vimperator chỉ là vì khi dùng nó trông sẽ nguy hiểm hơn 😁😁😁😁😁 Tuy vậy, sau 1 thời gian sử dụng mình thấy Vimperator quả là một extension tuyệt vời ;)
Bài viết này giới thiệu sơ qua cho bạn về cách cài đặt, sử dụng và custom Vimperator. Nếu bạn muốn có những trải nghiệm mới khi duyệt web, hãy thử qua Vimperator nhé.

Cám ơn các bạn đã dành thời gian đọc bài viết :D
