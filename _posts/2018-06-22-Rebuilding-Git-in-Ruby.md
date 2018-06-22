---
layout: post
title: Rebuilding Git in Ruby
tags:
- Ruby
- Git
- build-your-own-x
---

![](https://techtalk.vn/wp-content/uploads/2016/12/Git-logo.svg_-696x291.png)

## Tản mạn
Nếu bạn là 1 coder, chắc hẳn bạn vẫn đang sử dụng `git` hàng ngày. Đã bao giờ bạn tự hỏi là `git` hoạt động như thế nào chưa? Điều gì sẽ xảy ra mỗi khi chúng ta gõ những lệnh như: `git init`, `git add`, `git commit`, `git push`, ... Ngày não cũng gõ mấy lệnh này vài lần mà không đặt câu hỏi nó hoạt động thế nào thì cũng hơi phí đúng không =))

Đợt vừa rồi, lượn lờ trên [github trending](https://github.com/trending), mình đọc được 1 repo khá hay, tên là: [build-your-own-x](https://github.com/danistefanovic/build-your-own-x). Trong đó hướng dẫn bạn tự build lại rất nhiều thứ, từ BitTorrent Client, Blockchain/Cryptocurrency, Bot, Database, Docker, ... Và một trong những hướng dẫn đó là bài: Rebuilding Git in Ruby. Thấy bài viết khá thú vị + mình cũng là Rubyist nên là mình quyết định dịch lại bài này. 

Bài viết hướng dẫn build lại 3 commands cơ bản của Git là: `git init`, `git add` và `git commit`.

## Building Git in Ruby
### Git commands
Git được xây dựng dựa trên các modules và tuân thủ theo [các triết lý của UNIX](https://en.wikipedia.org/wiki/Unix_philosophy) - nhỏ gọn, hiệu quả. Mỗi command được thực thi bởi 1 file script với top level là lệnh `git`. Khi chúng ta gọi 1 lệnh (vd `git init`) thì từ khoá `init` sẽ được git nhận, tìm file `init` và chạy file đó. Dựa trên naming convention này, chúng ta sẽ build lại 1 số lệnh cơ bản của Git. 

```ruby
#!/usr/bin/env ruby

# bin/rgit

command, *args = ARGV

if command.nil?
  $stderr.puts "Usage: rgit <command> [<args>]"
  exit 1
end

path_to_command = File.expand_path("../rgit-#{command}", __FILE__)
if !File.exist? path_to_command
  $stderr.puts "No such command"
  exit 1
end

exec path_to_command, *args
```

File này sẽ làm các nhiệm vụ sau:

* Nếu không có subcommand nào được đưa vào thì sẽ in ra cấu trúc để sử dụng.
* Nếu không tìm được file chứa subcommand, in ra lỗi.
* Chạy subcommand nếu subcommand đó hợp lệ.

### Initializing a repository - git init
Git lưu trữ tất cả data và metadata trong thư mục `.git` trong thư mục gốc của project. Lệnh `git init` khi được gọi sẽ sinh ra một số thư mục/files :

```bash
.git
├── HEAD
├── config
├── objects
│  ├── info
│  └── pack
└── refs
    ├── heads
    └── tags
```

`HEAD` là file được hard code với giá trị: `ref: refs/heads/master`. File `config` sẽ chứa những thông tin về repo. Các thư mục còn lại, ban đầu sẽ là các thư mục rỗng.

Để implement lệnh `init`, chúng ta sẽ phải dùng khá nhiều lệnh `Dir.mkdir`:

```ruby
#!/usr/bin/env ruby

# bin/rgit-init

RGIT_DIRECTORY=".rgit".freeze
OBJECTS_DIRECTORY = "#{RGIT_DIRECTORY}/objects".freeze
REFS_DIRECTORY = "#{RGIT_DIRECTORY}/refs".freeze

if Dir.exists? RGIT_DIRECTORY
  $stderr.puts "Existing RGit project"
  exit 1
end

def build_objects_directory
  Dir.mkdir OBJECTS_DIRECTORY
  Dir.mkdir "#{OBJECTS_DIRECTORY}/info"
  Dir.mkdir "#{OBJECTS_DIRECTORY}/pack"
end

def build_refs_directory
  Dir.mkdir REFS_DIRECTORY
  Dir.mkdir "#{REFS_DIRECTORY}/heads"
  Dir.mkdir "#{REFS_DIRECTORY}/tags"
end

def initialize_head
  File.open("#{RGIT_DIRECTORY}/HEAD", "w") do |file|
    file.puts "ref: refs/heads/master"
  end
end

Dir.mkdir RGIT_DIRECTORY
build_objects_directory
build_refs_directory
initialize_head

$stdout.puts "RGit initialized in #{RGIT_DIRECTORY}"
```

Đoạn code trên có nhiệm vụ tạo thư mục `.rgit` chứa các thư mục cơ bản để lưu data + metadata. khi chúng ta gọi lệnh `rgit init` thì script này sẽ được chạy. Khá đơn giản đúng không =]]

### Adding files to the staging area - git add

Git cho phép chúng ta lưu lại 1 bản snapshot của trạng thái hiện tại bằng cách gọi lệnh `git add`. Tập hợp của các bản snapshots này gọi là `staging area`. List của các snapshots và metadata của nó được lưu trong thư mục `.rgit/index`. Để thêm 1 file vào staging, chúng ta cần: 

* Tạo 1 SHA dựa trên nội dung file.
* Tạo 1 blob bằng việc nén nội dung file.
* Lưu blob vào: `rgit/objects/<first-two-characters-of-sha>/<rest of sha>`
* Thêm SHA và path của file gốc vào index để chúng ta có thể gọi lại file đó sau này.

Index mà chúng ta vừa nhắc tới ở trên là 1 binary file, được tạo theo [format](https://github.com/git/git/blob/master/Documentation/technical/index-format.txt) này.

```bash
DIRC <version_number> <number of entries>

<ctime> <mtime> <dev> <ino> <mode> <uid> <gid> <SHA> <flags> <path>
<ctime> <mtime> <dev> <ino> <mode> <uid> <gid> <SHA> <flags> <path>
<ctime> <mtime> <dev> <ino> <mode> <uid> <gid> <SHA> <flags> <path>

# more entries
```

Nhiều metadata được đưa vào file này để giúp cho việc tính toán khi sử dụng các command khác. Mở thử file này, chúng ta sẽ thấy: (chẳng hiểu gì cả =)) vì nó được lưu dưới dạng binary format mà ^^)

```
cat .git/index
bin/rgit-initTREE52 1?Ibin/rgitU?U?2????        ???
C??B=????''9bin2 0
?Cԣ̏k?i??`V:??3'9Z?6??赠xa?cǢbF
```

Để đơn giản, chúng ta sẽ chỉ lưu những thông tin cần thiết, sử dụng text format vào file index này :v RGit's index file format sẽ có dạng:

```ruby
<SHA> <path>
<SHA> <path>
<SHA> <path>

# more entries
```

Giờ bắt đầu code thôi :v 

```
#!/usr/bin/env ruby

require "digest"
require "zlib"
require "fileutils"

RGIT_DIRECTORY = ".rgit".freeze
OBJECTS_DIRECTORY = "#{RGIT_DIRECTORY}/objects".freeze
INDEX_PATH = "#{RGIT_DIRECTORY}/index"

if !Dir.exists? RGIT_DIRECTORY
  $stderr.puts "Not an RGit project"
  exit 1
end

path = ARGV.first

if path.nil?
  $stderr.puts "No path specified"
  exit 1
end

file_contents = File.read(path)
sha = Digest::SHA1.hexdigest file_contents
blob = Zlib::Deflate.deflate file_contents
object_directory = "#{OBJECTS_DIRECTORY}/#{sha[0..1]}"
FileUtils.mkdir_p object_directory
blob_path = "#{object_directory}/#{sha[2..-1]}"

File.open(blob_path, "w") do |file|
  file.print blob
end

File.open(INDEX_PATH, "a") do |file|
  file.puts "#{sha} #{path}"
end
```

Giờ thì ta sẽ test thử bằng cách add file thêm vào staging area:

```bash
rgit add bin/rgit
```

Gọi lệnh `tree .rgit` chúng ta sẽ được kết quả như sau:

```
.rgit
├── HEAD
├── index
├── objects
│   ├── b3
│   │   └── 02dd6f8cd2b385b170e78c14503342c0ba6ae8
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```

Trong thư mục `objects`, chúng ta đã lưu bản nén của file `bin/rgit`. Bây giờ, check xem file index đang lưu gì nhé:

```
cat .rgit/index
b302dd6f8cd2b385b170e78c14503342c0ba6ae8 bin/rgit
```

### Committing files - git

Blobs là nội dung của 1 file cụ thể tại thời điểm cụ thể. Mỗi khi git lưu lại 1 bản snapshot của project, nó sẽ gói các sự thay đổi vào trong 1 commit.

Để lưu lại cấu trúc thư mục của 1 project, Git tạo ra một `tree object` cho mỗi thư mục của project. Mỗi một tree object chứa list các file đã tracked và các associated blob dưới dạng tree objects cho subdirectories.

Lệnh `commit` sẽ cần làm:

1. Build tree/blob object
2. Tạo 1 commit object để trỏ tới structure hiện tại.
3. Update branch hiện tại để trỏ tới commit vừa tạo ở bước 2.

Vì việc tạo 1 object là common task nên chúng ta sẽ tạo riêng ra 1 file:

```
# lib/rgit/object

require "fileutils"

module RGit
  RGIT_DIRECTORY = "#{Dir.pwd}/.rgit".freeze
  OBJECTS_DIRECTORY = "#{RGIT_DIRECTORY}/objects".freeze

  class Object
    def initialize(sha)
      @sha = sha
    end

    def write(&block)
      object_directory = "#{OBJECTS_DIRECTORY}/#{sha[0..1]}"
      FileUtils.mkdir_p object_directory
      object_path = "#{object_directory}/#{sha[2..-1]}"
      File.open(object_path, "w", &block)
    end

    private

    attr_reader :sha
  end
end
```

Giờ chúng ta sẽ viết code cho commit command:

```
#!/usr/bin/env ruby

# bin/rgit-commit

$LOAD_PATH << File.expand_path("../../lib", __FILE__)
require "digest"
require "time"
require "rgit/object"

RGIT_DIRECTORY = "#{Dir.pwd}/.rgit".freeze
INDEX_PATH = "#{RGIT_DIRECTORY}/index"
COMMIT_MESSAGE_TEMPLATE = <<-TXT
# Title
#
# Body
TXT

def index_files
  File.open(INDEX_PATH).each_line
end

def index_tree
  index_files.each_with_object({}) do |line, obj|
    sha, _, path = line.split
    segments = path.split("/")
    segments.reduce(obj) do |memo, s|
      if s == segments.last
        memo[segments.last] = sha
        memo
      else
        memo[s] ||= {}
        memo[s]
      end
    end
  end
end

def build_tree(name, tree)
  sha = Digest::SHA1.hexdigest(Time.now.iso8601 + name)
  object = RGit::Object.new(sha)

  object.write do |file|
    tree.each do |key, value|
      if value.is_a? Hash
        dir_sha = build_tree(key, value)
        file.puts "tree #{dir_sha} #{key}"
      else
        file.puts "blob #{value} #{key}"
      end
    end
  end

  sha
end

def build_commit(tree:)
  commit_message_path = "#{RGIT_DIRECTORY}/COMMIT_EDITMSG"

  `echo "#{COMMIT_MESSAGE_TEMPLATE}" > #{commit_message_path}`
  `$VISUAL #{commit_message_path} >/dev/tty`

  message = File.read commit_message_path
  committer = "user"
  sha = Digest::SHA1.hexdigest(Time.now.iso8601 + committer)
  object = RGit::Object.new(sha)

  object.write do |file|
    file.puts "tree #{tree}"
    file.puts "author #{committer}"
    file.puts
    file.puts message
  end

  sha
end

def update_ref(commit_sha:)
  current_branch = File.read("#{RGIT_DIRECTORY}/HEAD").strip.split.last

  File.open("#{RGIT_DIRECTORY}/#{current_branch}", "w") do |file|
    file.print commit_sha
  end
end

def clear_index
  File.truncate INDEX_PATH, 0
end

if index_files.count == 0
  $stderr.puts "Nothing to commit"
  exit 1
end

root_sha = build_tree("root", index_tree)
commit_sha = build_commit(tree: root_sha)
update_ref(commit_sha: commit_sha)
clear_index
```

File này làm những nhiệm vụ:

1. Exits với lỗi và in ra message nếu không có files nào được commit.
2. Tạo ra tất cả tree objects cho các file được commit vào trong file index.
3. Tạo 1 commit object và trỏ tới root tree object.
4. Update current branch để trỏ tới commit vừa tạo.
5. Xóa index.

Build tree này sẽ được thực hiện trong 2 bước. 1 là convert file index thành 1 hash. Bước 2 là convert nó thành tree object. Cả 2 bước đều dùng đệ quy.

Để lưu commit message, chúng ta sẽ mở file `COMMIT_EDITMSG` bằng text editor. Mỗi khi user exit editor, ta đọc lại file và đặt content vào trong commit.

Bây giờ chúng ta sẽ thử add thêm file `bin/rgit-add` vào staging và commit nó xem có điều gì xảy ra trong `.rgit` folder nhé:

```
.rgit
├── COMMIT_EDITMSG
├── HEAD
├── index
├── objects
│   ├── 63
│   │   └── 45493c987e6144cc68142ad2405db681b28628
│   ├── 8c
│   │   └── fe566596683acae588039156f40ecaff282c30
│   ├── ae
│   │   └── 161568392ed9aa321466446a9bb01acb111e4f
│   ├── b3
│   │   └── 02dd6f8cd2b385b170e78c14503342c0ba6ae8
│   ├── f9
│   │   └── 60e7d48c47e86289a653b0afc0b7a13a9d372e
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   └── master
    └── tags
```

Bây giờ, để xác định trạng thái hiện tại (xem code chúng ta đang ở bản nào), đầu tiên chúng ta sẽ tìm trong `.rgit/HEAD`. Nội dung trong file `HEAD` này sẽ trỏ tới `.rgit/refs/heads/master` -> master branch. Sau đó, mở file `.rgit/refs/head/master` ra, chúng ta sẽ thấy được commit gần nhất trên branch này. Commit này sẽ trỏ tới 1 tree object, tree object này sẽ trỏ tới 1 tree object khác đại diện cho thư mục `/bin/` mà chúng ta vừa commit bên trên, tree object của `bin` này sẽ trỏ tới 2 blob objects chứa nội dung đã nén của `bin/rgit` và `bin/rgit-add` tại thời điểm commit.

![](https://images.thoughtbot.com/rebuilding-git-in-ruby/mTnEMJCNS2wM3a9JtbCw_rgit-commit-tree.png)

Cấu trúc mà các object có thể trỏ qua lại lẫn nhau là 1 trong những thế mạnh của Git. Bằng cách thay đổi file cần trỏ đến, chúng ta sẽ có thể quay lại 1 thời điểm commit trong quá trình code.

## Kết luận

Túm lại:
1. Bài viết chỉ mang tính chất làm màu =]] vì hiếm người dở hơi tự nhiên build lại `git` - 1 sản phẩm đã được phát triển cả vài chục năm, tối ưu các kiểu ^^
2. Qua bài viết, chí ít chúng ta cũng sẽ hiểu được 1 số việc `under the hood` xảy ra khi chúng ta gõ những lệnh như `git init`, `git add`, `git commit` và làm thế nào để git có thể đưa code về 1 thời điểm commit trong quá khứ.
3. Nếu bạn có hứng thú, hãy build thêm những command khác cho `rgit` và là contributor cho repo [Rgit](https://github.com/JoelQ/rgit) nhé. ^^

Bài viết được tham khảo từ: https://robots.thoughtbot.com/rebuilding-git-in-ruby