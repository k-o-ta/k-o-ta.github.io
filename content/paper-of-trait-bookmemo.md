+++
title = "Schärli(シェルリ)のtraitに関する論文を読んでRustのtraitと比べた"
date = 2016-06-22
[taxonomies]
tags = ["books", "trait", "Rust"]
+++

Rustの特徴の一つとしてtraitがあるが、traitを考案した論文があると聞いたので読んでみた。  
[Nathanael Schärliらの論文](http://www.ptidej.net/courses/ift6251/fall06/presentations/061122/061122.doc.pdf)は25ページくらいの論文で英語だが平易な文体なので読むには苦労しないと思う。

## 出自
2003年にECOOP( European Conference on Object-Oriented Programming )にて、刊行されたNathanael Schärli(シェルリ)らによる論文に記載されており、継承に代わるコードのコンポーネント化や再利用の手法として紹介されている。

## 継承の問題点
継承を実現する方法として一般的であるClassの役割は以下の2つである
1. instance generator(インスタンスの生成)
  * Classはそれ自身で完結している方が良い(Classが大きくなる圧がかかる)  
2. reuse of code(コードの再利用)
  * Classはできるだけ小さい方が良い
1と2の役割が競合することが多い  
また、1の役割としては各クラスがClassのヒエラルキーにおいてそれぞれ決まった場所に配置されていることが期待されるが、2のコードの再利用の単位としてはヒエラルキーを問わずのあらゆる場所で使われることが期待される。このように、コードのコンポーネントの単位として一貫性を保ちにくい。

### 単一継承の問題点
* 2つの特性を持ったClassを表現しづらい

### 多重継承の問題点
* 複数継承元での同一メソッドのconflict
* superがどの親を指しているか分かりづらい

## mix-inの問題点
コードの再利用の手法としてClassの他にmix-inもよく使われるが、問題点がある
1. mix-inした順序で同一名のメソッド内の実行順序が変わる
2. glue codeがあちこちに散らばる
```
define mixin_double {
  super.property * 2 # <- super.propertyはmix-inされる側のpropertyでmoduleのメソッド内でmix-in先と糊付けしている
}
```

## traitの構成要素
1. (振る舞いを定義する)メソッドを実装する。実装を持つのでmix-in的な要素である
2. Class側で実装が必要なメソッドの宣言をする。実装は持たないのでinterface的な要素である
3. 1のメソッドはtraitを付与される構造体(やclass)のpropertyにアクセスしない(glue codeの排除)
4. 複数traitが付与されていても付与している順序は意味を持たない
  * 複数traitで重複したメソッドがある場合は付与された側で呼び出すメソッドを決める(多重継承やmix-inの問題の解消)
5. traitは付与されたclassのsemanticsに影響しない
  * traitに定義されているメソッドをtraitを付与されているclassにコピペして、traitを外しても同じ挙動になる

### Rustの場合
Rustは1,2,3,4,5をすべて満たしているが、4の複数trait間での重複メソッドの解決においてtrait名を指定することで複数traitのメソッドを出し分けることができる([UniversalFunctionCallSyntax](https://doc.rust-lang.org/book/ufcs.html))が、この手法は論文では非推奨な方法とされている
