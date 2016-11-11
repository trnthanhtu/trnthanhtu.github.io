---
layout: post
title:  "Freeze trong ruby"
date:   2016-11-10 10:40:00
---
Bài viết đầu tiên về ma thuật của nàng Elsa.  

## 1. Tổng quan. 
   Tôi có một object. Tôi muốn đóng băng nó

   Object.freeze 

   Bùm

   Object.frozen? #=> true

Vậy đấy. Dùng method "freeze" để đóng băng object nào đó.
Và dùng method "frozen?" để kiểm tra xem object đã đóng băng hay chưa.

## 2. Các vấn đề cần nắm rõ
Để hiểu vấn đề thì phải đào sâu deep.  
### a. Các kiểu dữ liệu được mặc định frozen.  

Trong Ruby có các kiểu dữ liệu mặc định là  
   **- Boolean  
     - Fixnum. Bignum, Float  
     -  Symbol**

Tất cả kiểu dữ liệu trên đều được freeze.  
Bạn có thể xem ví dụ sau
  
```
irb(main):001:0> 1.frozen? 
=> true  
irb(main):002:0> true.frozen?  
=> true  
irb(main):003:0> :ruby.frozen?  
=> true  
```
(Thử tưởng tượng nó ko được freeze thì thật điên loạn. 1 có thể gán bằng 2 hay true có thể là false. Oh that's crazytown) 

### b. Unfrozen trong vô vọng.

Nói một cách đơn giản là bạn chỉ có thể đông cứng nhưng ko thể làm tan chảy 1 thực thể (object) được freezed.

Tất nhiên khi bạn cố gắng thay đổi nó thì sẽ nhận được một msg `can't not modify Hash/Array/String/Fixnum/Bignum`...

Vậy làm thế nào để tôi có thể tương tác với giá trị object đó đây.?
Well cách đơn giản nhất là hãy làm việc với bản sao của nó bằng cách sử dụng những method có tính chất **"immutable"**.  
Hãy xem ví dụ sau  

```
 a = "String uses to test".freeze
 a.upcase!
 => RuntimeError: can't modify frozen String
 
 a.upcase # => 'A'
```
Khi ta sử dụng `upcase!` có nghĩa là nó tác động upcase chính đối tượng frozen được reference tới nên văng lỗi.

Ngược lại với `upcase`: nó sẽ trả về một đối tượng mới copy từ đối tượng cũ và được upcase.

(*Mặc khác để xác định một method là immutable hay ko thì có thể dựa vào việc check object_id*) 

### c. Freeze chỉ có tác động lên object chứ không phải variable .
Ví dụ  

```
 a = "String to test".freeze
 puts a #=> "String to test"
 
 a = "Assign new string"
 
 puts a #=> "Assign new string"

```
### d. Không dùng cho  classes được kế thừa từ BasicObject.
```
class BasicFoo < BasicObject; end
bf = BasicFoo.new
bf.freeze #=> NoMethodError: undefined method `freeze' for BasicFoo
```
Điều này xảy ra là vì: Method **freeze** chỉ được *định nghĩa* ở class Object.

### e. Không có deep freeze 
Khi bạn freeze một đối tượng thì có nghĩa là đối tượng đó không được **Modify**.

Với object có cấu trúc là  **Array** hay **Hash** thì không cho phép **Modify** chính nó nhưng vẫn có thể **modify** phần tử chứa trong nó.
(Thật quái phải không?).  
Bạn có thể kiểm tra bằng một ví dụ sau:  
Trường hợp modify Hash:

```
hash = {'key1' => 'value1'}.freeze
hash.merge({'key2' => 'value2'})
=> {"key1"=>"value1", "key2"=>"value2"}
hash
=> {"key1"=>"value1"}
```
Trường hợp modify element trong Hash:

```
hash = {'key1' => 'value1'}.freeze
hash['key1'] << "add new value"
=> "value1add new value"
hash
=> {"key1"=>"value1add new value"}
```

**Điều này cần phải lưu ý**: đó là không có deep_freeze hay đệ qui freeze đối với một đối tượng có cấu trúc array hay hash.

Do đó khi define một *CONSTANT* có dạng [] hay {} trong ruby bạn phải thực hiện **deep_freeze cho các phần tử item** được chứa trong nó như sau:

```
CONSTANT = ["a", "b", "c"].map(&:freeze)
=> ["a", "b", "c"]
CONSTANT[0] << 'd'
RuntimeError: can't modify frozen String

``` 
### f. Hash[key].frozen? #=> true

**Key** của các phần tử trong **Hash** được mặc định là freeze. Điều này có thể lý giải đơn giản hoá như sau:  

   Bởi vì key của hash có thể là **String**, **Symbol** hoặc một **Number**(nói chung). Trong khi đó **Symbol** và **Number** là **Immutable**. Do đó nếu key là môt **String** cũng được freeze cho đồng bộ với cách xử lý: Dựa vào object_id của key để xác định value có trong Hash.

---   
*Trên đây là một số hiểu biết của mình về freeze và các vấn đề liên quan. Mình nghĩ nó cũng chưa thật sự chính xác lắm nhưng cũng lý giải phần nào một số vấn đề có thể xảy ra.*
