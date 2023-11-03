## awesome\_events アプリケーションのセットアップ手順

### 1. 新しい Rails アプリケーションの作成

```bash
rails new awesome_events01 --skip-action-mailer --skip-action-mailbox --skip-action-text --skip-action-cable
```

### 2. Babel プラグインの追加

```bash
yarn add --dev @babel/plugin-proposal-private-methods
yarn add --dev @babel/plugin-proposal-private-property-in-object
```

### 3. 初期設定の変更

`config/application.rb` に追加:

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

```
# gem 'html2haml'
```

### 5. Welcome コントローラと index アクションの作成

```bash
rails g controller welcome index
```

### 6. ルートパスの設定

`config/routes.rb` に追加:

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

`app/javascript/packs/application.js` に追加:

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

`config/initializers/omniauth.rb` に追加:

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

```
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

```bash
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

### 14. チケット関連のモデル・コントローラ作成

```bash
bin/rails g model ticket user:references event:references comment
bin/rails g controller tickets
```

```bash
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

### 17. その他

ApplicationController:

```ruby
class ApplicationController < ActionController::Base
    helper_method :logged_in?

    private

    def logged_in?
        !!session[:user_id]
    end
end
```

新しいビューファイルの作成:

```bash
touch app/views/events/new.html.haml
touch app/views/events/show.html.haml
```

`app/views/events/show.html.haml` の内容:

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

### ファイルアップロード

```
bin/rails active_storage:install
```

```
gem 'image_processing', '~> 1.2'
```

```
gem 'active_storage_validations', '~> 0.8.8'
```

```
sudo apt-get install -y software-properties-common
sudo add-apt-repository ppa:alex-p/vips

sudo apt-get update
sudo apt-get install -y libvips
```

```
= image_tag @event.image, class: "card-img-top img-size-limit"

.img-size-limit {
  width: 200px;
  height: 200px;
  object-fit: cover;
}
```

### seed



```
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

