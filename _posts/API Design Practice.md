# API Design Practice

Application Programming Interface

Tai sao chung ta can design API, tot hon gi so voi web thong thuong?
- Su phat trien chong mat cua cac cong nghe web, mo hinh cu k dap ung duoc, giam tai cho server, phat trien theo huong phan chia ro rang tung phan - signle responsibility (do k phai render view nua, chi quan tam vao logic backend)
- De dang mo rong, document lai, phat trien duoc cong dong lon manh (GG, Facebook, ... co the tao cac app khac, dang nhap = facebook, tuong tac qua command line, Tong hop tin nhan tu Chatwork, messenger, skype vao 1 cho...) Vi du ve app WSM, gio muon tu dong viet form xin di muon hoac nhu a Trung muon lam con BOT ma tu dong approve request di muon, neu k co API thi se rat kho de thuc hien => Neu k co API, se rat kho de cac developer khac co the tao ra cac app moi, gop phan quang ba duoc san pham cua minh tren nhieu platform :D 
- Co 2 loai API la Public API - public ra cho cac dev khac su dung, Private API de cho cac cong ty su dung de giao tiep giua cac san pham cua cong ty (Nhu tinh nang dang nhap = WSM chang han)



Key requirements: - DX - Developer experiences :D 
* Should use web standards - RESTFUL, SSL everywhere, great document, http status code, 
* Friendly to the developer and can be explorable via a browser address bar

## 1. RESTful URLs and actions
Key principles of REST la chia nho cac API thanh cac logical resources, cac request se su dung HTTP requests voi method get, post, put, patch, delete. Cach chon resources thuong la danh tu co nghia :v - giong cac model

GET /tickets - Retrieves a list of tickets
GET /tickets/12 - Retrieves a specific ticket
POST /tickets - Creates a new ticket
PUT /tickets/12 - Updates ticket #12
PATCH /tickets/12 - Partially updates ticket #12
DELETE /tickets/12 - Deletes ticket #12

Cung 1 endpoint (/tickets, /tickets/12) nhung khac HTTP methods thi se co cac action khac.

1 vai van de:

+ Singular vs Plural: Nen dung so it hay so nhieu? dai da so la dung so nhieu. Nhung 1 so API van dung so it, vd nhu github (pull/12 thay vi pulls/12)
+ Nested resources deal with relations. Tuy nhien co 1 so quan diem khac, dung Flat thay vi Nested: (/messages?ticket_id=12, 1 dang cua filter
+ Long url: /custommers/:custommer_uid/projects/:project_uid/orders/:order_uid/lines/:line_uid -> gioi han nesting depth ve <= 3
+ Se co nhung truong hop k dung dc restful :v vd nhu active 1 tai khoan, github api cho phep start a git via `PUT /gists/:id/star`, hoac search voi nhieu resources (youtube `search` ra ca channel, playlist, videos)

## 2. SSL everywhere
- Security, neu k co SSL thi cai simple access tokens thay vi sign each request 

## 3. Documentation
PHan lon cac developer se check docs truoc khi bat dau tich hop vao.
Show example ve cac request. cac doc nay se co the goi luon tren browser hoac curl

[Stripe](https://stripe.com/docs/api)

Nen dung swagger de tao docs

## 4. Versioning
Tai sao chung ta can danh version?
+ De tranh truy cap vao cac tinh nang da bi outdate, vd: Facebook API da bo 
======= TODO =============

API version nen dat o trong headers hay o trong URL? Nen o trong URL de nhin cho truc quan, nhu trong key requirements ben tren.

1 API khong bao gio la on dinh mai dc, thay doi la chac chan xay ra. Quan trong la chung ta quan ly thay doi ntn. Document + thong bao lich force-update cu the de nguoi dung co the theo doi duoc. 

Cach dat version cho API: z.y.x

If a change

is a bugfix, x is incremented.
adds a feature, y is incremented.
breaks backward compatibility, z is incremented.


Dua ra bai toan ve quan ly code va file routes.

+ File routes cang ngay se cang dai, minh can quan ly ntn
+ QUan ly code ntn, `app/controllers/v1/....`, `app/controllers/v2/...` Nhung nen dung thua ke hay copy code sang? Neu uu, nhuoc diem.

## 5. Filtering, sorting & searching
Tan dung toi da parametters
Nhu da noi o tren, tot nhat la lam moi thu ro rang ngay o tren URL.

+ Filtering: Vd de lay het cac ticket thi ta dang query: `/tickets`, nhung gio chi muon lay cac tickets co state open thi se query: `/ticktes?state=open`

+ Sorting: `GET /tickets?sort=-priority` de sap xep theo thu tu giam dan cua priority, `GET /ticktets?sort=-priority,created_at` 

+ Searching: `GET /tickets?q=key_word`

=> Combie lai: `GET /ticktes?q=key_word&state=open&sort=-priority,created_at`

Alias for common queries: `GET /tickets/recently_closed`

## 6. Limiting fields are returned by the API
Ly do: Nhieu client k can full thong tin cua 1 resouce. VD tren web man hinh to, co the hien thi ca avatar, username, first name, last name, email, ... Nhung tren dt man hinh nho hon, no chi show avatar thoi chang han. => minimize network traffic + speed up API.

Use `fields` query params de lay:

`GET /tickets?fields=id,subject,customer_name&state=open&sort=-updated_at`

## 7. Trong cac method Update & create, nen return resource
Nen tra ve resource da duoc update/create, tranh de client phai goi len 1 API de cap nhat gia tri moi.

## 8. JSON only response
truoc day moi ng dung XML, tuy nhien thi cai nay kho parse, kho doc, data model khong tuong thich.

Compare trend: [XML API vs JSON API](https://trends.google.com/trends/explore?date=all&q=xml%20api,json%20api)

## 9. Pagination
Nhu da noi o tren, nen lam URL nhin truc quan hon:

Trong response tra ve nen tra them cac thong tin:

```
{ 	
	success: true,
	data: [...],
	meta: {
		paging: {
			page: current_page,
			items: so luong item trong page,
			counts: total items,
			pages: total pages,
			prev: Link to page truoc,
			next: link to page sau
		}
	}
}
```

`GET /tickets?page=3&items=15`
Trong ruby thi su dung gem [pagy](https://github.com/ddnexus/pagy)

## 10. Auto load related resource
Se co nhung API can thong tin lien quan.
vd: Trong API ve article thi can tra ve them thong tin cua user chang han.

Trick la su dung `embed` query:

`GET /articles/12?embed=user.name,user.avatar`

## 11. Authentication
Su dung `OAuth 2` + bearer token
Neu con thoi gian thi se kiem tra ve flow su dung access_token luon.

## 12. Caching
su dung ETag va Last-Modified

## 13. Errors
Error rat quan trong, can quan tam k khac gi cac resource khac.

+ Format phai day du, ro rang, neu ro loi gi, description chi tiet.
+ Phai quy dinh ve ma loi day du
+ HTTP status code phai dung quy dinh: 4xx for client issues, 5xx cho server issues.

vd: 

{
	code: 123,
	message: "Somethign ... ",
	description: "...."
}

{
	code: 1123,
	message: "laaaa",
	errors: [
		{
			code: 123,
			field: "first_name",
			message: "klajdslkfjas"
		}, 
		{
		....
		}
	]
}

Nhan tien noi luon ve HTTP status code:
2xx - Success. 200 OK, 201 Created, 204 No content
3xx: Redirection : 304: Not modified
4xx: 400: Bad request, 401 - unauthorize, 403: Forbidden, 404 - Not Found, 410 - Gone
5xx: server error: 500 - Internal server error, 503: service unavailabe

## Tricks
+ Luon su dung cac tools check lint: rubocop, js-lint, ...
+ Luon viet test (controller, logic, ...)
