---
layout: post
title:  "RSpecでTimecopをRailsのTime Helperに置き換える"
date:   2019-01-29 00:00:00
---
TL,DR:  gem Timecopの代わりにRailのTime Helperを使えます。「ActiveSupport::Testing::TimeHelpers」です。

「ActiveSupport::Testing::TimeHelpers」はRailsのActiveSupportのライブラリーです。Rails4.1で開放した。  

**Railプロジェクトで、Timecop代わりに**:
```
#Gemfile ファイル
group :test do
  gem "timecop"
end

# Test ファイル
describe "テストの説明文" do
  before do
    Timecop.freeze(Time.local(xxxx))
  end

  after do
    Timecop.return
  end
  
  ### テストの機能
end
```

**「TimeHelpers」を使える**

```
# spec/support/time_helpers.rb
RSpec.configure do |config|
  config.include ActiveSupport::Testing::TimeHelpers
end

describe "テストの説明文" do
  before do
    travel_to Time.local(xxxx)
  end

  after do
    travel_back
  end

  ### テストの機能
end
```

**注意**: Railプロジェクトだけで使える。
