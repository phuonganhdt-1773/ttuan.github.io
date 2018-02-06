# Dark corners of Unicode

## 1. Mở đầu

## 2. Some definitions
Để có được một cái nhìn tổng quan về Unicode, bạn có thể xem qua ở một số trang như [wikipedia](https://en.wikipedia.org/wiki/Unicode), [nedbat](http://nedbatchelder.com/text/unipain.html), [joel](http://www.joelonsoftware.com/articles/Unicode.html), ... Ở đây mình có thể tóm gọn lại như sau:

*Unicode* là một bảng rất lớn, map giữa những con số (*codepoints*) với các kí tự mà chúng ta sử dụng khi viết. Rất nhiều người thường dùng từ "Unicode" khi chúng ta muốn thể kiện "not ASCII", tuy nhiên, tất cả các kí tự ASCII đã được bao gồm trong Unicode.

*UTF-8* là một phương thức mã hoá, một cách để chuyển chuỗi các codepoints thành bytes. Tất cả các Unicode codepoints có thể được mã hoá trong UTF-*. ASCII cũng có thể được mã hoá, nhưng nó chỉ hỗ trợ 128 kí tự, chủ yếu là chữ caí Tiếng Anh và dấu câu.

Unicode được chia thành 17 tầng, được đánh số từ 0 đến 16. Tầng đầu tiên được gọi là Basic Multilingual Plan bởi nó chứa những chữ cái của các ngôn ngữ phổ biến. Các tầng còn lại thì bao gồm các kí tự ít được sử dụng hơn (symbol, emoji, ..)

## 3. Everything you know about text is wrong
Giả sử bạn đang muốn sort 1 mảng các chữ cái (một vấn đề khá phổ biến). Hãy cùng thử qua bằng Python nhé:

```python
>>> words = ['cafeteria', 'caffeine', 'café']
>>> words.sort()
>>> words
['cafeteria', 'caffeine', 'café']
```

Oops. Thuật toán sort của Python sẽ so sánh các Unicode codepoint của các chữ cái. Nên chữ "é" (U+00E9) sẽ lớn hơn chữ "f" (U+0066).

Bạn có biết là chữ cái "ß" trong tiếng Đức được suppose để sort như "ss"? Hay chúng ta sẽ sort chữ cái "æ" trong tiếng Icelandic ở đâu? Có tìm được chữ cái trong tiếng Anh giống với chữ "æ" ?

Tr
