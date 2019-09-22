---
layout: post
title: How to get Instagram public information
tags:
- Dev
---

![](https://blog-assets.hootsuite.com/wp-content/uploads/2018/11/how-to-get-verified-on-Instagram.jpg)

## Tản mạn
Nghe qua tiêu đề bài viết thì có thể bạn sẽ cười khẩy =)) "Lấy public thông tin thì khó gì :v Cứ vào thẳng trang Instagram mà lấy? Không thì lấy qua API của Instagram, cái này chắc chắn nó phải có API hoặc SDK cho mình lấy rồi chứ."

Theo logic thì Instagram là một mạng xã hội rất nổi tiếng, lại được Facebook chống lưng, chắc chắn phải support tận răng cho các developers rồi. Và ban đầu mình cũng nghĩ như thế. Kiểu gì chẳng có API, rồi các gem của ruby chắc cũng có sẵn hết rồi. Tạo tài khoản, lấy token, cho vào gọi API lấy thông tin là ra. (ez) Cơ mà ...

Gần đây mình có nhận một task đó là: Đầu vào là `username` của 1 tài khoản, ta cần phải lấy được số lượng followers và media của tài khoản đó. Bài viết dưới đây ghi lại roadmap mà mình đã thực hiện từ khi nhận task đến lúc tìm được 1 solution "tạm ổn" =))

## Strategy
Lưu ý: Nếu bạn đến đây để solution luôn, k muốn đọc lan man dài dòng, hãy move luôn tới mục 3 nhé :v

### 1. Instagram Legacy API
Việc đầu tiên mình thử đó là Google với từ khoá "Instagram developer" để đọc thử xem bọn này hỗ trợ developer thế nào, thế là tìm ngay được trang này: [https://www.instagram.com/developer/](https://www.instagram.com/developer/). Nhưng đọc ở ngay trang đầu thì có vẻ không ổn lắm:

> To continuously improve Instagram users' privacy and security, we are accelerating the deprecation of Instagram API Platform, making the following changes effective immediately. We understand that this may affect your business or services, and we appreciate your support in keeping our platform secure.

> These [capabilities](https://www.instagram.com/developer/changelog/) will be disabled immediately (previously set for July 31, 2018 or December 11, 2018 deprecation). The following will be deprecated according to the timeline we [shared previously](https://developers.facebook.com/blog/post/2018/01/30/instagram-graph-api-updates/):

> * Public Content - all remaining capabilities to read public media on a user's behalf on December 11, 2018

> * Basic - to read a user’s own profile info and media in early 2020

> For your reference, information on the [new Instagram Graph API](https://developers.facebook.com/products/instagram/).

Hmm. Instagram API đang dần ngưng support các API trước đây của họ :3 Sẽ deprecation từng phần API một và sẽ ngưng support hẳn vào đầu năm 2020.

Tuy nhiên mình vẫn thử xem dùng thằng này có lấy được thông tin gì không :v

Việc đăng ký 1 Client từ trang này không khó, chỉ cần vào trang [https://www.instagram.com/developer/clients/register/](https://www.instagram.com/developer/clients/register/), điền đầy đủ thông tin là bạn đã có thể tạo 1 client rồi :D

Sau khi đã có app rồi, bạn sẽ lấy được `Client ID` và `Client Secret`, chỉ cần lấy cặp này, add config vào cho một số gem như [instagram](https://github.com/facebookarchive/instagram-ruby-gem), [instagram-api-gem](https://github.com/agilie/instagram_api_gem), ... và bạn có thể lấy được thông tin trên Instagram.

Nhưng ... Vâng, lại chữ nhưng này =)) Bạn chỉ có thể lấy được thông tin cá nhân của bạn. :( Theo như doc trên trang của Insta, chế độ sandbox chỉ lấy được những thông tin sau:

> The behavior of the API when you are in sandbox mode is the same as when your app is live, but comes with the following restrictions:

> * Data is restricted to sandbox users and the 20 most recent media from each sandbox user
> * Reduced API rate limits


Okie, vậy là API này vẫn dùng được. Chỉ cần mình submit được app này lên trên Insta để họ duyệt là xong. Cơ mà ...

![](https://images.viblo.asia/1f3b9772-0c80-4799-a317-034ea23f866b.png)

Khi mình thử submit app để review thì Insta có đưa ra câu hỏi. Nếu click thử vào các option từ 1 đến 5, app bạn đều bị reject, hoặc sẽ nhận được gợi ý là k cần dùng Live làm gì cả, chỉ cần dùng sandbox là đủ :3

Nếu chọn option thứ 6, bạn sẽ có thể điền thông tin vào, sau đó gửi cho Insta review, tuy nhiên, yêu cầu khá ngặt và cần nhiều thông tin. Tất nhiên, 1 app còn chưa được release, mới đang trong giai đoạn phát triển như app của mình thì ... Nên thôi, tạm biết để đấy đã :v đi tìm thử xem cso giải pháp gì khác không :v

### 2. Non standard ways
Có một cách mà mình nghĩ đến ngay từ đầu, hơi củ chuối nhưng rất dễ làm. Đó là **Craw dữ liệu** =))

![](https://images.viblo.asia/d5b38e11-3cd3-4d2d-81cd-6bb3ac4751ac.png)

Dựa vào `href` của thẻ `<a>` như trong hình, chúng ta có thể lấy ra được số lượng lượt người follow :v Khá đơn giản. Tuy nhiên đơn giản để làm thì cũng đơn giản để toang :v Nếu 1 ngày Instagram thay đổi cấu trúc html, bạn sẽ k nhận được bất kì 1 cảnh báo gì, app sẽ lăn quay ra chết vào 1 ngày bình thường (không cần 1 ngày đẹp trời luôn :v ).

Trong quá trình Google, mình tìm thấy 1 cách rất rất thú vị, đến giờ vẫn không hiểu tại sao nó lại được truyền ra ngoài =))

Đó là, bạn chỉ cần truy cập đường link sau: `https://www.instagram.com/bluebottle/?__a=1`, với `bluebottle` là username của tài khoản, bạn sẽ nhận được 1 cục dữ liệu json. Parse đống dữ liệu này, bạn sẽ lấy được khá nhiều thôn tin, trong đó có cả thông tin về số lượng follower của account đó =))

Điều đặc biệt là trick này KHÔNG được nhắc đến trong bất kì 1 tài liệu chính thống nào của Instagram =)) Có thể là do 1 dev Instagram đã nghỉ việc tuồn ra ngoài chăng (yaoming) =))

Tuy có tồn tại 1 số cách không chính thống như trên, nhưng như đã nói ở đầu, mình muốn lấy public information, bao gồm cả followers và media, nên dùng theo 2 cách này rủi ro cao + chưa đáp ứng được => Tạm dẹp.

### 3. Instagram Graph API
Như trong trang Instagram Legacy API có nói, họ đang move dần qua support các API của Graph API, các bạn có thể tìm thấy tài liệu ở page này: [https://developers.facebook.com/docs/instagram-api](https://developers.facebook.com/docs/instagram-api).

Ngay ở đầu page, doc đã nói rất rõ là: Instagram Graph API **CHỈ** sử dụng để tương tác với các Instagram Business Accounts và Instagram Creator Accounts. Thật may là app mình đang dùng cũng chỉ giới hạn scope trong 2 loại tài khoản này chứ không cần tương tác với tài khoản cá nhân :v

Các bước chuẩn bị khá đơn giản, ta có thể xem được ở ngay page [Getting Started](https://developers.facebook.com/docs/instagram-api/getting-started). Để bắt đầu, bạn cần thực hiện theo các bước sau:

* Kiếm 1 tài khoản Instagram Business Account hoặc Creator Account. Để cho tiện thì mình [chuyển đổi tài khoản cá nhân sang tài khoản creator](https://help.instagram.com/2358103564437429).
* Tạo 1 page facebook và [connect tới tài khoản creator](https://developers.facebook.com/docs/instagram-api/overview#pages) ở trên. Mình dùng luôn tài khoản facebook cá nhân, tạo page rồi connect.
* Đăng ký 1 app facebook, settings basic. [https://developers.facebook.com/apps/](https://developers.facebook.com/apps/)

Làm thử theo đúng hướng dẫn ở page, mình cũng đã lấy được **instagram-page-id**, sử dụng token được lấy ở trang [Graph Tool Explorer](https://developers.facebook.com/tools/explorer) để GET thông tin về media của tài khoản insta cá nhân qua url:

```bash
curl -i -X GET \
 "https://graph.facebook.com/v4.0/17841405822304914/media?access_token={access-token}"
```

Tuy nhiên cái mình cần là followers + media của các tài khoản khác. Tất cả thông tin cần thiết đã được có trong page [Business Discovery](https://developers.facebook.com/docs/instagram-api/guides/business-discovery)

Khá rõ ràng: Bạn có thể thấy được thông tin public của tài khoản `bluebottle` dựa vào câu query:

```bash
# request
curl -i -X GET \
 "https://graph.facebook.com/v3.2/17841405309211844?fields=business_discovery.username(bluebottle){followers_count,media_count}&access_token={access-token}"

 # response
 {
  "business_discovery": {
    "followers_count": 267793,
    "media_count": 1205,
    "id": "17841401441775531" // Blue Bottle's Instagram Account ID
  },
  "id": "17841405309211844"  // ID of the Instagram account performing the query
}
```

Mọi chuyện có vẻ khá suôn sẻ, chúng ta đã lấy được thông tin mình cần, giờ chỉ cần tích hợp vào hệ thống là xong. Có vẻ ngon ;) Nhưng mình vẫn còn 1 thắc mắc.

Bình thường các tính năng liên quan tới facebook sẽ tuân theo follow:

- Server tạo app, xác định các quyền cần xin. sau đó từ App secret và App ID để tạo 1 đường link (hoặc pass vào Facebook SDK)
- User click vào link đó, sau đó FB sẽ show 1 popup để user confirm lại xem có đồng ý cấp quyền cho app đó hay không. Nếu người dùng click vào nút OK, facebook sẽ trả về cho phía client 1 access token, sau đó từ access token này, bên server của mình sẽ gọi để lấy thông tin ở các bước tiếp theo.

Ở đây, mình không cần người dùng phải thao tác với app mà chỉ dùng luôn tài khoản mình đã đăng ký (hay có thể hiểu "Server" và "User" trong follow bên trên kia là 1). Nên mình có thể tự lấy access token bằng Graph API tool, sau đó đặt token đó vào trong code server để gọi API.

Tuy nhiên, mình lại gặp 1 vấn đề, đó là: Server phải có background job chạy hàng ngày để update lại lượt followers, tuy nhiên, token mà mình đã lấy được bằng Graph API Tool lại không tồn tại lâu, nó có thể hết hạn sau 2 giờ, hoặc loại long-term support thì sẽ hết hạn sau 2 tháng.

Mình có thử google để tìm cách lấy fb token mà không bao giờ bị expire, nhưng đều nhận đc câu trả lời là không lấy được loại đó :v

[https://developers.facebook.com/docs/facebook-login/access-tokens/](https://developers.facebook.com/docs/facebook-login/access-tokens/)

Tuy không lấy được user access token có thời hạn mãi mãi, nhưng lại có 1 trick đối với những tài khoản sở hữu 1 page. Khá tiện vì để tương tác được với Instagram, mình cũng phải tạo 1 page và liên kết page đó vs tài khoản instagram ;) Bài hướng dẫn này là do khách hàng bên mình gửi cho =)) [[](https://goo-up.com/796/)](https://goo-up.com/796/)

Tóm lại là để lấy được token never expire, bạn cần thực hiện các bước sau:

+ Bước 1: Lấy token short term bằng [Graph API Tool](https://developers.facebook.com/tools/explorer), nhớ tick vào các quyền dưới đây:

 ![](https://goo-up.com/wp-content/uploads/inoue/02/03.jpg)

 + Bước 2: Đổi token đã lấy được ở bước 1 để lấy được 1 long-term token:

```bash
https://graph.facebook.com/v4.0/oauth/access_token?grant_type=fb_exchange_token&client_id=[CLIENT_ID]&client_secret=[CLIENT_SECRET]&fb_exchange_token=[TOKEN ở bước 1]
```

+ Bước 3:
	- Lấy thông tin cá nhân của tài khoản hiện tại bằng câu gọi: `https://graph.facebook.com/v4.0/me?access_token=[TOKEN ở bước 2]
`, copy trường `id`
	- Lấy never expired token bằng cách gọi:

	```bash
		https://graph.facebook.com/v4.0/【id】/accounts?access_token=[TOKEN bước 2]
	```
	Token bạn cần chính là token được trả về trong key "access_token" của response này.

Test thử token này ở trang [https://developers.facebook.com/tools/debug/accesstoken/](https://developers.facebook.com/tools/debug/accesstoken/), bạn sẽ thấy token này có thời gian Expires là "Never"

![](https://images.viblo.asia/a7267127-4727-4e9b-9901-ef4fee555afe.jpg)


Sau khi đã có được token này rồi, bạn chỉ cần thay nó vào để gọi API này là được ;)

```bash
curl -i -X GET \
 "https://graph.facebook.com/v3.2/17841405309211844?fields=business_discovery.username(bluebottle){followers_count,media_count}&access_token={access-token}"
```

Set token này vào 1 biến ENV ở trên server, sau đó viết cron job để chạy mỗi ngày 1 lần, update lại số lượt followers của user từ username ;) Done!!!

## Kết luận

Ông lớn không phải lúc nào cũng dễ chơi =)) Đôi khi document hoặc follow để làm 1 task liên quan tới Facebook, Google không hề đơn giản như bạn tưởng :v Trường hợp vs Instagram là 1 ví dụ.

Bài viết mang tính chất chia sẻ lại quá trình mình đã làm thế nào để cập nhật lượt followers của 1 tài khoản Insta. Hy vọng nó có ích cho các bạn ;)

Thank you for reading!!!
