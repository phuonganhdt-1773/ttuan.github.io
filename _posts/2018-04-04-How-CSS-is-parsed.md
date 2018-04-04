---
layout: post
comments: true
title: How CSS Work&#58; Part 1 - How CSS is parsed?
tags:
- Frontend
- BehindTheScenes
- CSS
---

## Mở đầu
Dạo gần đây mình phải ghép html cho 1 số trang của dự án trên công ty. Công việc này khá nhàn, chỉ cần xem từ file mock (đã được bên Front-end làm sẵn), sau đó mình ghép lại vào code view mà bên Backend đã viết, thế là đã có view đẹp lung linh =]] Tuy nhiên mình gặp phải 1 vài vấn đề, cần custom lại một vài nút hay layout (do KH không thuê design nữa nên phải tự bơi T.T), trong khi kiến thức về front-end của mình chắc bằng 0.1 :3

Sau vài lần cay cú vì làm cái gì cũng phải đi học, mình quyết định kiếm khóa front-end nào đó để học. Và được ku em công ty giới thiệu cho khóa này:
[Advanced css and sass](https://www.udemy.com/advanced-css-and-sass/). Khóa này mình thấy rất hay (hoặc do kiến thức front = 0.1 nên mình thấy cái gì cũng mới :v :v). Thú vị nhất là phần tác giả nói về cách mà Css được parse. Bài viết này mình sẽ tóm tắt lại quá trình parsed css, chủ yếu là để note lại cho nhớ :v

## How Css is parsed?
Điều gì xảy ra khi chúng ta load 1 trang web? Browser sẽ làm gì để có thể hiển thị được giao diện của trang web đó lên? Lúc bắt đầu biết tới css mình cũng có câu hỏi đó :v

Nói chung thì có khá nhiều việc, nhưng chúng ta có lẽ chỉ quan tâm tới 2 việc chính, ảnh hưởng tới việc hiển thị css. Đó là:

![](https://imgur.com/ciy8mtY)

Như vậy, chúng ta sẽ đi tìm hiểu 2 quá trình:
1. Giải quyết các conflict giữa các định nghĩa Css (tức là nếu 1 element được config/ thừa kế nhiều giá trị css khác nhau thì nên chọn cái nào để hiện thị?)
2. Tính toán các giá trị final value.


### 1. Resolve conflicting CSS declarations (Cascade)
Các declarations có thể đến từ nhiều nguồn khác nhau:
* Author - Các css mà dev viết.
* User - Css mà người dùng thay đổi (vd như người dùng thay đổi font-size của trình duyệt -> 1 declaration cho font-size)
* Browser (User agent) - Css mà trình duyệt định nghĩa sẵn (vd như các đường link thì sẽ được in chữ màu xanh, có gạch chân)

Khi mỗi source đều có css khác nhau cho 1 element, browser sẽ chọn css theo quy tắc:

> Important (weight) > Specìficity > Source Order

#### a. Important
1. User `!important` declarations.
2. Author `!importnat` declarations.
3. User declarations.
4. User declarations.
5. Default browser declarations.

--> Định nghĩa css mà có `!important` sẽ được ưu tiên nhất. Tuy nhiên, việc lạm dụng `!important` sẽ gây nhiều khó khăn kh maintain :3

Khi các rule có cùng mức độ quan trọng (importance), browser sẽ đi so sánh mức độ chi tiết (specificities).

#### b. Specificity

1. Inline styles.
2. IDs.
3. Classes, pseudo-classes, attributes
4. Elements, pseudo-elements

Khi có nhiều rule cùng áp dụng cho 1 slide, browse sẽ tính toán 4 giá trị trên và đem ra so sánh, sau đó chọn rule nào có giá trị cao nhất để đem ra hiển thị. Ví dụ:

```html
<nav id="nav">
	<div class="pull-right">
		<a class="button button-danger" href="link.html"> Don't click here! </a>
	</div>
</nav>
```

```css
.button {
	font-size: 20px;
	color: white;
	background-color: blue;
}    // (inline=0, id=0, class=1, elements=0) ---> (0, 0, 1, 0)

a {
	background-color: purple;
}	// (0, 0, 0, 1)

#nav div.pull-right a.button {
	background-color: orangered;
}	// (0, 1, 2, 2)

#nav a.button:hover {
	background-color: yellow;
}	// (0, 1, 1, 2)

```

Như vậy, khi so sánh, chúng ta sẽ thấy (0, 1, 2, 2) sẽ được chọn để hiển thị backgroud color cho `.button`.

---> Note:
* Inline style luôn có độ ưu tiên cao hơn style được viết trong file :v
* 1 `id` sẽ được ưu tiên hơn 1000 classes.
* 1 `class` sẽ được ưu tiên hơn 1000 elements.


#### c. Source order

Khi các css declaration có cùng specificity, declaratió cuối cùng trong code sẽ được chọn. :v

---> Note:
* Chúng ta nên thay đổi độ chi tiết của các declaration hơn là thay đổi thứ tự các declarations.
* Nếu phải dùng css của bên thứ 3, bạn cần lưu ý để đặt author stylesheet cuối cùng.

### 2. Process final CSS value

#### a. Process
Quá trình process css gồm 6 bước, được thể hiện bằng hình bên dưới:

![](https://i.imgur.com/X4WNipJ.png)

1. Declared value: Lấy tất cả các value đã được định nghĩa (author declarations)
2. Cascaded value: Lấy giá trị cascaded.
3. Specified value: Nếu không có giá trị cascaded, sẽ lấy giá trị default.
4. Computed value: Convert các relative values thành absolute (ví dụ: `color: red` thì sẽ chuyển thành mã màu gì)
5. Used value: Tính toán, dựa trên layout để đưa ra giá trị thích hợp. (ví dụ: 66% thì quy ra px là bao nhiêu) -> phần này có thể cần used value của các phần tử parent.
6. Actual value: Làm tròn (phụ thuộc vào browser và device)

===================================================================
Các units được convert như thế nào?

![](https://i.imgur.com/owaoak9.png)


Hãy chú ý vào các chữ được in đậm :v ---> Note:
* Default font-size của trình duyệt thường là 16px.
* Các giá trị % hoặc relative đều sẽ được covert ra pixels.
* Nếu dùng `font-size: xx%` -> giá trị sẽ được tính dựa vào **parent** font-size.
* Nếu dùng `width(padding, margin, ...): x%` -> giá trị sẽ được tính dựa vào **parent** length.
* `em` dùng cho `font-size` sẽ được tính bằng **parent** font-size.
* `em` dùng để tính length (padding, ..) sẽ cần dựa trên **current** font-size.
* `rem` luôn dựa vào document's root font-size (nếu k có sẽ dùng default font-size của trình duyệt.
* `vh` and `vw` được tính trên % của viewport height và width.

#### b. Inheritance

![](https://i.imgur.com/a5OE10s.png)

Note:
* Kế thừa cho phép pass values từ phần tử cha cho phần tử con :v
* 1 số properties được kế thừa: font-family, font-size, color,... (padding, margin sẽ không được kế thừa).
* Computed value là giá trị sẽ được dùng để kế thừa, không phải declared value.
* Chỉ lấy giá trị kế thừa nếu không có value nào được định nghĩa cho phần tử đó.
* Để force việc kế thừa cho 1 gía trị, ta có thể dùng từ `inherit`. Để reset giá trị initial của 1 propety, ta dùng keywork `initial`.


## Kết luận
Done. Hy vọng qua bài viết, bạn sẽ biết thêm về việc Css được parse như thế nào, các giá trị được tính toán ra sao. :v

Trong part 2, mình sẽ nói thêm về việc trình duyệt sử dụng những gía trị đã tính toán đó để hiển thị lên như thế nào :D

See you soon!!!
