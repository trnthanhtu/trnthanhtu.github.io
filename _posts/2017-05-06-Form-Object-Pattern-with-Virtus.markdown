---
layout: post
title:  "Form Object Pattern with gem Virtus"
date:   2017-05-06 00:00:00
---

`Form Object Pattern` được sử dụng lúc nào?

  - Khi cần phải  thực hiện việc create/update multiple ActiveRecord models trên cùng một form
  - Khi việc sử dụng `accepts_nested_attributes_for` trở nên phức tạp hoặc không muốn sử dụng
  - Tuân thủ nguyên tắc **"single responsibility principle"**
  - Thin Model (việc chứa quá nhiều logic trong một model để phục vụ việc tương tác với form là không cần thiết gây nên sự phức tạp)

 `Form Object Pattern` tương tự như `Service Object`. Nó là một lớp "layer" đặt giữa view và controller ( cũng như model) để thực hiện nhiệm vụ từ cả hai. Ví dụ:
   - validate data
   - CRUD
   - notification
   - create nested attributes

  Do đó việc sử dụng Form Object sẽ giữ cho C-M đơn giản.

Trong bài viết này sẽ giới thiệu gem Virtus (Link: https://github.com/solnic/virtus)
Vì gem sẽ hỗ trợ ta thực hiện form object pattern đơn giản và nhanh hơn (bản chất thì vẫn như vậy)

1. Bài toán 
Có 2 model `User`  1-1 `Grade`. Trong đó Grade chứa thông tin `grade`(cấp bậc) `name`(tên lớp) của user.
Hãy tạo một chức năng register cho user bao gồm cả việc input thông tin grade.

2. Sử dụng Form Object

Add gemfile

```
#Gemfile
gem virtus
```

- Tạo mới 1 class extend Virtus để định nghĩa các thuộc tính và validate

+ Đầu tiên là khởi tạo
```
#app/forms/user_register_form.rb
class UserRegisterForm
  include ActiveModel::Model
  include Virtus.model
```

- Khai báo các attribute và type cần thiết.
Ở đây ta khai báo thêm các attribute "ảo " như `confirm` để thực hiệc việc kiểm tra confirm checkbox

```
  attribute :firstname, String
  attribute :lastname, String
  attribute :grade, Integer
  attribute :name, String
  attribute :confirm, String
```

- Thực hiện add các validates

```
  validates :firstname, presence: true
  validates :lastname, presence: true
  validates :name, presence: true
  validates :grade, presence: true,
                    inclusion: {in: 1..12}
  validates :confirm, acceptance: true
```

- Định nghĩa method `save` tương tự như method save mặc định của ActiveRecord, để thực hiện những action mà chúng ta cần thực hiện 

```
  def save
    return false unless valid?
    create_register
    true
  end

  private
    def create_register
      ActiveRecord::Base.transaction do 
        user = User.create!(firstname: firstname, lastname: lastname)
        user.build_grade(name: name, grade: grade).save!
      end
    end

```

Lưu ý: Luôn sử dụng `transaction` khi thực hiện việc thay đổi nhiều model.


- Xử lý controller

```
class RegistersController < ApplicationController
  def index
    @users = User.all
  end

  def new
    @user_register_form = UserRegisterForm.new
  end

  def create
    @user_register_form = UserRegisterForm.new(user_register_form_params)
    if @user_register_form.save
      redirect_to registers_path, notice: "created"
    else
      render :new
    end
  end

  private

  # Using strong parameters
  def user_register_form_params
    params.require(:user_register_form).permit(:firstname, :lastname, :name, :grade, :confirm)
  end
end

```

- Xử lý views

```
<%= form_for @user_register_form, url: registers_path do |f| %>
  <%= f.label :firstname %>
  <%= f.text_field :firstname %>

  <%= f.label :lastname %>
  <%= f.text_field :lastname %>

  <%= f.label :name %>
  <%= f.text_field :name %>

  <%= f.label :grade %>
  <%= f.text_field :grade %>

  <%= f.label :confirm %>
  <%= f.check_box :confirm %>

  <%= f.submit %>
<% end %>
```


Như vậy Form Object UserRegisterForm chịu trách nhiệm nhận các input đầu vào từ form register và sau đó thực hiện confirm, validate và lưu data ở các model tương ứng.
Nếu nhiều form, ta sẽ tạo nhiều Form Object tương ứng để xử lý mà không có đá động gì tới model gốc .





