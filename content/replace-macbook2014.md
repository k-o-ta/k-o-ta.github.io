+++
title = "macbookのOSをUbuntu22.04にした"
date = 2022-05-22
[taxonomies]
tags = ["macbook"]
[extra]
toc = true
+++

プライベートでmid 2014な13インチmacbookを使ってきたが、流石に古いため新しいOSは提供されなくなり、最近はスピーカーも壊れてしまっていた。下取りに出して新しいmacbookを購入することも考えたが、あまり良い値段も付かないだろうし、半年ほど前にバッテリーは交換したばかりだったので自分でパーツを交換して使い続けることにした。

### やったこと
* スピーカーの交換
* SSDの交換
* OSをUbuntu22.04にする

#### 事前準備
* 何かあったら困るのでmacのbackupをとっておく
* ubuntuのインストールUSBメディアを作成しておく
  * 適当に調べれば出てくる
* macOSのインストールUSBメディアを作成しておく
  * macOSに戻したくなったときに使う
  * USBメディアをフォーマットした上で[手順](https://support.apple.com/ja-jp/HT201372)に従う
#### スピーカーの交換
* [交換用スピーカー](https://www.amazon.co.jp/gp/product/B07Z9233HQ)
* [macbookの裏蓋を開けるドライバー](https://www.amazon.co.jp/gp/product/B07Z9233HQ)
* [macbookの内部で使うドライバー](https://www.amazon.co.jp/gp/product/B002SQLDS2)
手順は[こちら](https://jp.ifixit.com/Guide/MacBook+Pro+13-Inch+Retina+Display+Mid+2014%E3%81%AE%E5%8F%B3%E5%81%B4%E3%82%B9%E3%83%94%E3%83%BC%E3%82%AB%E3%83%BC%E3%81%AE%E4%BA%A4%E6%8F%9B/27848)を参考にした。右側スピーカーだけ変えて音のバランスが狂うのも嫌だったので、左側も換えた。右側の交換ができれば、手順は載っていないが左側は難しくないと思う。ついでに手元にCPUグリスのあまりがあったので、塗り直しておいた。[この手順](https://jp.ifixit.com/Guide/MacBook+Pro+13-Inch+Retina+Display+Mid+2014+%E3%81%AE%E3%83%92%E3%83%BC%E3%83%88%E3%82%B7%E3%83%B3%E3%82%AF%E3%81%AE%E4%BA%A4%E6%8F%9B/27839)にしたがってヒートシンクを取り外してグリスを塗り直せば良い。

![inside-mac-book](https://www.dropbox.com/s/gyd9hlumns2soz6/inside-macbook.jpg?dl=1)
#### SSDの交換
購入時から256GBのSSDを使い続けていたが、だいぶ古い & 容量も物足りなかったので新しいものに交換した。
* [SSD](https://www.amazon.co.jp/gp/product/B08C2S7HTT)
* [SSDのアダプター](https://www.amazon.co.jp/gp/product/B01CWWAENG)
macbookので利用されているSSDはM.2とは型が違うのでアダプターが必要。適当にググって評判が良さそうなのを使った。
手順は[引き続きifixit](https://jp.ifixit.com/Guide/MacBook+Pro+13-Inch+Retina+Display+Mid+2014+SSD%E3%81%AE%E4%BA%A4%E6%8F%9B/27849)を利用した。SSDをアダプターに取り付けるときに結構硬いので破壊しないように気をつけながら力を込める必要があった。交換前に新しいSSDにデータを移しておく方法もよく見かけるが、今回はOSもUbuntuにする予定だったので、データ移行はしなかった。
### OSをUbuntu22.04にする
* 作っておいたUbuntuのインストールUSBメディアを挿してoptionキーを押したまま電源を入れるとBIOSが起動するので、ボリュームの選択画面でUSBメディアを選択して、Ubuntuをインストールしていく

![ubuntu-on-mac](https://www.dropbox.com/s/no6sjpp6mgw2spl/ubuntu-on-mac.jpg?dl=1)

#### 感想
スピーカーは直ったし、新しいOSというのはそれだけで気分が良い。一点不満があるとすればバッテリーはmacOSほど長持ちしていない気がしているところ。
