---
layout: post
title: How I met Vim
---


## Tản mạn
Mình biết đến Vim đã khá lâu, từ đợt còn học năm 2 đại học. Có lần lên lớp thấy thằng bạn cùng lớp "biểu diễn", lúc đó cũng thấy lạ lạ hay hay. Cảm giác xem nó code mà không hiểu nó gõ gì mà thao tác cứ nhanh như máy vậy, nhiều lúc không hiểu nó bấm gì mà có thể nhập, xóa, di chuyển con trỏ nhanh thế mà k cần đụng vào chuột.

Tối đó về nhà cũng search các kiểu, cài Vim vào dùng thử thì thấy mọi thứ chẳng
giống lúc mình xem đứa bạn thực hành gì cả. Giao diện xấu òm, highlight code
thì tù, không có check syntax, không có autocomplete, không compile tự động
được, ... Mà cái tù túng nhất của Vim là có hẳn 1 bộ các phím tắt và quy ước
sẵn, cần học thuộc đống quy tắc đó thì mới "pro" Vim được. Kết quả là sau 1 tối
nghiên cứu, mình bỏ luôn ý định học Vim. Quay trở về với DevC, với CodeBlock :v

Mọi chuyện sẽ chẳng có gì đáng nói nếu đến đợt tháng 2 năm nay, công việc của
mình yêu cầu cần phải thao tác rất nhiều với server. Hàng ngày ssh vào VPS,
chỉnh sửa code, config, ... bằng Vim. Lúc đó mình mới bắt đầu tìm hiểu và thực
sự thấy câu nói của [Tim Pope](https://github.com/tpope):

> Vim is forever.

quả thật không hề nói quá.

## Vim là gì và tại sao nên học Vim?
Vim là một trình text editor, được tích hợp sẵn trên một số distro của Linux, giúp bạn có thể chỉnh sửa text. Giao diện của nó gọn
gàng, đơn giản.

Vậy nếu Vim chỉ là một trình editor giống như Notepad, Gedit, Sublime,..., tại
sao chúng ta nên học Vim?

Khi mới bắt đầu, mình chỉ học Vim vì yêu cầu công việc, nhưng sau đó, mình phát
hiện ra rất nhiều điều thú vị từ Vim:

* **Fast and furious**: Nếu bạn đã quen dùng IDE, chắc hẳn bạn sẽ phải than
    phiền khá nhiều về tốc độ của nó. Từ Netbean, RubyMine, Pycharm, ... Mỗi khi
    khởi động hay compile ta sẽ phải đợi 1 lúc lâu. Với Vim, khởi động chỉ cần 1, 2s, tất cả các thao tác compile bạn đều phải chạy tay bạn sẽ phải chạy tay (hoặc có thể viết 1 đoạn script rồi map nó vào 1 phím nào đó), điều này sẽ giúp bạn hiểu thêm về các câu lệnh với compiler. Thêm
    vào đó, thành thạo Vim cũng là 1 lợi thế rất lớn đối với những người thường
    phải làm việc với server.
* **Edit text at the speed of thought**: Đây cũng là câu mở đầu trong cuốn
    ```Practical Vim```. Bạn có thể hình dung khi đã thành thục với Vim rồi,
    bạn có thể "trò chuyện" với editor để thực hiện ý định của mình.
* **Control everything**: Nếu các trình editor khác đã thiết kế sẵn giao diện,
phím tắt, bạn chỉ cần dùng luôn thì trong Vim, bạn sẽ phải thiết lập lại từ đầu.
Chính bạn tạo ra 1 editor cho riêng mình. Nếu bạn muốn thay đổi giao diện, map
phím cho phù hợp với thói quen hay
thêm 1 tính năng mới cho riêng mình, chỉ cần
sửa config trong ```.vimrc``` file. Mọi thứ trở nên rất đơn giản.

Và còn rất nhiều các điều thú vị nữa bạn sẽ khám phá ra được khi sử dụng Vim. ;)

## Cơ bản về Vim
Rất nhiều bạn ban đầu cảm thấy rất khó để có thể làm quen với Vim, lý do chính
là do bộ quy tắc rất khó học của nó. Ví dụ như muốn xóa 1 từ thì sẽ là
```daw``` hãy ```x``` cho tới bao giờ xong cái từ đó :v

Rất nhiều người cũng có suy nghĩ như bạn, và chính mình khi bắt đầu cũng như
vậy, cho tới khi mình đọc được bài viết
[này](http://stackoverflow.com/questions/1218390/what-is-your-most-productive-shortcut-with-vim).

Và để bắt đầu, bạn hãy mở terminal lên, gõ `vimtutor`, chỉ mất 15 phút để đưa
bạn đến với thế giới của Vim.


### Các mode trong Vim

Vim có nhiều mode nhưng thông thường, bạn sẽ làm việc chủ yếu với 3 mode cơ bản:

* Insert mode: Mode này cho phép bạn nhập, chèn các kí tự.
* Command mode: Mode này giúp bạn thực hiện các command, tương tác với text
object. Vd như nếu bạn gõ `d` trong Insert mode sẽ tạo ra kí tự "d" trên màn hình, nhưng
trong Command mode nó hiểu đây là 1 lệnh xóa(delete) một text object nào đó.
* Visual mode: Cho phép bạn select 1 vùng text nhất định nào đó.


Học Vim không hề khó như bạn nghĩ, chỉ cần bạn năm vững "cấu trúc câu" của nó.
Sau đó, tất cả các việc còn lại chỉ còn là: Hãy nói theo cách của Vim.

### Cấu trúc 1 câu lệnh của Vim luôn là:

> [number][command][motion/ text object]

Ta cùng phân tích cấu trúc trên 1 chút. Như bạn thấy, 1 câu lệnh trên có 3 phần:
number, command, motion/ text object. Number thì chính là số lần bạn sẽ thực hiện
câu lệnh, number này mặc định là 1. Phần quan trọng và cũng là phần thú vị
của Vim chính là Command và Motion/ text object.

* Command: Là các hành động mình muốn làm như thêm, xóa, sửa, thay thế, ...

Command | Action
------------ | -------------
x | Xóa 1 kí tự sau con trỏ
r | Thay thế 1 kí tự sau con trỏ
s | Xóa kí tự dưới con trỏ và chuyển sang chế độ insert mode.
d | Delete - Xóa text được định nghĩa bởi motion.
c | Change - Xóa text được định nghĩa theo motion sau đó tự chuyển về chế độ insert.
y | Yank - Copy text được định nghĩa bởi motion.

* Motion / Text object: Vim coi text trong file như 1 object: 1 từ, 1 câu, 1 {,
... mỗi thứ đó là 1 object, bạn có thể thao tác với các object đó.

Command | Motion
$ | Tới cuối dòng.
G | Tới cuối file
f. | "Find ." - Tới vị trí đầu tiên xuất hiện dấu "." sau con trỏ.
iw | "In word" - trong 1 từ.
it | "In tag" - trong 1 tag html.
i{ | "In {" - trong 1 dấu ngoặc.

Ngoài dùng `i`, Vim còn cung cấp `a`. Vd: ```i{``` là để chỉ tất cả các text có
trong dấu {} thì ```a{``` dùng để chỉ cả dấu {} và nội dung bên trong nó.

Done. Nếu bạn vẫn đọc tới đây thì phần thú vị nhất đã tới ;)

Việc duy nhất bây giờ bạn cần làm đó là rap nối các phần lại thôi.

* Bạn muốn xóa 1 từ: ```daw``` (delete a word)
* Bạn muốn thay thế nội dung trong dấu "": ```ci"``` (change in "")
* Xóa từ vị trí con trỏ tới cuối file: ```dG``` (delete to G-eof)
* Copy nội dung cho tới lúc gặp dấu ".": `yf.` (yank (to) find .)
* Bạn muốn di chuyển con trỏ xuống dưới 10 dòng: `10j`

Bạn có thể tự mình rap nối rất nhiều câu lệnh với Vim như vậy, chỉ cần nhớ rõ
cấu trúc câu của Vim là ok.
Trong các bài viết tiếp theo, mình sẽ giới thiệu với các bạn về các text object/
command mở rộng, được cộng đồng dùng Vim phát triển. Bạn sẽ thấy Vim thú vị hơn
rất nhiều ;)

Have happy time with Vim ;)





