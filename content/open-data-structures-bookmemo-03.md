+++
title = "みんなのデータ構造読書メモ 第一章(3)"
date = 2020-08-19
tags = ["algorithm", "data structure"]
draft = true
+++

### 練習問題
Rustで実装した
#### 1.1
スタックを使う。Rustの[Vec](https://doc.rust-lang.org/std/vec/struct.Vec.html)は[push](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.push)と[pop](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.pop)をサポートしているのでそれを使った
<!-- more -->
#### 1.2
引き続きスタックを使う。[Vec](https://doc.rust-lang.org/std/vec/struct.Vec.html)の[with_capacity](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.with_capacity)で50を指定した
#### 1.3
最大でも42行前の値しか出力しないので、42行分格納できれば十分。古い値から使われなくなるので、キューを使う。Rustには[VecDequeue](https://doc.rust-lang.org/std/collections/struct.VecDeque.html)がある
#### 1.4
セットを使う。[HashSet](https://doc.rust-lang.org/std/collections/struct.HashSet.html)を使った。
