# Railsアプリケーションの設定と機能拡張ガイド

このドキュメントは、Railsアプリケーションの初期設定から機能拡張までを説明しています。

## 初期設定

### 1. プロジェクトの作成
新規Railsアプリケーションを作成します。
```bash
rails new awesome_events_rails7_01 --skip-action-mailer --skip-action-mailbox --skip-action-text --skip-action-cable --javascript=importmap
```

### 2. Gemファイルの編集
Gemファイルを編集して、必要なgemを設定します。
```ruby
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.1.2"

# Bundle edge Rails instead: gem "rails", github: "rails/rails", branch: "main"
gem "rails", "~> 7.0.8"

# The original asset pipeline for Rails [https://github.com/rails/sprockets-rails]
gem "sprockets-rails"

# Use sqlite3 as the database for Active Record
gem "sqlite3", "~> 1.4"

# Use the Puma web server [https://github.com/puma/puma]
gem "puma", "~> 5.0"

# Use JavaScript with ESM import maps [https://github.com/rails/importmap-rails]
gem "importmap-rails"

# Hotwire's SPA-like page accelerator [https://turbo.hotwired.dev]
gem "turbo-rails"

# Hotwire's modest JavaScript framework [https://stimulus.hotwired.dev]
gem "stimulus-rails"

# Build JSON APIs with ease [https://github.com/rails/jbuilder]
gem "jbuilder"

# Use Kredis to get higher-level data types in Redis [https://github.com/rails/kredis]
# gem "kredis"

# Use Active Model has_secure_password [https://guides.rubyonrails.org/active_model_basics.html#securepassword]
# gem "bcrypt", "~> 3.1.7"

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem "tzinfo-data", platforms: %i[ mingw mswin x64_mingw jruby ]

# Reduces boot times through caching; required in config/boot.rb
gem "bootsnap", require: false

# Use Sass to process CSS
# gem "sassc-rails"

# Use Active Storage variants [https://guides.rubyonrails.org/active_storage_overview.html#transforming-images]
gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'hamlit-rails', '~> 0.2.3'
gem 'omniauth', '~> 1.9.1'
gem 'omniauth-github', '~> 1.4.0'
gem 'omniauth-rails_csrf_protection', '~> 0.1.2'
gem 'rails-i18n', '~> 7.0.0'
gem 'active_storage_validations', '~> 0.8.8'
gem 'kaminari', '~> 1.2.0'
gem 'searchkick', '~> 4.3.0'
gem 'elasticsearch', '~> 7.0'
gem 'html2haml'

group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
  gem 'factory_bot_rails'
  gem 'rspec-rails', '~> 5.0'
end

group :development do
  # Use console on exceptions pages [https://github.com/rails/web-console]
  gem "web-console"

  # Add speed badges [https://github.com/MiniProfiler/rack-mini-profiler]
  # gem "rack-mini-profiler"

  # Speed up commands on slow machines / big apps [https://github.com/rails/spring]
  # gem "spring"
  gem 'rubocop', require: false
  gem 'rubocop-performance', require: false
  gem 'rubocop-rails', require: false
end

group :test do
  # Use system testing [https://guides.rubyonrails.org/testing.html#system-testing]
  gem "capybara"
  gem "selenium-webdriver"
  gem 'faker'
  gem 'rails-controller-testing'
end
```

### 3. プラグインと依存ライブラリ
Babelプラグインとフロントエンドライブラリを追加します。
```bash
yarn add --dev @babel/plugin-proposal-private-methods
yarn add --dev @babel/plugin-proposal-private-property-in-object
```

### 4. アプリケーションの基本設定
アプリケーションのタイムゾーンやルーティングなどを設定します。
```ruby
config.time_zone = "Tokyo"
config.i18n.default_locale = :ja
config.session_store :cookie_store, key: '_session'
config.middleware.use ActionDispatch::Cookies
config.middleware.use config.session_store, config.session_options
```

### 5. 認証設定
OmniAuthによる認証機能を設定します。
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

## コード変換とフレームワーク

### 1. ERBからHAMLへの変換
ERBファイルをHAMLに変換します。
```bash
rails hamlit:erb2haml
```

### 2. Bootstrapの導入
Bootstrapをインポートし、アプリケーションで使用できるようにします。
```javascript
import "bootstrap";
import "bootstrap/scss/bootstrap.scss";
```

## コントローラとビュー

### 1. Welcomeコントローラとビュー
Welcomeページ用のコントローラとビューを生成します。
```bash
rails g controller welcome index
```

### 2. その他のコントローラとビュー
セッション、イベントなどのコントローラとビューを生成します。
```bash
bin/rails g controller sessions
```

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

```bash
bin/rails g controller retirements
```

```bash
touch app/views/retirements/new.html.haml
```

## モデルとデータベース

### 1. Userモデル
Userモデルを作成し、データベースに反映します。
```bash
bin/rails g model user provider uid name image_url
```

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

### 2. その他のモデル
イベントやチケットのモデルを作成します。
```bash
# 省略（変更なし）
```

## アプリケーション詳細設定

### 1. アプリケーションコントローラ
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

### 2. ファイルアップロード機能
#### ファイルアップロード機能を設定します。
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

#### ImageMagickのインストール

```
sudo apt-get update
sudo apt-get install imagemagick -y
convert --version
```

```
# config/environments/development.rb
config.active_storage.variant_processor = :mini_magick
```

### 3. シードデータ
開発用のサンプルデータを追加します。
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

## 機能の拡張

### 1. ページネーションの設定
Kaminariを利用してページネーション機能を導入します。
```bash
bin/rails g kaminari:views bootstrap4
```

### 2. Elasticsearchのセットアップ
全文検索エンジンElasticsearchを導入します。
全文検索エンジンElasticsearchをセットアップする手順は以下の通りです。

システムのパッケージリストを更新します。

```bash
sudo apt update
sudo apt install default-jre
java -version
```

Elasticsearchの公式GPGキーを追加します。

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'
sudo apt update
sudo apt install elasticsearch
```

Elasticsearchサービスを起動し、システム起動時に自動で起動するように設定します。

```bash
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```

Elasticsearchが正しく動作しているか確認します。

```bash
curl -X GET "localhost:9200/"
```

日本語解析のためのプラグイン `kuromoji` をインストールします。

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-kuromoji
```

Elasticsearchサービスを再起動します。

```bash
sudo systemctl restart elasticsearch
```

RailsアプリケーションでElasticsearchのインデックスを作成するために、以下のコマンドを実行します。

```bash
bin/rails r Event.reindex
```
