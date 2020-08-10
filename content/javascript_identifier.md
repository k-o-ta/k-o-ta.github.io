+++
title = "jsの変数命の規則"
date = 2020-08-11
draft = true
[taxonomies]
tags = ["javascript"]
+++

[ECMAScriptの定義](https://www.ecma-international.org/ecma-262/#prod-IdentifierName)を参照
```
IdentifierName::
  IdentifierStart
  IdentifierName IdentifierPart

IdentifierStart::
  UnicodeIDStart
  $
  _
  \UnicodeEscapeSequence (\\u0026)のような形

UnicodeIDStart::
  any Unicode code point with the Unicode property “ID_Start”

IdentifierName::
  IdentifierStart
  IdentifierName IdentifierPart

IdentifierPart::
  UnicodeIDContinue
  $
  \UnicodeEscapeSequence
  <ZWNJ>
  <ZWJ>

UnicodeIDContinue::
  any Unicode code point with the Unicode property “ID_Continue”
  All of ID_Start, plus nonspacing marks, spacing combining marks, decimal numbers, connector punctuations, stability extensions. These are also known simply as Identifier Characters, since they are a superset of the ID_Start. The set of ID_Start characters minus the ID_Continue characters are known as ID_Only_ Continue characters.

```
* ID_Startは[Unicodeに定義されていて](http://unicode.org/reports/tr31/tr31-5.html#Default_Identifier_Syntax)、以下が含まれる
  * [Uppercase letters](https://www.compart.com/en/unicode/category/Lu)
  * [lowercase letters](https://www.compart.com/en/unicode/category/Ll)
  * [titlecase letters](https://www.compart.com/en/unicode/category/Lt)
  * [modifier letters](https://www.compart.com/en/unicode/category/Lm)
  * [other letters](https://www.compart.com/en/unicode/category/Lo)
  * [letter numbers](https://www.compart.com/en/unicode/category/Nl)
  * stability extensions

せっかくBNF記載されているのに日本語で説明するのも野暮な感じだが、以下あたりは知っておいても良いかもしれない
* 使って良い記号は`$`と`_`のみ(IdentifierStart)
* アルファベット以外のひらがなを使って良い(UnicodeIDStart)
* 数字は先頭以外で使える(UnicodeIDContinue)
* unicodeエスケープ記法を使える

unicodeエスケープ記法の例(コメントアウトしないとSSGがうまく出力してくれなかった)
```javascript
// \u6771 = '京'
// -> "京"
// 東
// -> "京"
```

