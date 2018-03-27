---
layout: post
title:  "Cách tích hợp sb-admin-2 vào dự án Rails"
date:   2018-03-27 00:00:00
---
TL,DR: Không nên sử dụng theme sb-admin boostrat bằng cách include các thư viện thông thường mà nên sử dụng bower (etc) để 

##### 1.Setup rails app
- Sử dụng gem bower_rails
Link: https://github.com/rharriso/bower-rails

- Init config
```
  rails g bower_rails:initialize
```

##### 2.Apply sb-admin

- Config bower
 Xem các thư viện mà sb-admin cần trong file bower.json sau đó chỉnh sửa file `Bowerfile` tương ứng.
*Example:*
```
  # Bowerfile
    asset "jquery"
    asset 'bootstrap', '~3.3.7'
    asset 'datatables', '~1.10.4'
    asset 'datatables-plugins', '~1.0.1'
    asset 'flot', '~0.8.3'
    asset 'font-awesome', '~4.6.3'
    asset 'holderjs', '~2.4.1'
    asset 'metisMenu', '~1.1.3'
    asset 'morrisjs', '~0.5.1'
    asset 'datatables-responsive', '1.0.6'
    asset 'bootstrap-social', '~4.8.0'
    asset 'flot.tooltip', '~0.8.4'

    resolution "font-awesome", "~4.6.3" <--- Chỉ định phiên bản của font-awesome
```

- install bower với quyền `root  `

```
  sudo npm install -g bower
  rake bower:install
```

- Config assets cho rails
 + css
     * copy file `sb-admin-2.css` vào thư mục `assets/admin/sb-admin-2.css`
     * Tạo file `admin.css`

        ```
        /*
        *= require bootstrap
        *= require metisMenu
        *= require morrisjs
        *= require_tree ./admin/
        *= require font-awesome
        */
        ```

 + js
     * copy file sb-admin-2.js vào thư mục assets/admin/sb-admin-2.js
     * Tạo file admin.js

        ```
        /*
        //= require turbolinks
        //= require jquery
        //= require rails-ujs
        //= require bootstrap/dist/js/bootstrap
        //= require metisMenu
        //= require_tree ./admin/
        */
        ```

+ Include new path `js/css` vào `assets`

  ```
   Rails.application.config.assets.precompile += %w( admin.css admin.js )
  ```

##### 3.Chạy precomplie

 Khi chạy precompie sẽ có những lỗi phát sinh và cách sửa như sau:

**a. Không nhận được bootstrap**

    Sprockets::FileNotFound: couldn't find file 'bootstrap' with type 'text/css'

Fix:
 - Thêm vào gem file

    ```
    gem 'therubyracer'
    gem 'less-rails'
    ```

 - Đổi `admin.css` -> `admin.css.sass`

**b.Font-awesome ko sử dụng được**  
 **Nguyên nhân**: Không precomplie được những file với extension khác

Fix:
 - Include thêm vào file config:  config/initializers/assets.rb

 ```
 Rails.application.config.assets.precompile << /\.(?:svg|eot|woff|woff2|ttf)\z/
```

 - import font-face font-awesome

  ```
  # app/assets/stylesheets/admin.css.scss
  @font-face {
      font-family: 'FontAwesome';
      src: font-url('font-awesome/fonts/fontawesome-webfont.eot?v=4.0.3');
      src: font-url('font-awesome/fonts/fontawesome-webfont.eot?#iefix&v=4.0.3') format('embedded-opentype'),
      font-url('font-awesome/fonts/fontawesome-webfont.woff?v=4.0.3') format('woff'),
      font-url('font-awesome/fonts/fontawesome-webfont.ttf?v=4.0.3') format('truetype'),
      font-url('font-awesome/fonts/fontawesome-webfont.svg?v=4.0.3#fontawesomeregular') format('svg');
      font-weight: normal;
      font-style: normal;
  }
  ```

##### 4.Deploy với cap  
Thêm config vào deploy.rb

```
    #deploy.rb
    namespace :bower do
      desc 'Install bower'
         task :install do
           on roles(:web) do
              within release_path do
              with rails_env: fetch(:rails_env) do
                 execute :rake, "bower:install['--allow-root'] CI=true"
              end
            end
           end
         end
       end
    before 'deploy:compile_assets', 'bower:install'
```
option : `--allow-root` <-- cấp quyền install bower lib vào system

**Lưu ý: Phải cài package bower trên sever **
