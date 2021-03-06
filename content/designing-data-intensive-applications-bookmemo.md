+++
title = "データ指向アプリケーションデザインを読んだ"
date = 2019-09-24
[taxonomies]
tags = ["books", "distributed systems"]
+++

[データ指向アプリケーションデザイン](https://www.oreilly.co.jp/books/9784873118703/)を読んだ。業務で担当しているアプリケーションをマイクロサービスに分割する上でデータ構造をどのように分けていくかの参考にしたかった。  
今担当しているアプリケーションは形は複数システムに分割しているがデータベースは共有している。そこがボトルネックになって障害が発生したり、いびつな実装をせざるを得ないことがあるのでそれを解決したかった。

マイクロサービス間ではDBを共有しないのが一般的だが、その場合に得られるもの(スケーラビリティなど)や、失うもの(データの一貫性)、できるだけデータの一貫性を保つためにどのような方法が存在するかなどが、かなり具体的かつ丁寧に書かれている。

記録のシステム(Systems of Record)と導出データという考え方も面白かった。記録のシステムは正となるデータで厳密に記録される。導出データは他のデータを変換した結果であり、失っても良い。こうしてデータの性質を分けておくことで柔軟なシステムにできる。たとえば分析用のデータは必要な属性だけを残したり他のデータと予め結合した導出データにすることで使いやすくできるし、キャッシュなども導出データとして考えられる。何よりデータベースをマイクロサービスごとに分割したときに他システムにデータを提供しなければならないときに、API経由で提供する以外にも導出データとしてメッセージングやKVSなどを通して導出データを提供することができる。
