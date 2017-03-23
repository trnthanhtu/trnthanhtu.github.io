---
layout: post
title:  "Distinct Include&Extend Require&L"
date:   2017-03-23 00:00:00
---
**Include** & **Extend**  
- Khi muốn cơ chế mix-in trong ruby thì bạn sử dụng Include và Extend.
- Dùng để DIY code.
- Khi có nhiều class muốn sử dụng chung một module.
- Khi bạn viết cả class và module đó trong 1 file.

*Example:* 
```
module A
  def A'
     puts ‘Call method A’ from Module A'
  end
end

class B
   include A
end

A.new.A'
#=> ‘Call method A’ from Module A '
```

=> Thế làm thế nào để phân biệt:

Khác nhau là cách bạn muốn gọi method đó như thế nào trong class đó:
**instance method** hay **class method**
- `Include`: Gọi như một instance method: 
```
Object.new.method
```
- `Extend`: Gọi như một class method 
```
Object.method
```

**Vấn đề phát sinh**  

Nhiều class, module quá muốn tách ra từng file (separate) cho dễ quản lý. Rồi làm thế nào để gọi nó được đây?  
=> Sinh ra **require** và **load**.

Sự khác nhau là gì? 

`require` : load 1 lần duy nhất.

`load` : load mỗi lần bạn gọi.
