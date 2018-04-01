---
layout: post
comments: true
title: How Google Authenticator Work?
tags:
- Google
---

## Mở đầu
Nếu bạn có tài khoản thẻ tín dụng HSBC, có lẽ bạn biết tới 1 thiết bị nhỏ, nhìn như máy tính bỏ túi, thường được gắn vào chùm chìa khoá của 1 số người. Tính năng của nó là có thể sinh ra một đoạn mã, gồm 6 kí tự, để bạn có thể sử dụng để đăng nhập vào tài khoản HSBC online. Trước đây mình khá thắc mắc là tại sao 1 thiết bị không kết nối mạng lại có thể tạo ra 1 mật khẩu dùng để đăng nhập tài khoản online? Làm thế nào mà cả bên app offline và web online có thể đồng bộ đuộc mật khẩu đó? Có thắc mắc nhưng chẳng bao lâu sau mình quên ngay (do lười =]]).

Đợt này mình join vào 1 dự án, khách hàng yêu cầu cần bật tính năng Xác minh 2 bước (two-factor authentication) cho cả tài khoản Slack và Github. Trong hướng dẫn enable 2FA của Github có hướng dẫn sử dụng [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=vi). App này có thể sử dụng để generate one time password cho cả Github và Slack. Và mình quyết tâm tìm hiểu xem app này hoạt động như thế nào :v 

## How It Work?
Phần lớn các app dạng như Google Authenticator đều cài đặt dựa trên thuật toán [Time-based One-Time Password](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm). Bạn có thể tham khảo thuật toán trên Google. Ở đây mình chỉ tóm tắt lại. Thuật toán muốn hoạt động thì cần có: 

* Một **shared secret** (thường sẽ là 1 chuỗi các bytes).
* Một **input từ thời gian hiện tại**.
* Một **hàm đăng nhập**.

Sau đây ta sẽ cùng tìm hiểu xem từng nhân tố này đóng vai trò ntn trong thuật toán.

### 1. Shared Secret
Một **shared secret** là điều kiện bắt buộc để có thể cài đặt thuật toán trên điện thoại của bạn. Thường khi bạn bắt đầu, client app sẽ yêu cầu bạn phải quét một mã QR hoặc nhập bằng tay chuỗi kí tự. Chuỗi kí tự bí mật đó là một chỗi base32-encoded. Lý do tại sao không sử dụng base64 thì bạn có thể tham khảo ở [đây](https://en.wikipedia.org/wiki/Base32#Advantages).

Thông thường thì các service của Google sẽ hiển thị secret key theo format sau:
```
xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx
```

Giá trị này là một chuỗi 256 bits nhưng có thể nhỏ hơn trong 1 vài services khác. Những QR code khác cũng chứa cùng thông tin token nhưng dưới dạng URL:

```
otpauth://totp/Google%3Ayourname@gmail.com?secret=xxxx&issuer=Google
```

### 2. Input (Current Time)
Giá trị `input` là thời gian hiện tại, sẽ được lấy rất đơn gian từ điện thoại của bạn mà không cần phải tương tác với server một khi bạn đã setup thành công client app. Tuy nhiên, có một lưu ý rất quan trọng đó là: **thời gian trên máy điện thoại của bạn phải chính xác**. 

Nếu bạn đổi thời gian trên máy -> input cho thuật toán khác đi -> một chuỗi số bảo mật khác sẽ được sinh ra và tất nhiên chuỗi này sẽ không map được với chuỗi kí tự được sinh ra trên server -> không thể đăng nhập được.

### 3. Signing Function
Hàm signing được sử dụng là **HMAC-SHA1**. HMAC là viết tắt của từ [Hash-based mesage authentication code](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code) và nó là một thuật toán sử dụng hash function mã hoá 1 chiều(trong trường hợp này là [SHA1](https://en.wikipedia.org/wiki/SHA-1)) để sinh ra value. Sử dụng HMAC sẽ đảm bảo cho việc xác thực người dùng. HCir những người nào biết chính xác secret key mới có thể tạo ra được cùng output khi đã biết input (là thời gian hiện tại). Thuật toán được mô tả như công thức sau:

```
hmac = SHA1(secret + SHA1(secret + input))
```
Ngoài thuật toán TOTP, còn một thuật toán được sử dụng khá rộng rãi để sinh mật khẩu, tên là HOTP hay [HMAC-Based One-Time Password Algorithm](https://tools.ietf.org/html/rfc4226). Thuật toán này khá giống với TOTP, chỉ khác là nếu TOTP sử dụng thời gian hiện tại là input thì HOTP sử dụng 1 incrementing counter, counter này sẽ được phải được đồng bộ trên cả client và server.

### 4. Algorithm
Đầu tiên, chúng ta sẽ cần decode base32 chuỗi secret key. Base32 không cho phép kí tự space mà chỉ chấp nhật các chữ cái in hoa. 

```
original_secret = xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx
secret = BASE32_DECODE(TO_UPPERCASE(REMOVE_SPACES(original_secret)))
```

Bước tiếp theo, chúng ta sẽ lấy input từ current time. Thời gian được sử dụng sẽ là [UNIX time](https://en.wikipedia.org/wiki/Unix_time), hoặc số lượng giây từ lúc bắt đầu tạo ra máy tính:

```
input = CURRENT_UNIX_TIME()
```

Nhưng ở đây có một điểm đáng chú ý, đó là nếu lấy thời gian là `current_time`, bạn sẽ không thể nhập kịp code vì thời gian thay đổi từng giây, bạn khó có thể nhập xong được chuỗi 6 kí tự trong vòng 1 giây được. Đó là lý do Google Authenticator sinh ra code hợp lệ, có thể sử dụng được trong 1 khoảng thời gian (thường sẽ là 30 giây). Như vậy, công thức tính input sẽ thành: 

```
input = CURRENT_UNIX_TIME() / 30
```

Công thức này còn cho phép sự chênh lệch thời gian giữa server và client ở 1 mức chấp nhận được (30s) :v

Tổng hợp lại, chúng ta sẽ có một signing function, HMAC-SHA1:

```
original_secret = xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx
secret = BASE32_DECODE(TO_UPPERCASE(REMOVE_SPACES(original_secret)))
input = CURRENT_UNIX_TIME() / 30
hmac = SHA1(secret + SHA1(secret + input))
```

Mọi thứ đã khá rõ ràng, và chúng ta có thể dừng lại ở đây :v . Tuy nhiên, có 1 điểm là HMAC output là một chuỗi kí tự khá dài (do nó sử dụng SHA1 mà :v) - một chuỗi 20bytes, 40 kí tự hex và không ai muốn nhập 1 chuỗi 40 kí tự vào mỗi lần đăng nhập cả T.T Chúng ta cần một chuỗi 6 kí tự.

Để convert chuỗi 20-bytes HSA1 thành 1 chuỗi 6 kí tự, chúng ta cần phải cắt bớt nó đi 1 ít. Chúng ta sẽ sử dụng 4 bits cuối của SHA1.(Giá trị kí tự từ 0 đến 15) để đánh index cho chuỗi 20-bytes được sinh ra bên trên. và sử dụng tiếp 4 bytes liền sau đó cho vào index. 

```
four_bytes = hmac[LAST_BYTE(hmac):LAST_BYTE(hmac) + 4]
```

Chúng ta có thể chuyển chúng thành chuẩn 32 bit unsigned integer (4 bytes = 32 bit).

```
large_integer = INT(four_bytes)
```

Bây giờ thì chúng ta đã có một con số, trông dễ nhìn hơn. Tuy nhiên, đây vẫn là 1 con số rất lơn (2^32 -1). Chúng ta có thể sinh ra chuỗi 6 kí tự bằng cách lấy phần dư của phép chia mod cho 1000000.

```
large_integer = INT(four_bytes)
small_integer = large_integer % 1,000,000
```

Bằng cách này, chắc chắn ta sẽ thu được 1 số có 6 chữ số =]]

Tổng kết lại, thuật toán hoàn chỉnh của chúng ta sẽ là: 

```
original_secret = xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx
secret = BASE32_DECODE(TO_UPPERCASE(REMOVE_SPACES(original_secret)))
input = CURRENT_UNIX_TIME() / 30
hmac = SHA1(secret + SHA1(secret + input))
four_bytes = hmac[LAST_BYTE(hmac):LAST_BYTE(hmac) + 4]
large_integer = INT(four_bytes)
small_integer = large_integer % 1,000,000
```

## Kết luận
Trên đây mình đã trình bày cách hoạt động của Google Authenticator, bạn có thể hiểu rõ hơn tại sao 1 thiết bị không cần có mạng lại có thể sinh ra code bảo mật để đăng nhập online. Nếu muốn, bạn có thể tự code một chương trình để sinh mã bảo mật cho riêng mình bằng cách modify một vài bước trong thuật toán trên. =))

Thuật toán trên đã được implement lại bằng Go tại repo: [https://github.com/robbiev/two-factor-auth](https://github.com/robbiev/two-factor-auth)

Thông tin trong bài viết của mình được tham khảo từ blog: https://garbagecollected.org/2014/09/14/how-google-authenticator-works/

Happy coding =))













