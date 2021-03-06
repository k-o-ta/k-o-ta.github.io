+++
title = "ハイパフォーマンス ブラウザネットワーキングを読んだ"
date = 2017-02-01
[taxonomies]
tags = ["books", "browser"]
+++

[ハイパフォーマンス ブラウザネットワーキング](https://www.oreilly.co.jp/books/9784873116761/)を読んだ。ブラウザにまつわるネットワーク技術について、そもそもレイテンシや帯域とはなにかというところから、TCP/UDPやTLSというWebを支えるプロトコル、果てはHTTP2.0やWebRTCという新しいプロトコルまで広く深く学ぶことができる。

## 1章から4章まで
レイテンシ、帯域、TCP、UDP、TLSについて学ぶ。技術革新によってEndtoEndのレイテンシーは改善が見込まれるが、光の速さを追い抜くことはできないため、パフォーマンス向上に寄与する度合いは限定的である。アプリケーションのパフォーマンスを向上させるにはそれを認識した上でパケットの無駄な往復を減らすような、レイテンシを隠蔽できるアプリケーションを構築する必要があると思った。

## 5章から8章まで
ワイヤレスネットワークについて学ぶ。私は以前通信業界のシステム開発をしていたが、専ら有線接続を前提とした世界だったのでこのあたりは新鮮に感じた。特にモバイルユーザーにとってはネットワークの利用方法がバッテリーに大きく影響するのでアプリケーション開発においても気をつけたい。日本国内の特に都市部のユーザーのモバイルネットワーク環境は基本的に良好なので開発するときも有線接続の設定にしてしまうが、ネットワーク環境が悪いときにどういう体験になるかは気にしておきたい(特に月末になるとギガが足りない状態になるので)  
その他モバイルネットワークの特性との相性を気にするべき要素(ビーコンとかリアルタイム解析とかキープアライブとか)や断続的なデータの送受信よりはまとめてデータを送受信する、といったことが書かれている

## 9章から13章まで
HTTPについてまとめられている。実務的にはHTTP1.1までの歴史や最適化についてがとても役立つのだが、個人的にはHTTP2についての出自や目指すところが大変面白かった。光速という物理的制約がある中でプロトコル自体を改善することでより良いパフォーマンスを得る取り組みには本当に頭が下がる。実際にGoogleなどのサービスで運用されているということも実運用に耐えうることを示している。自分もこういうWebに貢献できる仕事をしたいものだ。

## 14章から18章まで
ブラウザのネットワークAPI周りについて書かれている。現職の仕事ではXMLHttpRequest以外はあまり使うことはないのだが、Server-Sent Eventsなんかは社内管理画面でも結構使えそう。現状イベントをメールで送信するフローがよくあるので。WebRTCは良くビデオ通話の文脈で目にするが、もっと汎用的に使えるものだとわかった。ブラウザの中にこんな複雑なものが入っており、それが現実的な速度で動くようになっているの凄すぎないか?


## 総じて
一度ですべてを理解する必要はなく(というか無理)、何度も後で読み返す価値のある本だった
