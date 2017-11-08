---
layout: post
title: Awesome Alfred
tags:
- Mouse Free
- Vim
---

The purpose of this post is explaining how **Amazing** Alfred is 🤖

![](https://www.howtogeek.com/wp-content/uploads/2017/09/2017-09-14_0001.jpg)

# Mở đầu
> Alfred là phần mềm đầu tiên mà mình trả tiền để mua =]]

Số là gần đây, trong lúc mình có tìm hiểu về các phần mềm Productivity trên MacOS thì có đọc được bài tổng hợp [này](https://github.com/nikitavoloboev/my-mac-os). Trong bài này, anh tác giả đặt phần mềm [Alfred](https://www.alfredapp.com/) lên trên đầu. Mình thấy hơi tò mò, không hiểu tại sao nó lại được xếp lên đầu như thế. Thế là sau khi bỏ ra 1 buổi tối tìm hiểu, mình thật sự khá bất ngờ vì những thứ mà Alfred có thể làm được => Mình đã quyết định bỏ ra  💶23 để mua và mình nghĩ mình đã có một quyết định đúng đắn 😄

Trong bài viết này, mình sẽ viết về những thứ thú vị mà Alfred có thể làm được ;)
# Alfred basic
Sau khi cài đặt Alfred, bạn có thể show Alfred bằng cách bấm tổ hợp `⌥ Space` để show Alfred. Tuy nhiên do mình dùng nhiều nên mình đổi config thành `⌘ Space` 😄

Dưới đây là 1 số tính năng cơ bản của Alfred. Nó khá giống với ứng dụng `Spotlight` có trên MacOS.

### 1. Launch applications
`⌘ space app_name` để tìm app -> Ấn Enter để chạy :D
### 2. Search web
`google, wiki, .... + keyword` để search với từ khoá.
### 3. Find files quickly
`open + tên file` -> Mở file nhanh

`spacebar` -> Search file

`find` -> Follow keyword để mở trong Finder

`⌘ o` -> Open file
### 4. Speed up your Mac productivity
|Keyword| Tính năng|
|---|---|
|`define`| Tra từ điển bằng app mặc định|
| `13 * 123`| Làm toán cơ bản|
| `screensave`| Start Screen Save|
|`trash`| Show Trash|
|`emptytrash`| Empty Trash|
|`sleep`| Sleep Mac|
|`lock`, `restart`, `shutdown`| :D |
|`eject`| Eject all volumes|
|`hide`, `quit`, `forcequit`| Hide, Quit or Force Quit app|

### 5. Save your clips
`⌥ ⌘ c` -> Open Clipboard History

Sau khi bạn chọn -> `⌘ c` để copy ;)

### 6. Listen to Music
Alfred cung cấp rất nhiều phím tắt, keyword để tương thích với iTunes, nhưng do mình k hay nghe nhạc bằng iTunes nên là sẽ next qua phần này =))

### 7. Send email
Alfred giúp bạn gửi email nhanh chóng = từ khoá `email`.
Bạn cũng có thể search contacts từ Alfred ;)

===> Các tính năng cơ bản khác của Alfred như search Bookmarks, tương thích với 1Password, chạy lệnh Terminal, ... của Alfred bạn có thể xem thêm ở page [Guide and Tutorial](https://www.alfredapp.com/help/guides-and-tutorials/).

# Alfred Workflow
Đây là tính năng mình thấy là hay nhất của Alfred. Và tất nhiên, để có thể sử dụng tính năng này, bạn cần mua gói [Power Pack](https://www.alfredapp.com/powerpack/).

Workflow có thể hiểu như là extension/ plugin cho Alfred. Với tính năng này, Alfred trở thành 1 phần mềm scriptable -> Bạn có thể tạo/ viết code để tạo ra những tools để giải quyết vấn đề của mình.

Cá nhân mình thấy thì mọi vấn đề của mình đã được người khác giải quyết = các Workflows rồi 😅 Mình chỉ cần down về rồi dùng thôi :v Dưới đây là 1 số Workflows mà mình thấy thú vị 😃

### 1. Evernote
Nếu bạn là người thường xuyên sử dụng [Evernote](https://evernote.com/) cho việc take note, quản lý todo list, reminder, ... thì bạn k thể bỏ qua ứng dụng này.

[Evernote Workflow](https://www.alfredforum.com/topic/840-evernote-9-beta-2-for-alfred-3-search-create-append-set-reminders-all-within-alfred/) cho phép bạn có thể tìm kiếm (bằng `ens @`, `ens #`, `ens`), thêm note(`enn @notebook #tag1 #tag2 !reminder :Title`), thêm note từ clipboard, selected text, append text vào existed note, ... Còn rất nhiều tính năng khác nữa, bạn có thể vào link bên trên để tải/ đọc các tính năng của Workflow này nhé ;)

### 2. Short URL
Bạn có 1 đường link rất dài và muốn share/ lưu lại nhưng vì nó dài nên nhìn khá là bất tiện. Hãy short link lại dùng [Shortent Workflow](https://www.alfredforum.com/topic/935-workflowshorten-url-support-googl-bitly-tcn-jmp-isgd-vgd/).

Với url đã được copy vào clipboard, bạn chỉ cần gõ: `short ⌘ v enter` -> Url shortent sẽ được copy vào clipboard ;). Giờ chỉ cần paste để share thôi ;)

### 3. Download video
Bạn muốn download 1 video mà chưa tìm thấy nút download ở đâu. Một số trang như facebook/ youtube, muốn down video về là cả 1 loạt các bước :3 [DownVid](https://raw.githubusercontent.com/vitorgalvao/alfred-workflows/master/DownVid/DownVid.alfredworkflow) là 1 workflow cho phép bạn download video từ url. Ví dụ, bạn đang xem video nào đó trên Youtube, hãy copy url, gõ: `dv + enter` để down video về 1 folder cài đặt sẵn. Rất tiện lợi đúng không :D

### 4. WebScreenshot
Bạn muốn chụp 1 bức ảnh screenshot và share cho người khác. Thông thường sẽ là bấm chụp màn hình/ hoặc chọn 1 phần của màn hình, sau đó lưu lại, vào web rồi up ảnh -> share. ==> Khá nhiều thao tác đúng không :v

Hãy để [WebScreedshot](https://github.com/vitorgalvao/alfred-workflows/tree/master/WebScreenshot) giúp bạn.

Sau khi đã add vào Alfred, bạn có thể remap lại phím tắt để dùng thuận tiện hơn. Ví dụ mình đặt `⌃ I` để select vùng sẽ chụp screenshot. Sau khi đã chọn, workflow này sẽ tự động upload lên trang [https://i.imgur.com/](https://i.imgur.com/) Link ảnh trả về sẽ được copy vào clipboard. Bạn chỉ cần paste và share thôi :D

### 5. Search Safari and Chrome tabs
Nếu bạn có thói quen mở nhiều tab cùng 1 lúc trên Chrome hoặc Safari thì đây là 1 workflow mà bạn không thể bỏ qua.

Bạn có thể download workflow ở đây: [Search Safari and Chrome tabs](https://www.alfredforum.com/topic/236-search-safari-and-chrome-tabs-updated-feb-8-2014/)

Khi bạn đang có nhiều tab, bạn có thể search tab đó bằng cách gõ `⌘ Space t tên_tab` với t là alias để gọi tới workflow này :D

### 6. Secure shell
Bạn làm việc với nhiều server, công việc cần phải `ssh` vào server mỗi ngày? Bạn chán việc phải mở terminal lên, sau đó gõ những dòng dài: `ssh -i ..../abc.pem ubuntu@ip_address`? [Secure shell](http://www.packal.org/workflow/secure-shell) có thể giúp cho bạn.

Bằng cách tạo 1 file `config` trong thư mục `~/.ssh`, bạn sẽ có thể dùng `Secure shell` để ssh vào server 1 cách dễ dàng. Sercure shell sẽ search trong file `config` và đưa ra list gợi ý các server có thể login. Việc của bạn chỉ cần chọn thôi :D

Và còn rất nhiều, rất nhiều các workflow hữu ích khác. Bạn có thể tham khảo thêm ở các trang:

* [https://www.alfredapp.com/workflows/](https://www.alfredapp.com/workflows/)
* [https://github.com/zenorocha/alfred-workflows](https://github.com/zenorocha/alfred-workflows)
* [https://github.com/deanishe/alfred-workflow](https://github.com/deanishe/alfred-workflow)

# Alfred snippets
Alfred Snippets là một tính năng không thể bỏ qua khi bạn sử dụng gói [Powerpack](https://www.alfredapp.com/powerpack/). Nếu như trước đây, để sử dụng tính năng text expander, bạn cần sử dụng 1 số phần mềm khác như [Text Expander](https://smilesoftware.com/textexpander) hay [Typinator](http://www.ergonis.com/products/typinator/) thì giờ đây, Alfred snippet có thể giúp bạn.

> Nguồn cảm hứng để mình dùng tính năng Text Expander bắt đầu từ bài viết [Write one never write again](https://medium.com/@NikitaVoloboev/write-once-never-write-again-c2fa1f6c4e8).

Đơn giản là bạn sẽ map 1 số từ, câu mà bạn hay gõ thường ngày thành 1 **short world** nào đó. Nó hoàn toàn giống như khi bạn sử dụng snippets khi viết code với Sublime hay Vim vậy. Gõ 1 chữ ngắn gọn, cả đoạn template sẽ được in ra. (len)

Để tận dụng tối đa được tiện ích của Alfred snippets, theo mình nên làm theo các bước sau đây:

### 1. Tìm được các câu/ từ mà mình hay gõ hàng ngày
Tất nhiên :v Nếu bạn chỉ google, lên mạng tìm 1 vài bộ snippets mà mọi người hay dùng (ví dụ như [đây](https://www.alfredapp.com/extras/snippets/)) thì bạn sẽ vẫn chưa thể giải quyết triệt để vấn đề của mình. Với cá nhân mình, do 1 ngày mình gõ khá nhiều (code, chat, viết blog =)) ) nên các từ mình viết cũng khá nhiều. Ví dụ như:

* Sau khi code xong và chuyển qua gửi pullrequest, mình cần gửi link ticket vào box chat và nhờ mọi người review với cú pháp:

`Please review this pullrequest: _pullrequest\_url. Thank you :)`
* Khi gõ markdown, mình rất hay phải dùng tới cú pháp viết link markdown:

`[text_abc](link_url_copied_from_clipboard)`

* Trong bài viết này mình có sử dụng khá nhiều kí tự đặc biệt như `⌘ ⌃ ⇧ ⎋` hoặc emoji như `🤖 😄 👍`
* Nếu bạn sử dụng app [Fantastical](https://flexibits.com/fantastical) thì Alfred snippet có thể giúp bạn rất nhiều. Ví dụ `1h␣` cho `1 hour`, `a1p` cho `at 1 pm`, ....
* Các thông tin mà bạn cần nhập với tần suất cao, ví dụ như: số chứng minh thư, địa chỉ, email, ....

### 2. Chia thành từng bộ snippet và đặt alias
Sau khi đã tìm ra được các từ hay sử dụng, bây giờ bạn có thể mở Alfred snippet lên và tạo từng bộ alias 1, sau đó đăt alias tương ứng với mỗi từ đó.

Ví dụ như mình có để 1 số bộ snippet như: `Fantastical`, `Mac symbol`, `Markdown`, `Personal`, `Regular mess`, ....

Lưu ý là Alfred support `prefix` cho mỗi bộ snippet. Hãy tận dụng nó để tránh việc trùng giữa các alias

### 3. Đồng bộ các bộ snippet
Hãy sử dụng tính năng đồng bộ snippet của Alfred/ Dropbox để có thể lưu trữ lại hoặc sử dụng ở trên máy tính khác.

Bạn có thể tham khảo guide ở [Syncing Your Alfred Settings Between Macs
](https://www.alfredapp.com/help/advanced/sync/).

# Kết luận

Trên đây, mình có nói về 1 số tính năng free và có tính phí của Alfred. Đây là 1 phần mềm mình thấy rất hay nếu bạn biết tận dụng các tính năng của nó.

Đặc biệt, đây là 1 phần mềm dạng `scriptable`, hãy viết script để có thể đáp ứng nhu cầu, giải quyết các vấn đề mà bạn gặp phải nhé :D
