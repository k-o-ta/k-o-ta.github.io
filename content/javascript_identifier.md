http://unicode.org/reports/tr31/tr31-5.html#Default_Identifier_Syntax 記法
<identifier> := <ID_Start> <ID_Continue>*

https://www.ecma-international.org/ecma-262/#prod-IdentifierName

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
  Uppercase letters(https://www.compart.com/en/unicode/category/Lu), lowercase letters(https://www.compart.com/en/unicode/category/Ll), titlecase letters(https://www.compart.com/en/unicode/category/Lt), modifier letters(https://www.compart.com/en/unicode/category/Lm), other letters(https://www.compart.com/en/unicode/category/Lo), letter numbers(https://www.compart.com/en/unicode/category/Nl), stability extensions

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

