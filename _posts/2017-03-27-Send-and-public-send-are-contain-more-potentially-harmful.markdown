---
layout: post
title:  "Be careful with Send & public_send method"
date:   2017-03-27 00:00:00
---

Những thứ tiềm ẩn nguy hiểm tới chương trình của bạn khi dùng `send` và `public_send`.

`send` : là gọi method của object
`public_send`: chỉ `send` những public method

--

### 1.Hãy xem xét nhữg ví dụ sau:

**ví dụ 1:**

Trong Kernel's module có method là `exit`. và mọi Object đều có thể gọi `Object.exit`.

Chuyện gì xảy ra nếu tau chạy lệnh ở dưới trong irb

```
  1.send(:exit)  

```
=> Stop IRB!

**ví dụ 2:**  

```
class Foo < ActiveRecord::Base
 scope :newest -> { order(id: desc)}
 scope :oldest -> { order(id: asc)}
end

class IndexController < ApplicationController
  def index
    @foos = Foo.send(params[:sort])
  end
end
 
```

Chuyện gì xảy ra nếu ta truyền lên params[:sort] = 'destroy_all'

Lúc đó sẽ thành 

```
@foos = Foo.send(:destroy_all)

#=> DELETE * FROM Foo

```

**ví dụ 3:**

Từ ví dụ 1, thay bởi public_send kết hợp với `instance_eval` (`instance_eval` là một public method của Kernel module)


```
# irb
1.public_send(:instance_eval, 'exit')

```
=> Stop IRB



### 2. Làm thế nào để phòng tránh

- Nên sử dụng `public_send` thay vì sử dụng `send` bởi vì `public_send` chỉ call những public method

- **Đừng tin những gì mà người dùng input vào**. Chỉ sử dụng `public_send` và `send` trong trường hợp **hardly code** hay bạn thật sự controll và **tin tưởng** những tên của method.

- Sử dụng whitelist (kiểm tra các giá trị params được phép truyền vào)

Từ ví dụ 2 ta có thể xử lý như sau

```
class Foo < ActiveRecord::Base
 scope :newest -> { order(id: desc)}
 scope :oldest -> { order(id: asc)}
end

class IndexController < ApplicationController
  def index
    @foos = Foo.send(permit_sort])
  end

  private

    def permit_sort
       params[:sort].presence_in(%w(newest oldest)) || :newest
    end
end

```

`presence_in`: Trả về giá trị (recevier) nếu chúng được `include` trong list đối số hay là `nil`.
(Ở đoạn code trên ta thêm xử lý so sánh nếu nil thì trả về giá trị `newest`)

