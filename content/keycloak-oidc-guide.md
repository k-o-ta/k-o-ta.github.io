+++
title = "keycloakでOIDC認証するときに知っておくと良いこと"
date = 2020-11-04
[taxonomies]
tags = ["OIDC", "keycloak"]

[extra]
toc = true
+++

keycloakを使ってOIDC認証を導入することになり、設定画面で何を設定すべきかなどに困ったのでまとめておく。  
前提
* OAuth2の基本的な理解
* OIDCの基本的な理解()
  * OIDC自体を知りたい場合は[この説明](https://tech-lab.sios.jp/archives/8651)とかがわかりやすいと思う
* keycloakのOIDCクライアントは作成済み([手順](https://keycloak-documentation.openstandia.jp/master/ja_JP/server_admin/index.html#oidc%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88))

## 極めて単純なkeycloakの要素
keycloakの設定方法が分かりづらいのは管理画面に設定する項目が大量にあるからだが、極めて単純化すると最初に知るべきなのはrealm, client, userくらいである。

### realm
[用語集](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%B3%E3%82%A2%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E7%94%A8%E8%AA%9E)の記述を抜粋すると
> レルムは、ユーザー、クレデンシャル、ロール、および、グループのセットを管理します。ユーザーは属しているレルムにログインします。レルムは互いに分離されており、制御するユーザーのみを管理して、認証することができます。

となっている。言ってみればnamespace的なもので、複数アプリケーションでシングルサインオン(SSO)するときは同一のrealmにユーザーやアプリケーションを登録する

### client
[用語集](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%B3%E3%82%A2%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E7%94%A8%E8%AA%9E)の記述を抜粋すると
> クライアントは、Keycloakにユーザーの認証を要求できるエンティティーです。多くの場合、クライアントはKeycloakを使用して自分自身を保護し、シングル・サインオン・ソリューションを提供するアプリケーションとサービスのことを指します。

となっている。keyclaokを利用するアプリケーションのことで、複数アプリケーション間でSSOを実現するときはアプリケーション分clientをrealmに登録する。

### uesr
[用語集](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%B3%E3%82%A2%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E7%94%A8%E8%AA%9E)の記述を抜粋すると
> システムにログイン可能なエンティティーのことです。電子メール、ユーザー名、住所、電話番号、生年月日など自分自身に関連する属性を持ちます。また、グループ・メンバーシップが割り当てられたり、特定のロールが割り当てられたりします。

となっている。認証されるユーザーのこと。

### 関連図
単純に考えると、keycloakの役割は以下である。
* アプリケーション(`client`)に変わって`user`の認証を行う
* `client`や認証した`user`の情報をOIDC tokenに変換してアプリケーションに提供する

あるrealmでの要素間の関係は以下のようになる。OIDC `token`の生成元となる要素と、生成されるOIDC要素を色で分けている(OIDCはkeycloakで使えるプロトコルのひとつなので`<<protocol>>`としている。OIDC以外のプロトコル(SAMLやOAuth2)を利用するときはtokenが変わるだろう)  
keycloakの要素に記載した `> Clients` というのは実際にkeycloakの設定画面からアクセスしたときのパンくずを表している。

![simple_keycloak](/content/keycloak-oidc-guide/keycloak_simple.svg)

## 単純でないkeycloakの要素
上の項で説明したようにkeycloakは`realm`,`client`,`user`の設定をOIDCのtokenに変換するわけだが、`client`や`user`のどの情報をOIDC tokenに反映するかの設定をしたり、OIDCの仕様にない情報をtokenに埋め込めるようになっている。これらの設定がkeycloakの管理画面を難しくしている要因だろう。  
なお、OIDCを通して得られる情報として[ID token](https://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#IDToken),や[Access token](https://openid-foundation-japan.github.io/rfc6749.ja.html#anchor8),認証されたユーザーに関する情報を取得するための[UserInfoエンドポイント](https://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#UserInfo)がある。ただし、Access tokenはOAuth2のもので、実装方法も認可サーバーに依存している。

### role
[用語集](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%B3%E3%82%A2%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E7%94%A8%E8%AA%9E)の記述を抜粋すると
> ロールは、ユーザーのタイプまたはカテゴリーを識別します。 Admin、user、 manager、employee はすべて、組織内に存在する典型的なロールです。アプリケーションは、ユーザーの扱いをきめ細く管理することが難しくなる場合があるため、個々のユーザーではなく、特定のロールにパーミッションを割り当てることが多いです。

まずは`role`について。これはあまり難しくないが`user`に属性を付けるのに使う。OIDCがデフォルトで持つユーザー情報には`role`は存在しないので、組織情報などを付与できると便利。注意が必要なのは`role`はkeycloak内部で権限情報として利用されることもあるということ。(特にdefaultで用意されている`role`)にはそれらの権限が付与されており、keycloakのRest apiを使うときなどに`role`の有無によって実行できる操作が変わってくる。  
また、`realm`単位と`client`単位の`role`が存在する。

#### realm role
`realm`全体で使えるrole([ドキュメント](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%83%AC%E3%83%AB%E3%83%A0%E3%83%AD%E3%83%BC%E3%83%AB))

#### client role
`client`単位で使えるrole([ドキュメント](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%AB))  
設定箇所が`client`からなことに注意

### user
前述の通りだが、`user`と`role`を結びつけるための設定が必要

#### user role mapping
[用語集](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%B3%E3%82%A2%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E7%94%A8%E8%AA%9E)の記述を抜粋すると
> ユーザー・ロール・マッピングは、ロールとユーザーの間のマッピングを定義します。ユーザーには、0以上のロールを関連付けることができます。このロールマッピング情報を、トークンとアサーションにカプセル化して、アプリケーション(client)が管理するさまざまなリソースに対するアクセス許可を決定できるようにします。

`user`と`role`を結びつける設定。注意が必要なのが、デフォルトだと`user`に結びつけた`role`情報はすべて`token`内に表示されることである。`role`の説明で記載したが、`role`はkeycloak内部で権限情報として利用されることもあるので不正に`token`を取得されたときに悪用される可能性がある。それを防ぐための仕組みが`client`に対して設定できる`role scope mapping`である


### client
前述の通りだが、keycloak内のどの情報を`token`に含めるかの設定を行うmappingの設定を`role scope mapping`と`protocol mapper`で行っている。

#### role scope mapping
`user`に対して設定した`role`を`token`に含めるかどうかの設定ができる([ドキュメント](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#_role_scope_mappings))

#### protocol mapper
[用語集](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%B3%E3%82%A2%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E7%94%A8%E8%AA%9E)の記述を抜粋すると
> 各クライアントに対して、どのクレームおよびアサーションをOIDCトークンまたはSAMLアサーションに格納するかを調整できます。クライアントごとに、プロトコル・マッパーを構成します。

クレームとはOIDCの`token`を構成する要素(`token`はクレームの集合)  
`role scope mapping`は`role`情報を`token`に含めるかどうかを設定しているが、それ以外の`token`に乗るべきすべての情報は`protocol mapper`で設定されている。keycloak内部でprotocolといったときはOIDC,OAtuh2,SAMLといった認証/認可プロトコルのことを意味していると思ってよい。

### client scope
前述の通り`token`にどのような項目を乗せるかはclientが設定しているが、clientが複数ある場合何度も同じ設定を繰り返すのは面倒(OIDCの`token`などは仕様でも決まっているのでわざわざ自分で設定したくない)  
そのため複数clientで使い回せるmappingの設定をする仕組みが`client scope`である。
[用語集](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%B3%E3%82%A2%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E7%94%A8%E8%AA%9E)の記述を抜粋すると
> クライアントが登録されたら、そのクライアントのプロトコル・マッパーとロールスコープのマッピングを定義する必要があります。いくつかの共通の設定を共有することで、新しいクライアントの作成を容易にするために、クライアント・スコープを保存すると便利なことがよくあります。これは、 scope パラメーターの値に基づいて条件付きでクレームやロールを要求する場合にも便利です。Keycloakは、このためにクライアント・スコープの概念を提供します。

設定画面ではトップレベルに用意されている`Client Scopes`と`Clients`配下に用意されている`Client Scopes`があるので注意。前者が`client scope`の定義で後者が`client`と`client scope`を結びつけるためのもの

### service account
最後に`service account`について説明しておく。keycloakで単に認証だけ行い、rest apiなどを設定しない場合はこの設定は不要なので読み飛ばして構わない。  

[用語集](https://keycloak-documentation.openstandia.jp/11.0.0/ja_JP/server_admin/index.html#%E3%82%B3%E3%82%A2%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88%E3%81%A8%E7%94%A8%E8%AA%9E)の記述を抜粋すると
> 各クライアントには、アクセストークンを取得するための組み込みサービス・アカウントがあります。

となっているが、これだけ読んでもよくわからないと思う。
まずkeycloakのREST APIを利用するには事前にトークンを取得する必要がある。このトークンは`access token`とよばれOAuth2のそれと同一である。この`access token`を取得する方法は2つある([参照](https://keycloak-documentation.openstandia.jp/master/ja_JP/server_development/index.html#%E7%AE%A1%E7%90%86rest-api))
1. master `realm`のユーザー名とパスワード名で`access token`取得のためのAPIを叩く
2. `service account` を利用して`access token`取得のためのAPIを叩く

1のmaster `realm`のユーザーとパスワードを利用する方法は単純ではあるが、master `realm`にREST API用の`user`アカウントを作る(もしくは既に存在する強い権限をもつ`user`アカウントを使う)必要があるためユーザー名、パスワードの管理を含めて煩雑である。  
2の`service account`を利用すると1の不便を解消できる。`service account`はOIDCのCode Grant Flowを利用する`client`にだけ生成される内部的な`user`である。対象の`client`のcredentialコードを利用するだけでtoken取得のためのAPIを叩くことができる。そのためmaster `realm`に設定を追加する必要がない。また、`service account`を利用したときに得られる`access token`の権限は`client`の`Service Account Role`という箇所から設定できる。ここら辺keycloak自身もOIDCやOAuth2の仕組みを上手く利用しているのだなと感じる。


### 関連図
上述した要素たちを関連図に付け加えると以下のようになる

![keycloak](/content/keycloak-oidc-guide/keycloak.svg)
元々書いてあった`client`,`user`以外に`token`の元情報になる`role`とそれらから派生した要素(`realm role`, `client role`, `service account`)は色をまとめてある。それ以外はkeycloak内部の情報を`token`に反映するときのマッピングや制限であることがわかるだろう。引き続き各要素には管理画面のどこから設定できるかを記載してある。

keycloakの難しさは
* 設定項目の名前から具体的に何が設定されるかがわかりにくいところ
* 設定箇所の多さと、それ故に管理画面内のどこで何が設定されるのかがわかりにくいこと

だが、それぞれ

* 設定項目の名前から具体的に何が設定されるかがわかりにくいところ
  * 本記事の説明を参照しつつkeycloak独自の要素とprotocol側の要素の名称をきちんと区別し、用語の意味を理解する
* 設定箇所の多さと、それ故に管理画面内のどこで何が設定されるのかがわかりにくいこと
  * 関連図を参考に設定する箇所を探す

ことで少しでもkeycloak導入の手助けになれば嬉しい

