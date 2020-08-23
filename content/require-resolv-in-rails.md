+++
title = "Railsでresolvを使うときにrequireするか"
date = 2020-08-24
tags = ["Ruby", "Ruby on Rails"]
+++

## 結論
`require 'resolv'`する

## 背景
Railsを書いていて、[resolv](https://docs.ruby-lang.org/ja/latest/library/resolv.html)を使う場面があった。開発環境では`require 'resolv'`なしでも動いていたが、ステージング環境で試したところ`uninitialized constant Resolv`が出た。確認したところ開発環境では[eventmachine](https://github.com/eventmachine/eventmachine/blob/v1.2.7/lib/eventmachine.rb#L39)で`require`され、`eventmachine`は[thin](https://github.com/macournoyer/thin/blob/v1.7.2/lib/thin.rb#L7)で`require`されていた。thinは開発環境だけで利用していたため開発環境とステージング環境で挙動が変わった。

ところで、すべての環境でthinを使っていても`require 'resolv'`は入れておいたほうがよい。アプリケーションサーバーの内部で何をrequireしているかはアプリケーションコード側からは気にするべきではないので。

調査したときはresolvに適当に例外になるコードを入れて`require 'resolv'`を失敗させてrequireしている箇所を確認したが、もう少し楽な方法はあるだろうか。
