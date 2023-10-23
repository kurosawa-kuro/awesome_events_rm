# awesome\_events アプリケーションのセットアップ手順

## 1. 新しい Rails アプリケーションの作成

```bash
rails new awesome_events01 --skip-action-mailer --skip-action-mailbox --skip-action-text --skip-action-cable
```

## 2. Babel プラグインの追加

```bash
yarn add --dev @babel/plugin-proposal-private-methods
yarn add --dev @babel/plugin-proposal-private-property-in-object
```

## 3. 初期設定の変更

`config/application.rb`に以下を追加します。

```ruby
config.time_zone = "Tokyo"
config.i18n.default_locale = :ja

config.session_store :cookie_store, key: '_session'
config.middleware.use ActionDispatch::Cookies
config.middleware.use config.session_store, config.session_options
```

## 5. ERB ファイルを HAML に変換

```bash
rails hamlit:erb2haml
```

コメントアウト
```
# gem 'html2haml'
```

## 6. Welcome コントローラと index アクションの作成

```bash
rails g controller welcome index
```

## 7. ルートパスの設定

ルーティング設定に以下を追加します。

```ruby
root 'welcome#index'
```

## 8. Bootstrap、jQuery、Popper.js の追加

```bash
yarn add bootstrap@4.4.1 jquery@3.5.1 popper.js@1.16.1
```

## 9. Bootstrap のインポート

```javascript
# rails_training/awesome_events/awesome_events03/app/javascript/packs/application.js

import "bootstrap";
import "bootstrap/scss/bootstrap.scss";
```

## 10. アプリケーションのレイアウト

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
  %header.navbar.navbar-expand-sm.navbar-light
    .container
      = link_to "AwesomeEvents", root_path, class: "navbar-brand"
      %ul.navbar-nav
        %li.nav-item
          = link_to "Gihubでログイン", "/auth/github", class: "nav-link", method: :post
  .container
    = yield
```

## 11. OmniAuth の設定

```
touch config/initializers/omniauth.rb
```

`config/initializers/omniauth.rb`に以下を追加します。

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

## 12. User モデルの作成

```bash
bin/rails g model user provider uid name image_url
```

## 13. セッション関連のコントローラ作成

```bash
bin/rails g controller sessions
```

## 14. イベント関連のリソース作成

```bash
bin/rails g resource event owner_id:bigint name place start_at:datetime end_at:datetime content:text
```

## 15. チケット関連のモデル・コントローラ作成

```bash
bin/rails g model ticket user:references event:references comment
bin/rails g controller tickets
```

## 16. 退会機能のコントローラ作成

```bash
bin/rails g controller retirements
```

## 17. ルーティングの設定

`config/routes.rb`に以下を追加します。

```ruby
Rails.application.routes.draw do
  root "welcome#index"
  get "/auth/:provider/callback" => "sessions#create"
  delete "/logout" => "sessions#destroy", as: :logout

  resource :retirements

  resources :events do
    resources :tickets
  end
end
```