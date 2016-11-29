---
layout: post
comments: true
title: Độ phức tạp của thuật toán
---

## Mở đầu
Là một lập trình viên, chắc hẳn bạn đã từng ít nhiều nghe tới khái niệm "Độ phức tạp của thuật toán". Rất nhiều người cho rằng độ phức tạp của thuật toán đại diện cho thời gian chạy nhanh hay chậm của 1 chương trình, nhưng liệu đây có phải là 1 quan niệm đúng? Bài viết dưới đây sẽ cho bạn cái nhìn tổng quan về độ phức tạp của 1 thuật toán.

## Tại sao cần đo độ phức tạp của thuật toán
Thông thường khi giải quyết 1 bài toán, chúng ta có thể đưa ra các giải thuật khác nhau nhưng sẽ phải chọn một giải thuật tốt nhất. Thông thường thì ta sẽ căn cứ vào các tiêu chuẩn sau:

* Giải thuật đúng đắn.

* Giải thuật đơn giản.

* Giải thuật thực hiện nhanh.

Để kiểm tra tính đúng đắn của 1 giải thuật, ta thường sẽ phải thử nó với một bộ dữ liệu nào đó rồi so sánh kết quả thu được với kết quả đã biết. Tuy nhiên điều này không chắc chắn vì có thể giải thuật này đúng với bộ dữ liệu này nhưng lại không đúng với bộ dữ liệu khác. Tính đúng đắn của 1 giải thuật cần được chứng minh bằng toán, tạm thời chúng ta không đề cập ở đây.

Đối với các chương trình chỉ dùng 1 vài lần thì yêu cầu giải thuật đơn giản sẽ được ưu tiên vì chúng ta cần 1 giải thuật dễ hiểu, dễ cài đặt, ở đây không đề cao vấn đề thời gian chạy vì chúng ta chỉ chạy 1 vài lần.

Tuy nhiên, khi 1 chương trình sử dụng nhiều lần, yêu cầu tiết kiệm thời gian sẽ được đặc biệt ưu tiên. Tuy nhiên, thời gian thực hiện chương trình lại phụ thuộc vào rất nhiều yếu tố như: cấu hình máy tính, ngôn ngữ sử dụng, trình biên dịch, dữ liệu đầu vào, ... Do đó ta khi so sánh 2 giải thuật đã được implement, chưa chắc chương trình chạy nhanh hơn đã có giải thuật tốt hơn. "Độ phức tạp của thuật toán" sinh ra để giải quyết vấn đề này.

## Cách để tính độ phức tạp của thuật toán

Độ phức tạp của một thuật toán là 1 hàm phụ thuộc vào độ lớn của dữ liệu đầu vào.Tìm
xem 1 đối tượng có trong danh sách N phần tử hay không?, sắp xếp tăng dần dãy số
gồm N số, ... N ở đây chính là độ lớn của dữ liệu đầu vào

Để ước lượng độ phức tạp của một thuật toán, người ta thường dùng khái niệm bậc O-lớn và bậc Theta (Θ)

### 1. Big O

 Ở đây ta dùng một đại lượng tổng quát là tài nguyên cần dùng R(n). Đó có thể là số lượng phép tính (có thể tính cả số lần truy nhập bộ nhớ, hoặc ghi vào bộ nhớ); nhưng cũng có thể là thời gian thực hiện chương trình (độ phức tạp về thời gian) hoặc dung lượng bộ nhớ cần phải cấp để chạy chương trình (độ phức tạp về không gian).

#### 1.1 Định nghĩa:

Giả sử f(n) và g(n) là các hàm thực không âm của đối số nguyên không âm n. Ta
nói "g(n) là O của f(n)" và viết là: g(n) = O(f(n)) nếu tồn tại các hằng số
dương C và n0 sao cho g(n) <= C * f(n) với mọi n >= n0

![BigO.png](https://viblo.asia/uploads/images/15710dd85247a62e3aa5a11089b38835729597e9/152bb61c9d16aa82125099294a73dabdb0271c06.png)




#### 1.2 Cách tính:

Độ phức tạp tính toán của giải thuật: O(f(n))

• Việc xác định độ phức tạp tính toán của giải thuật trong thực tế có thể tính bằng một số quy tắc đơn giản sau:

– Quy tắc bỏ hằng số:

            T(n) = O(c.f(n)) = O(f(n)) với c là một hằng số dương

– Quy tắc lấy max:

            T(n) = O(f(n)+ g(n)) = O(max(f(n), g(n)))

– Quy tắc cộng:

            T1(n) = O(f(n))                     T2(n) = O(g(n))

            T1(n) + T2(n) = O(f(n) + g(n))

– Quy tắc nhân:

            Đoạn chương trình có thời gian thực hiện T(n)=O(f(n))

            Nếu thực hiện k(n) lần đoạn chương trình với k(n) = O(g(n)) thì độ phức tạp sẽ là O(g(n).f(n))

![bigO.png](https://viblo.asia/uploads/images/15710dd85247a62e3aa5a11089b38835729597e9/226a551e491ae0112589aa661af1e0edbb91f54c.png)



#### 1.3 Ví dụ:


Ví dụ 1:
```
s=n*(n-1) /2;
```

Trong ví dụ trên, độ phức tạp của thuật toán là O(1)

Ví dụ 2:

```
s = 0;                        // O(1)
for (i=0; i<=n;i++){
  p = 1;                      // O(1)
  for (j=1;j<=i;j++)
    p = p * x / j;            // O(1)
  s = s+p;                    // O(1)
}

```

Số lần thực hiện phép toán `p = p * x / j` là n(n-1) / 2

=> Độ phức tạp của đoạn code này là O(1) + O(1) + O(n(n-1)/2) + O(1) + O(1) = O(n2)

Ví dụ 3:

```
 for (i= 1;i<=n;i++) {
    for (u= 1;u<=m;u++)
      for (v= 1;v<=n;v++)
        //lệnh
    for j:= 1 to x do
      for k:= 1 to z do
        //lệnh
}
```

=> Độ phức tạp của thuật toán này là: O(n*max(n*m, x*z))

### Theta và Omega

Tương tự như Big O, nếu như tìm được các hằng số C, k1, k2 đều dương, không phụ
thuộc vào n, sao cho với các số n đủ lơn, các hàm R(n), f(n) và h(n) đều dương
và `R(n) >= C.f(n)` va `k1.h(n) =< R(n) <= k2.h(n)` thì ta nói thuật toán có độ
phức tạp cỡ lớn hơn Omega(f(n)) và đúng bằng cỡ Theta(h(n))

Chúng ta có thể hiểu Big(O), Omega, Theta như những hàm tiềm cận của hàm tính độ phức
tạp của thuật toán.

![Theta.png](https://viblo.asia/uploads/images/15710dd85247a62e3aa5a11089b38835729597e9/965c619f8ad28bf58cde2e1a86a02da1cf3f9357.png)
![Omega.png](https://viblo.asia/uploads/images/15710dd85247a62e3aa5a11089b38835729597e9/ebc693e45bac6e7ab437384d3c42eb6ab04d40f2.png)



## Kết luận
Bài viết trên đã đưa ra 1 cái nhìn tổng quan về độ phức tạp của thuật toán. Rằng
đó không chỉ đại diện cho thời gian chạy nhanh/ chậm của 1 chương trình mà nó
đại diện cho những động thái của hệ thống khi kích thước đầu vào tăng lên.

Hy vọng sau bài viết này, mỗi khi bạn đặt tay viết 1 đoạn chương trình nào đó,
hãy cân nhắc, tính toán để đoạn chương trình có độ phức tạp trong mức cho phép.

Cám ơn các bạn đã dành thời gian đọc bài.

Nguồn tham khảo:

https://vi.wikipedia.org/wiki/%C4%90%E1%BB%99_ph%E1%BB%A9c_t%E1%BA%A1p_thu%E1%BA%ADt_to%C3%A1n
http://kcntt.duytan.edu.vn/Home/ArticleDetail/vn/168/2006/xac-dinh-do-phuc-tap-thuat-toan://kcntt.duytan.edu.vn/Home/ArticleDetail/vn/168/2006/xac-dinh-do-phuc-tap-thuat-toan
http://tek.eten.vn/danh-gia-do-phuc-tap-thuat-toan
