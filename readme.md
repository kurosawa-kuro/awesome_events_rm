## Awesome Events アプリケーション セットアップ手順

### 1. 新しい Rails アプリケーションの作成
```bash
rails new awesome_events01 --skip-action-mailer --skip-action-mailbox --skip-action-text --skip-action-cable
```

### 1. 新しい Rails アプリケーションの作成
Gemをコピーする
```ruby
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.5'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 6.1.7.6'
# Use sqlite3 as the database for Active Record
gem 'sqlite3', '~> 1.4'
# Use Puma as the app server
gem 'puma', '~> 4.1'
# Use SCSS for stylesheets
gem 'sass-rails', '>= 6'
# Transpile app-like JavaScript. Read more: https://github.com/rails/webpacker
gem 'webpacker', '~> 4.0'
# Turbolinks makes navigating your web application faster. Read more: https://github.com/turbolinks/turbolinks
gem 'turbolinks', '~> 5'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem 'jbuilder', '~> 2.7'
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false
gem 'hamlit-rails', '~> 0.2.3'
gem 'omniauth', '~> 1.9.1'
gem 'omniauth-github', '~> 1.4.0'
gem 'omniauth-rails_csrf_protection', '~> 0.1.2'
gem 'rails-i18n', '~> 6.0.0'
gem 'active_storage_validations', '~> 0.8.8'
gem 'kaminari', '~> 1.2.0'
gem 'searchkick', '~> 4.3.0'
gem 'elasticsearch', '~> 7.0'
gem 'html2haml'

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  gem 'factory_bot_rails'
end

group :development do
  # Access an interactive console on exception pages or by calling 'console' anywhere in the code.
  gem 'web-console', '>= 3.3.0'
  gem 'listen', '~> 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'

  gem 'rubocop', require: false
  gem 'rubocop-performance', require: false
  gem 'rubocop-rails', require: false
  gem 'brakeman'
  gem 'rack-mini-profiler', require: false
end

group :test do
  # Adds support for Capybara system testing and selenium driver
  gem 'capybara', '>= 2.15'
  gem 'selenium-webdriver'
  # Easy installation and use of web drivers to run system tests with browsers
  gem 'webdrivers'
  gem 'simplecov', require: false
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

### 2. Babel プラグインの追加
```bash
yarn add --dev @babel/plugin-proposal-private-methods
yarn add --dev @babel/plugin-proposal-private-property-in-object
```

### 3. 初期設定の変更
追加先: `config/application.rb`
```ruby
config.time_zone = "Tokyo"
config.i18n.default_locale = :ja
config.session_store :cookie_store, key: '_session'
config.middleware.use ActionDispatch::Cookies
config.middleware.use config.session_store, config.session_options
```

### 4. ERB ファイルを HAML に変換
```bash
rails hamlit:erb2haml
```
コメントアウト:
```ruby
# gem 'html2haml'
```

### 5. Welcome コントローラと index アクションの作成
```bash
rails g controller welcome index
```

### 6. ルートパスの設定
追加先: `config/routes.rb`
```ruby
Rails.application.routes.draw do
  root 'welcome#index'
  resources :events
  get "/auth/:provider/callback" => "sessions#create"
  delete "/logout" => "sessions#destroy"
  resource :retirements, only: %i[new create]
  resources :events do
    resources :tickets
  end 
end
```

### 7. Bootstrap、jQuery、Popper.js の追加
```bash
yarn add bootstrap@4.4.1 jquery@3.5.1 popper.js@1.16.1
```

### 8. Bootstrap のインポート
追加先: `app/javascript/packs/application.js`
```javascript
import "bootstrap";
import "bootstrap/scss/bootstrap.scss";
```

### 9. アプリケーションのレイアウト
```haml
!!!
%html
  %head
    %meta{ content: "text/html; charset=UTF-8", "http-equiv": "Content-Type" }
    %title AwesomeEvents
    = csrf_meta_tags
    = csp_meta_tag
    = stylesheet_link_tag "application", media: "all", "data-turbolinks-track": "reload"
    = javascript_pack_tag "application", "data-turbolinks-track": "reload"
  %body
    %header.navbar.navbar-expand-sm.navbar-light.bg-light
      .container
        = link_to "AwesomeEvents", root_path, class: "navbar-brand mr-auto"
        %ul.navbar-nav
          %li.nav-item
            = link_to "イベントを作る", new_event_path, class: "nav-link"

            %li.nav-item
              = link_to "退会", new_retirements_path, class: "nav-link"
            %li.nav-item
              = link_to "ログアウト", logout_path, class: "nav-link", method: :delete

            %li.nav-item
              = link_to "GitHubでログイン", "/auth/github", class: "nav-link", method: :post
    .container
      - if flash.notice
        .alert.alert-success
          = flash.notice
      - if flash.alert
        .alert.alert-danger
          = flash.alert
      = yield
```

### 10. OmniAuth の設定
```bash
touch config/initializers/omniauth.rb
```
追加先: `config/initializers/omniauth.rb`
```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
    OmniAuth.config.allowed_request_methods = [:get, :post]
    if Rails.env.development? || Rails.env.test?
      provider :github, "2e122e66d4597660c5d4", "09bfba470d48627b9f983d91b85c845567fe583c"
    else
      provider :github, Rails.application.credentials.github[:client_id], Rails.application.credentials.github[:client_secret]
    end
end
```

### 11. User モデルの作成
```bash
bin/rails g model user provider uid name image_url
```
マイグレーション:
```ruby
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :provider,  null: false
      t.string :uid,       null: false
      t.string :name,      null: false
      t.string :image_url, null: false
      t.timestamps
    end

    add_index :users, %i[provider uid], unique: true
  end
end
```

### 12. セッション関連のコントローラ作成
```bash
bin/rails g controller sessions
```

### 13. イベント関連のリソース作成
```bash
bin/rails g resource event owner_id:bigint name place start_at:datetime end_at:datetime content:text
```

```ruby
class CreateEvents < ActiveRecord::Migration[6.0]
  def change
    create_table :events do |t|
      t.bigint :owner_id
      t.string :name,       null: false
      t.string :place,      null: false
      t.datetime :start_at, null: false
      t.datetime :end_at,   null: false
      t.text :content,      null: false
      t.timestamps
    end

    add_index :events, :owner_id
  end
end
```

```bash
touch app/views/events/new.html.haml
touch app/views/events/show.html.haml
touch app/views/events/edit.html.haml
```

```haml
%h1.mt-3.mb-3= @event.name
.row
  .col-8
    .card.mb-2
      %h5.card-header イベント内容
      .card-body
        %p.card-text= @event.content
    .card.mb-2
      %h5.card-header 開催時間
      .card-body
        %p.card-text= "#{l(@event.start_at, format: :long)} - #{l(@event.end_at, format: :long)}"
    .card.mb-2
      %h5.card-header 開催場所
      .card-body
        %p.card-text= @event.place
    .card.mb-2
      %h5.card-header 主催者
      .card-body
        - if @event.owner
          = link_to("https://github.com/#{@event.owner.name}", class: "card-link") do
            = image_tag @event.owner.image_url, width: 50, height: 50
            = "@#{@event.owner.name}"
        - else
          退会したユーザです
```

### 14. チケット関連のモデル・コントローラ作成
```bash
bin/rails g model ticket user:references event:references comment
bin/rails g controller tickets
```

```ruby
class CreateTickets < ActiveRecord::Migration[6.0]
  def change
    create_table :tickets do |t|
      t.references :user
      t.references :event, null: false, foreign_key: true, index: false
      t.string :comment
      t.timestamps
    end

    add_index :tickets, %i[event_id user_id], unique: true
  end
end
```

### 15. 退会機能のコントローラ作成
```bash
bin/rails g controller retirements
```

```bash
touch app/views/retirements/new.html.haml
```

### 16. アプリケーションコントローラの追加機能
```ruby
class ApplicationController < ActionController::Base
    helper_method :logged_in?
    private
    def logged_in?
        !!session[:user_id]
    end
end
```

### 17. ファイルアップロード
```bash
bin/rails active_storage:install
```

```haml
= image_tag @event.image, class: "card-img-top img-size-limit"

.img-size-limit {
  width: 200px;
  height: 200px;
  object-fit: cover;
}
```

### 18. シードデータの追加
追加先: `db/seeds.rb`
```ruby
# db/seeds.rb

# User seed data
user1 = User.create!(
  provider: "github",
  uid: "uid1",
  name: "User One",
  image_url: "https://example.com/user1.jpg"
)

user2 = User.create!(
  provider: "github",
  uid: "uid2",
  name: "User Two",
  image_url: "https://example.com/user2.jpg"
)

user3 = User.create!(
  provider: "github",
  uid: "uid3",
  name: "User Three",
  image_url: "https://example.com/user3.jpg"
)

# Event seed data
event1 = Event.create!(
  owner: user1,
  name: "Event One",
  place: "Place One",
  start_at: DateTime.now + 1.days,
  end_at: DateTime.now + 2.days,
  content: "Event One content here."
)

event2 = Event.create!(
  owner: user2,
  name: "Event Two",
  place: "Place Two",
  start_at: DateTime.now + 3.days,
  end_at: DateTime.now + 4.days,
  content: "Event Two content here."
)

event3 = Event.create!(
  owner: user3,
  name: "Event Three",
  place: "Place Three",
  start_at: DateTime.now + 5.days,
  end_at: DateTime.now + 6.days,
  content: "Event Three content here."
)

# Ticket seed data
Ticket.create!(
  user: user1,
  event: event2,
  comment: "Looking forward to it!"
)

Ticket.create!(
  user: user2,
  event: event3,
  comment: "Can't wait!"
)

Ticket.create!(
  user: user3,
  event: event1,
  comment: "See you there!"
)
```