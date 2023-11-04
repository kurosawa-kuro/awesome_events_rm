# Railsアプリケーションの設定と拡張

## 1. Railsプロジェクトの開始
## 1.1 新規Railsアプリケーションの作成
```bash
rails new awesome_events01 --skip-action-mailer --skip-action-mailbox --skip-action-text --skip-action-cable
```

## 1.2 Gemファイルの編集
Gemの内容を設定します。
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

## 2. プラグインと依存ライブラリの設定
## 2.1 Babelプラグインの導入
```bash
yarn add --dev @babel/plugin-proposal-private-methods
yarn add --dev @babel/plugin-proposal-private-property-in-object
```

## 2.2 フロントエンドライブラリの追加
Bootstrap、jQuery、Popper.jsを導入します。
```bash
yarn add bootstrap@4.4.1 jquery@3.5.1 popper.js@1.16.1
```

## 3. 基本的なアプリケーション設定
## 3.1 アプリケーションの基本設定を定義
タイムゾーンやロケールを設定します。
```ruby
config.time_zone = "Tokyo"
config.i18n.default_locale = :ja
config.session_store :cookie_store, key: '_session'
config.middleware.use ActionDispatch::Cookies
config.middleware.use config.session_store, config.session_options
```

## 3.2 ルート設定の定義
アプリケーションのルートパスを設定します。
```ruby
Rails.application.routes.draw do
  root "welcome#index"
  get "/auth/:provider/callback" => "sessions#create"
  delete "/logout" => "sessions#destroy"

  resource :retirements, only: %i[new create]

  resources :events, only: %i[new create show edit update destroy] do
    resources :tickets, only: %i[new create destroy]
  end
  get 'status' => 'status#index', defaults: { format: 'json' }
end
```

## 3.3 OmniAuthによる認証設定
OmniAuthを使用した認証の初期設定を行います。
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

## 4. コードの変換とフレームワークの導入
## 4.1 ERBからHAMLへの変換
ERBファイルをHAMLフォーマットに変換します。
```bash
rails hamlit:erb2haml
```

## 4.2 Bootstrapフレームワークの導入
BootstrapをJavaScriptパックにインポートします。

追加先: `app/javascript/packs/application.js`
```javascript
import "bootstrap";
import "bootstrap/scss/bootstrap.scss";
```

## 5. コントローラとビューの準備
## 5.1 Welcomeコントローラとビューの作成
Welcomeページのコントローラとビューを生成します。
```bash
rails g controller welcome index
```

## 5.2 その他のコントローラとビュー
セッションやイベント、チケット、退会機能に関連するコントローラとビューを生成します。


## 6. モデルとデータベースの設定
## 6.1 Userモデルの作成とマイグレーション
Userモデルを作成し、データベースに反映します。

## 6.2 その他のモデルとマイグレーション
イベントやチケットに関連するモデルとマイグレーションを実行します。

## 7. アプリケーションの詳細設定
## 7.1 アプリケーションコントローラのカスタマイズ
セキュリティやヘルパーメソッドに関する設定を行います。
```ruby
class ApplicationController < ActionController::Base
    helper_method :logged_in?
    private
    def logged_in?
        !!session[:user_id]
    end
end
```

## 7.2 ファイルアップロード機能
Active Storageを使用したファイルアップロード機能の設定を行います。
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


ImageMagickのインストール

```
sudo apt-get update
sudo apt-get install imagemagick -y
convert --version
```

```
# config/environments/development.rb
config.active_storage.variant_processor = :mini_magick
```

## 7.3 シードデータの追加
開発用のサンプルデータをデータベースに追加します。
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

# ドキュメント清書版

## 8. 機能の拡張

### 8.1 Kaminariを利用したページネーションの設定

ページネーション機能を導入するために、以下のコマンドを実行してビューファイルを生成します。

```bash
bin/rails g kaminari:views bootstrap4
```

### 8.2 Elasticsearchのセットアップ

