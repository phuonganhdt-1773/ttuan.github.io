---
layout: post
comments: true
title: Test guide for RoR
tags:
- Become a Rubyist
---

# Test Guide

Từ trước tới nay, việc viết test luôn là một việc quan trọng trong khi viết code. Quá trình phát triển phần mềm đã chỉ ra rằng, việc viết code mà k viết test sẽ khiến bạn rất chật vật khi phải maintain, refactor hay mở rộng code. Tuy nhiên, việc viết test còn mang lại rất nhiều lợi ích khác nữa. Bài viết dưới đây được tóm tắt lại từ [test guide for RoR](https://github.com/HaChan/ror_test_guide) sẽ trả lời giúp bạn các câu hỏi như: Tại sao cần viết test, cần test những gì và test như thế nào?,... Ở đây mình chỉ note lại những vấn đề mình cho là cần thiết/ phổ biến khi code và viết test RoR.

## Tại sao cần phải viết test

Mục đích lớn nhất của việc test đó là: **Giảm cost khi phát triển phần mềm**. Nó giảm được cost là vì các lý do sau:

### 1. Tìm bugs
Viết test có thể đưa ra các cases, phát hiện ra bugs một cách sớm hơn. Fix bug ở thời điểm đầu sẽ đơn giản hơn vì khi đó, sự phụ thuộc trong code chưa nhiều. Càng để đâu, khi dependencies trong code lớn, muốn fix bug, bạn sẽ phải fix ở rất nhiều file.

### 2. Cung cấp documentation
Theo thời gian, ta có thể quên dần spec, tuy nhiên, nếu nhìn vào test, ta có thể biết tại sao lại cần thiết kế code theo kiểu này. Nhìn vào test, ta có thể suy ra spec một cách nhanh + đơn giản.

### 3. Phát hiện điểm yếu trong cấu trúc code
Code không tốt sẽ dẫn đến việc viết test khó.
* Nếu như việc viết test cần setup nhiều và phức tạp thì chứng tỏ code đang thực hiện rất nhiều việc, chứa nhiều context.
* Nếu việc viết test mà kéo theo quá nhiều object hay đoạn code khác chứng tỏ code đang có nhiều dependencies.
* Việc viết test khó chứng tỏ các object khác sẽ khó sử dụng lại code hiện tại của bạn.

=> Để thực sự giảm cost, cả app code và test cần được thiết kế 1 cách cẩn thận.

## Cần test những gì

Trong hướng đối tượng, ta coi mỗi Object là 1 hộp đen. Các object trao đổi với nhau qua các messages. Với mỗi object, ta chỉ cần quan tâm tới 2 loại messages:
* Incomming mesages: là những method public của 1 object
* Outgoing messages: là những method public của object khác mà object đang cần test gọi tới.

Ví dụ:

```Ruby
class Foo
	def incomming
		Bar.new.outgoing
	end
end
```

=> **Mọi Incomming messages đều phải được test.** Ta cần test giá trị trả về của những messages này bằng cách so sánh giá trị trả về của nó với một giá trị mong muốn nào đó.

**Các Outgoing messages thì k cần phải test** vì nó đã được test ở chính object của mình rồi.

Outgoing messages được chia nhỏ thành 2 loại:
* Query messages: những methods không có hiệu ứng phụ (ghi file, lưu DB, gọi API, ...) -> K cần test.
* Command messages: những methods có hiệu ứng phụ. Ta sẽ phải test *việc gọi các methods này*. Sending object cần chứng minh là những messages này được gọi 1 cách chính xác. Những test này gọi là test hành vi. Test behavior bao gồm việc test xem message được gọi bao nhiêu lần, với tham số gì.

## Test như thế nào
Viết test trước khi code, bất cứ khi nào có thể.

### 1. Test incomming messages
Trong phần này ta sẽ sử dụng ví dụ sau:

```ruby
class Gear
  attr_reader :chainring, :cog, :rim, :tire
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog       = cog
    @rim       = rim
    @tire      = tire
  end

  def gear_inches
    ratio * Wheel.new(rim, tire).diameter
  end

  def ratio
    chainring / cog.to_f
  end
end

class Wheel
  attr_reader :rim, :tire
  def initialize(rim, tire)
    @rim       = rim
    @tire      = tire
  end

  def diameter
    rim + (tire * 2)
  end
end

Gear.new(52, 11, 26, 1.5).gear_inches
```
#### a. Xoá những methods không sử dụng
**Tất cả public messages của object đều cần có sự phụ thuộc**, tức là các object khác sẽ cần phải gọi tới những public messages này. Nếu message nào k được sử dụng tại bất kỳ đâu trong ứng dụngt hì messages này nên được xoá.

Như trong vd trên:

|Object|Incomming Message|Outgoing Message|Có phụ thuộc hay ko|
|:-----|:----------------|:---------------|:------------------|
|Wheel|diameter||Yes|
|Gear||diameter|No|
||gear_inches||Yes|
||ratio||Yes|

những methods mà không phụ thuộc vào bất kỳ object nào thì ta k nên viết test cho chúng, cùng với đó là xoá bỏ những methods này ra khỏi ứng dụng.

#### b. Đảm bảo public methods chạy đúng
Như trên đã đề cập, ta phải test bằng cách **đánh giá các giá trị trả về** khi hàm được gọi.

Test Wheel:

```ruby
describe Wheel do
  let(:wheel) {Wheel.new 26, 1.5}
  describe "#diameter" do
    it {expect(wheel.diameter).to eq 29}
  end
end
```

Việc test Gear lại mất thời gian hơn nhiều. Do hiện tại trong code, hàm `gear_inches` tạo và sử dụng 1 object khác. -> Thời gian chạy test và khả năng test fail tăng lên khá nhiều.

> Test chạy nhanh nhất khi mà nó động đến code ít nhất. Khi một object phụ thuộc vào nhiều object khác được test, no sẽ ảnh hưởng bởi không chỉ object đó mà còn các object mà nó phụ thuộc.

Việc viết test cho Gear gặp khó khăn -> ta nên nghĩ tới việc thay đổi code.

#### c. Tách biệt Object cần test
Ta có thể fix ví dụ trên thành:

```ruby
class Gear
  attr_reader :chainring, :cog, :wheel
  def initialize(args)
    @chainring = args[:chainring]
    @cog       = args[:cog]
    @wheel     = args[:wheel]
  end

  def gear_inches
    # The object in the'wheel' variable plays the 'Diameterizable' role.
    ratio * wheel.diameter
  end

  def ratio
    chainring / cog.to_f
  end
end
```

Gear bây giờ chỉ quan tâm tới object truyền vào có implement method `diameter` (Diameterizable). Do Wheel cũng là 1 Diameterizable nên ta có thể viết:

```ruby
describe Gear do
  let(:gear) {Gear.new chainring: 52, cog: 11, Wheel.new(26, 1.5)}
  describe "#gear_inches" do
    it {expect(gear.gear_inches).to eq 137}
  end
end
```

Tuy nhiên, vấn đề ở đây là: Nếu `Wheel` khởi tạo mất nhiều thời gian, hoặc trong 1 application tồn tại nhiều Ọbject khác cũng có role là `Diameterizable` thì việc sử dụng 1 class cụ thể để test là không hợp lý.

==> **Thay vì truyền vào 1 class cụ thể, ta có tểh truyền 1 object có chứa role**

Ta có thể implement 1 object giả để phục vụ cho test.

##### Tạo test double
Test double ám chỉ các object fake được sử dụng để thay thế object thật của ứng dụng. Ta có thể thay thế Wheel bằng 1 test double trong test của Gear như sau:

```ruby
class DiameterDouble
  def diameter
    10
  end
end

describe Gear do
  let(:gear) {Gear.new chainring: 52, cog: 11, wheel: DiameterDouble.new}
  describe "#gear_inches" do
    it {expect(gear.gear_inches).to eq 137}
  end
end
```
**stub** cung cấp 1 giá trị tĩnh cho 1 lời gọi hàm được thực hiện trong test.
Ta có thể dùng stub để viết lại đoạn code test bên trên như sau:

```ruby
describe Gear do
  let(:wheel) {double("wheel")}
  let(:gear) {Gear.new chainring: 52, cog: 11, wheel: wheel}
  describe "#gear_inches" do
    before {allow(wheel).to receive(:diameter){10}
    it {expect(gear.gear_inches).to eq 137}
  end
end
```

Như vậy là ta đã tách được Wheel và Gear ở trong test. Wheel chạy nhanh hay chậm thì giờ cũng k ảnh hưởng tới Gear nữa.

Tuy nhiên, ở đây lại xảy ra 1 vấn đề, đó là *ta chưa test được Role `Diameterizable` trong hệ thống*. Tức là nếu ta thay hàm `diameter` của Wheel thì test vẫn pass mà ứng dụng sẽ gặp lỗi nếu như ta truyền Wheel vào Gear.

-> Khi interface của 1 role bị thay đổi thì tất cả các object thuộc role đó cũng cần phải được update theo, và ta cũng cần test để đảm bảo cho các role này.

##### Test Role
Để kiểm tra mỗi object đều implement cùng 1 role nào đó, ta cần test mỗi object đều implement các method thuộc role.

Trong RSpec, ta dùng `shared example` để chia sẻ test cho các object cùng implement role đó.

Ví dụ:


```ruby
class Wheel
  attr_reader :rim, :tire
  def initialize(rim, tire)
    @rim       = rim
    @tire      = tire
  end

  def diameter
    rim + (tire * 2)
  end
end

class DiameterDouble
  def diameter
    10
  end
end
```

rspec

```ruby
RSpec.shared_examples "a Diameterizable" do
  describe "#diameter" do
    it "should respond to diameter" do
      expect(diameterizable).to respond_to :diameter
    end
  end
end

RSpec.describe Wheel do
  let(:wheel) {Wheel.new 26, 1.5}
  it_behaves_like "a Diameterizable" do
    let(:diameterizable) {wheel}
  end
end

RSpec.describe DiameterDouble do
  it_behaves_like "diameterizable" do
    let(:diameterizable) {DiameterDouble.new}
  end
end
```

-Sử dụng `double` của rspec:

rspec

```ruby
RSpec.shared_examples "a Diameterizable" do |diameterizable|
  describe "#diameter" do
    it "should respond to diameter" do
      expect(diameterizable).to respond_to :diameter
    end
  end
end

RSpec.describe Wheel do
  let(:wheel) {Wheel.new 26, 1.5}
  it_behaves_like "a Diameterizable", wheel
end

describe Gear do
  let(:wheel) {double("wheel")}
  let(:gear) {Gear.new chainring: 52, cog: 11, wheel: wheel}
  #...
  it_behaves_like "a Diameterizable", wheel
end
```


### 2. Test outgoing messages
#### a. Query method
Không cần thiết, do những method này đã được test ở những object implement chúng.

#### b. Command method

Ví dụ:

```ruby
class RegisterUser
  def initialize user, mailer
    @user = user
    @mailer = mailer
  end

  attr_reader :user, :mailer

  def perform
    user.save
    mailer.deliver_later
  end
end
```

user.save làm nhiệm vụ lưu user vào DB, mailer.deliver_later làm nhiệm vụ gửi mail. Cả 2 methods này đều là command method do chúng tồn tại side effect. Ở đây ta cần test instance của Register User khi gọi hàm perform thì nó sẽ phải gọi hàm `save` của `user` object và hàm `deliver_later` của `mailer` object.

**Mock** được sử dụng để test hành vi. Mock định nghĩa sự mong đợi 1 method được gọi bởi method khác khi nó được chạy.

Ví dụ:

```ruby
describe RegisterUser do
  let(:user) {double "user"}
  let(:mailer) {double "mailer"}
  let(:register_user) {RegisterUser.new user, mailer}

  describe "#perform" do
    it do
      expect(user).to receive :save
      expect(mailer).to receive :deliver_later
      register_user.perform
    end
  end
end
```

## Kết luận
Trên đây là 1 số vấn đề đáng quan tâm trong test. Mình tổng hợp lại một số vấn đề:

* Cần nắm rõ lý do + lợi ích của việc viết test.
* Có 2 kiểu messages: Incomming và Outgoing messages. Để test Incomming message, ta có thể dùng Stub, test Role dùng shared example. Test Outgoing message, ta dùng Mock.

Ngoài ra, trong bài gốc còn chia sẻ cách viết Test kế thừa, private, .... Các bạn có thể tham khảo thêm tại link gốc.

