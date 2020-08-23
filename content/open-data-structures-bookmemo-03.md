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
#### 1.4, 1.5
セットを使う。[HashSet](https://doc.rust-lang.org/std/collections/struct.HashSet.html)を使った。

#### 1.6
[BTreeSet](https://doc.rust-lang.org/std/collections/struct.BTreeSet.html)を使った。

#### 1.7
[BinaryHeap](https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html)を使った。

#### 1.8
一行ずつ読み取り、偶数番目の行を出力、奇数番目の行を[Vec](https://doc.rust-lang.org/std/vec/struct.Vec.html)に入れて、読み取りが終わったら出力する

#### 1.9
[shuffle](https://docs.rs/rand/0.7.3/rand/seq/trait.SliceRandom.html#tymethod.shuffle)を使う。
