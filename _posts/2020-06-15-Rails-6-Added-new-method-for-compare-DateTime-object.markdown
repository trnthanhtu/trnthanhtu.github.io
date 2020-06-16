---
layout: post
title:  "Rails 6: Use before? and after? instead of compare syntax"
date:   2019-06-15 00:00:00
---

  **TL,DR**:  With Rails 6, can be used two methods: before? and after? instead of  two systax" > "and "<" when compares Date, DateTime, Time object.  
   Rails 6で、オペレーターの代わりに「Date」や「DateTime」や「Time」オブジェクトなどを比較すろために使用する新しいメソッド「before?」と「after?」を追加された。


**1. before?**  
Rails 5:  
```
Time.now - 1.hours < Time.now
=> true

Date.yesterday < Date.today
=> true

```

Rails 6
```
(Time.now - 1.hours).before? (Time.now)
=> true

Date.yesterday.before?(Date.today)
=> true
```
**2. after?**  
Rails 5:  
```
(Time.now + 1.hours) > Time.now
=> true

Date.yesterday > Date.today
=> false

Date.tomorrow > Date.today 
=> true
```

Rails 6
```
(Time.now + 1.hours).after? Time.now
=> true

Date.yesterda.after? Date.today
=> false

Date.tomorrow.after? Date.today 
=> true
```

Reference (参照):　https://github.com/rails/rails/pull/32185 
