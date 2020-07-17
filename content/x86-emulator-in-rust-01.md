+++
title = "自作エミュレータで学ぶx86アーキテクチャコンピュータが動く仕組みを徹底理解！をrustで進める: 2章"
description = "x86-emulator-in-rust"
date = 2018-05-27
tags = ["x86-emulator"]
+++

### Chapter2
#### 2.3
Emulator Structを作ってそこにメソッドを生やしていく。基本的にはサンプルコードをシンプルに真似るだけで良いが、`short_jump`命令は符号付きの即値をeip(命令ポインタ: 符号なし)に加算する箇所だけ、即値の正負を判定して、絶対値をとってから加算/減算を行う必要がある。
~~~rust
impl Emulator {
    fn short_jump(&mut self) {
        let diff: i8 = self.get_sign_code8(1) + 2;
        match diff > 0 {
            true => {
                self.eip += diff as u32;
            }
            false => {
                self.eip -= diff.abs() as u32;
            }
        }
    }
}
~~~

i8型のdiffをそのままu32にキャストしても、diffが負数の場合はCのサンプルと同じ挙動にはならない

コードは[こちら](https://github.com/k-o-ta/rust_emu/tree/emu2.3)

