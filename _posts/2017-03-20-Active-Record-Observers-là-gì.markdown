---
layout: post
title:  "Active Record Observers là gì"
date:   2017-03-20 00:00:00
---

Bonus Features của **Active Record**.  

 **Observers** là gì và khi nào dùng?

--- 
**Đặt vấn đề**:

Implement chức năng send email trong model Project tới user khi một project đc tạo ra.

```
class Project < ActiveRecord::Base
  after_save :send_email
  
  private
  
  def send_email
    # implement send email logic here
  end

end

```

Đợi đã, 1 cái `callback` thì ko sao nhưng với 10+ cái `callback` đc nhồi nhét vô trong model thì trông thật khó nhìn với hầm bà lằng cùng các `scope`, `validate`, `attributes`,... Cả một vấn đề đấy

Phương án: Tách thành `module` nhỏ nhỏ rồi `include` các kiểu vào trong model chính.

Câu hỏi đặt ra: Có cách nào hỗ trợ bởi Rails ko?

---

**ActiveRecord::Observers là gì?**

- Là một feature của ActiveRecord
- Là một cách để include các functionality không phải là **core concept** vào model
- Là phương pháp để giữ cho `the actual data model` không bị  *cluttering up or obfuscate* bởi các functionality được thêm vào.
- Được sử dụng với những feature sau: transaction logging, send notification, alerts, send email (background job)


**Example:**  Refactor đoạn code ở trên

```
# app/models/project.rb
class Project < ActiveRecord::Base
 # nothing here
end


# app/observers/email_observer.rb
class EmailObserver < ActiveRecord::Observers
  observer Project, Issue # Note 1
  
  def send_email
    # logic here
  end
end

# config/environment.rb
config.observers = [:email] # Note 2 

```


Chú thích :

 **Note1:** Observers sẽ tự động map với class cùng tên với nó (ví dụ: `EmailObserver` sẽ map với model `Email` )
Ở đây ta dùng cho model project và cả model issue (multiple model)
nên phải dùng cách khai báo như sau:

```
  observer Project, Issue, # something else
```  
**Note2:**  Đây là cách configure một observer 

-----
Tuy nhiên lạm dụng nó cũng chả tốt lắm đâu.
Đôi khi cứ như cách thông thường là đủ rồi.
