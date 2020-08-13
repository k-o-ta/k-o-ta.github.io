+++
title = "みんなのデータ構造読書メモ 第一章(1)"
date = 2020-08-14
tags = ["algorithm", "data structure"]
+++

## 第1章

### 1.1 効率の必要性
#### 大きなデータセット
* Googleの検索に8.5秒も必要、とされているのは注釈欄のコンピューターの速度が数ギガヘルツ(数十億回/秒)なのを{% katex(block=false) %}10^9{% end %}回/秒として、85億件のWebページをすべて見ようとすると(データセットの要素をすべて見るという前提はその前の段落に書いてある){% katex(block=false) %}\frac{8.5\times10^8}{10^9} = 8.5{% end %}秒かかるということ

<!-- more -->

* 38250台のサーバーが必要、とされているのは一つのリクエストに8.5秒かかるという前提で複数サーバーで並列処理できた場合、一つのリクエストに1秒でレスポンスするには1/8.5台のサーバーが必要。秒間4500クエリ受け付けているので{% katex(block=false) %}4500 \times \frac{1}{8.5} = 38250{% end %}台のサーバーが必要ということ


### 1.3 数学的背景
#### 1.3.1 指数と対数
* 自然対数と二進対数との比較
{% katex(block=true) %}
\ln k = \frac{\log_2 k}{\log_2 e} = \frac{\log_2 k}{\frac{\log_e e}{log_e 2}} = \log_e 2 \times \log_2 k = \ln 2 \times \log_2 k
{% end %}
#### 1.3.2 階乗
* {% katex(block=false) %}\ln (n!){% end %}の近似値はその前で示されているスターリングの近似({% katex(block=false) %}n! = \sqrt{2\pi n} (\frac{n}{e})^n e^{\alpha(n)}{% end %})を使って
{% katex(block=true) %}
\ln (n!) \newline
= \ln (1 \times 2 \times ... \times n) \newline
= \ln (\sqrt{2\pi n} (\frac{n}{e})^n e^{\alpha(n)}) \newline
= \ln \sqrt{2\pi n} + \ln (\frac{n}{e})^n + \ln e^{\alpha(n)} \newline
= \ln (2\pi n)^\frac{1}{2} + \ln n^n + \ln e^{-n} + \ln e^{\alpha(n)} \newline
= \frac{1}{2}\ln (2\pi n) + n\ln n - n + \alpha (n)
{% end %}

#### 1.3.3 漸近記法
ビッグオー記法は今までなんとなく使っていたが正しい定義は初めて見た。{% katex(block=false) %}O(f(n)){% end %}は集合を表していて、nがある程度より大きいときは定数倍しても常に{% katex(block=false) %}f(n){% end %}より小さくなる関数の集合、というような意味合いだった。{% katex(block=false) %}f(n){% end %}はどんな関数でも構わないはずだが、アルゴリズムの世界でよく使われるのが{% katex(block=false) %}\alpha{% end %}(定数)、 {% katex(block=false) %}\log n{% end %}、{% katex(block=false) %}n^b{% end %}、{% katex(block=false) %}c^n{% end %}などなのだろう。ある関数が{% katex(block=false) %}O(f(n)){% end %}に含まれるということは定数倍しても{% katex(block=false) %}f(n){% end %}に満たなく、ある関数とはアルゴリズムの世界では例えば実行時間である。つまり実行時間を表す関数が{% katex(block=false) %}O(f(n)){% end %}に含まれるということは、最悪でも{% katex(block=false) %}f(n){% end %}内には実行完了するし、{% katex(block=false) %}f(n){% end %}に含まれる関数の間での実行時間の差は大きくても定数倍なので丸めて{% katex(block=false) %}O(f(n)){% end %}で同じ、といってしまって良いよね、ということである。

#### 1.3.4 ランダム性と確率
* コインをk回投げたときの表が出る回数の期待値を期待値の定義を使って求めている。{% katex(block=false) %}\dbinom{k}{i}{% end %}は
{% katex(block=true) %}{}_k C _i{% end %}
のようである。
二項係数の性質{% katex(block=false) %}i\dbinom{k}{i} = k\dbinom{k-1}{i-1}{% end %}も、
{% katex(block=true) %} {}_k C_i = k{}_{k-1} C_{i-1}{% end %}
で表される。
