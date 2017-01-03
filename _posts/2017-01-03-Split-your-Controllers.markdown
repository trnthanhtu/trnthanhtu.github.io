---
layout: post
title:  "RESTful - Split your controllers"
date:   2017-01-03 00:00:00
---

Cái này không hẵn là định viết nhưng mà thấy nên note lại để rút kinh nghiệm tổ chức project tốt hơn. Cũng tại lười

---

**Bài toán**:

Giả sử đã có một controller User với các action CRUD.
Hãy tạo thêm một action mới có chức năng là list các user với đk thích hợp để approve lên chức giám đốc chẳng hạn với path là
`/users/approves`.

---

**Giải quyết bài toán.**

Lúc trước lười nên vô controller `user` đã có để viết thêm action `approve`:

```
# file users_controller.rb

class UsersController < ApplicationController
   def index
   end
   # CRUD actions
   # action list admin
   def list_approve
   end
end

# file routes.rb
resources :user do
 get '/admin', to: user#admin'
end
```

Thật nhanh gọn lẹ. Nhưng đó chỉ là một cách đối phó của kẻ lười. 

Trường hợp nếu nhiều `action` tương tự được mở rộng ra, với nhiều logic trong đó thì cái controller ban đầu sẽ trở nên phức tạp một mớ hỗn độn.

Do đó **CẦN** **NÊN** và **PHẢI** chia tách và cấu trúc controller của bạn ngay từ lúc bắt đầu.

Từ đoạn code trên ta sẽ tạo mới một controller là `Users::AdminsController` với cấu trúc tổ chức như sau:

```
app/
| - controllers\
      |- user\
         | - approves_controller.rb
      |-users_controller.rb  

```

code:


```
users_controller.rb

class UsersController < ApplicationController
  # CRUD actions
end

user/approves_controller.rb

class User::AprrovesController < ApplicationController
  def index
  end

  def update
  end 

  ...
end

```
Tạo routes Sử dụng từ khoá `namespace`

```
namespace :user do
  resources :approves
end

```
Cuối cùng ta sẽ có một routes `RESTful` như sau:

```
GET       /user/approves
GET       /user/approves/new
POST      /user/approves
GET       /user/approves/1
GET       /user/approves/1/edit
PATCH/PUT /user/approves/1
DELETE    /user/approves/1

```
Mục đích ở đây là vẫn Keep được RESTful, và chia nhỏ controller ra.
Trong mỗi Controller chỉ chứa các CRUD đơn giản để việc phát triển và mở rộng sau này thật dễ dàng.

---


Bổ sung: Một số trường hợp sử dụng **scope** trong routes

- Nếu muốn tạo `ApprovesController` thay vì tiền tố `User::` nhưng vẫn giữ nguyên path `routes` thì

```
scope '/user' do
  resources :approves
end

```

- Nếu muốn tạo path là `/approves` thay vì `/user/approves` nhưng vẫn giữ nguyên `User::ApprovesController` thì

```
scope module: "user" do
  resources :approves
end

```

---