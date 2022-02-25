+++
title = "railsをAPIサーバーとして使うときのOIDC認証"
date = 2022-02-23
[taxonomies]
tags = ["Ruby on Rails"]
[extra]
toc = true
+++

## やりたいこと
* railsでOIDC認証したい
* railsをAPIサーバーとして使ったときにもOIDC認証したい
  * 認証フローはcode grantにする
  * 認証情報はsessionに入れる

## OIDC認証する
[OIDC用のgem](https://github.com/omniauth/omniauth_openid_connect) を使う
READMEを参考に設定を行う。
* 認証フローをcode grantにするため`response_type`は`:code`とする。
* `callback_path`はOIDC認証後にredirectするURLで、たとえば`/auth/openid_connect`としておく([後述](#OIDC認証後のredirect))


## エンドポイントを生やす
### SPAからログイン済みかどうか確認するためのAPI
未ログインの場合は[ログイン用のpath](#OIDC認証するためのpath)に遷移する
```ruby
# config/routes.rb
get 'spa_logged_in' => 'sessions#spa_logged_in'

# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def spa_logged_in
    if session[:spa_login].present?
      render json: true
    else
      render json: false
    end
  end
end
```

### OIDC認証するためのpath
OIDC認証後に元いたページにredirectするためにreferrer情報をsessionに保存しておく
```ruby
# config/routes.rb
get 'spa_new' => 'sessions#spa_new'

# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def graphql_new
    session[:return_to] = request.referer
    '/auth/openid_connect'
  end
end
```

### OIDC認証後のredirect
OIDC認証後に認証済みであることをsessionに登録し、元いたページにredirectする
```ruby
# config/routes.rb
get 'spa_callback' => 'sessions#spa_callback'


# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def spa_callback
    session[:spa_login] = true
    return_to = session[:return_to]
    session.delete(:return_to) if session.key?(:return_to)
    redirect_to return_to
  end
end
```


## おまけ: SPAとAPIサーバーが別ドメインの場合
cross originでcookieを使ったsession管理はブラウザの制約が厳しかったり失敗したときにリスクが大きいので注意。[徳丸さんの発表資料]( https://www.slideshare.net/ockeghem/phpconf2021spasecurity)が参考になる

徳丸さんの資料では他のライブラリではCORSでcookieを共有できる`credentials: true`相当のを設定をしたときに、デフォルトだとすべてのoriginで有効になってしまう、とされている。
[rack-cors](https://github.com/cyu/rack-cors)のREADMEでは`If a wildcard (*) origin is specified, this option cannot be set to true`と書かれており、cookie共有するときは必ずoriginを設定しなければならないように読めるので、その心配はなさそう(未確認)。
```ruby
# Gemfile
gem 'rack-cors'

# config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'http://localhost:8080', %r{https://my-domain(\.test)?\.com}
    resource '*', headers: :any, methods: %i[get post patch put], credentials: true
  end
end

```

