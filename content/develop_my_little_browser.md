+++
title = "自作ブラウザを作る(2022-03-19更新)"
date = 2022-03-19
[taxonomies]
tags = ["browser", "自作ブラウザ"]
[extra]
toc = true
+++

自作ブラウザを作ってみたいので試した記録をここに残していく

[repository](https://github.com/k-o-ta/femto-re)

### 環境構築
* 言語: C++
  * 普段趣味の開発ではRustを使っているが、C++を使ってみればRustの嬉しさ、辛さがもっとよくわかりそう
  * 世で一般的に使われているブラウザのはC++で動いている。開発に困ったときに実物のコードを見て参考にするなら言語が同じほうがちょっとうれしいかもしれない
* GUI: GTK+
  * あまり理由はないが、クロスプラットフォームで動くのと好きなブラウザ(firefox)で採用されているので
    * とは言え、最初はubuntuだけで動けば良い
* build方法
  * CMakeを使う
    * C++の開発環境としてCLionを利用したい。CLionが標準的にサポートしているのがCmake

まず開発を始められるのが重要なのでそこまで色々考慮せずに始める。困ったら作り直せば良い。

### window
どこから作り始めようか考えたときに、偶然[Rebuild.fm](https://rebuild.fm/324/)を聴いていたらruiさんの話で[SerenityOS](https://serenityos.org/)という自作OSの作者が「どこから始めても良いがGUIが好きだからGUIから始めた。作りたいものがmockでも動き始めればそれを本物にしたくなるから興味が続く」といった内容のものがあった。自分に当てはめて考えるとやはりまずはwindowが立ち上がって何かしら表示されるのを見たいので、windowを作るところから始めた。

C++でGTK+を使うには[gtkmm](http://www.gtkmm.org/en/)を使う。また、文字列を表示させるのも簡単なことならGTK+でできるが、より高度なtextのレンダリングを可能にする[Pango](https://docs.gtk.org/Pango/)を使うことにした。Firefoxでも採用されていたらしい

```CMakeLists
# CMakeLists

cmake_minimum_required(VERSION 3.20)
project(femto_re)

set(CMAKE_CXX_STANDARD 20)

# external modules
find_package(PkgConfig REQUIRED)
PKG_CHECK_MODULES(GTKMM3 REQUIRED gtkmm-3.0)
PKG_CHECK_MODULES(PANGOMM REQUIRED pangomm-1.4)

INCLUDE_DIRECTORIES(${GTKMM3_INCLUDE_DIRS})
LINK_DIRECTORIES(${GTKMM3_LIBRARY_DIRS})
ADD_DEFINITIONS(${GTKMM3_CFLAGS_OTHER})

INCLUDE_DIRECTORIES(${PANGOMM_INCLUDE_DIRS})
LINK_DIRECTORIES(${PANGOMM_LIBRARY_DIRS})
ADD_DEFINITIONS(${PANGOMM_CFLAGS_OTHER})

add_executable(femto_re femto-re.cpp)
TARGET_LINK_LIBRARIES(femto_re ${GTKMM3_LIBRARIES})
```

gtkmmでpangoを使ってtextをレンダリングする方法は[document](https://developer-old.gnome.org/gtkmm-tutorial/3.24/sec-drawing-text.html.en)に載っているが、一点気をつけることは描画内容を変更するときは`Gtk::DrawingArea`を継承したクラスの`on_draw`メソッド内で行うようにすることだ。この先「parseが完了した後に描画する」みたいなことをするが、parse完了を[signal](https://developer-old.gnome.org/gtkmm-tutorial/3.24/chapter-signals.html.en)で送信して受信した`DrawingArea`がそれを描画、のようにする場合signalを受信したcallbackでは直接描画内容を変更せずに[queue_draw()](https://developer-old.gnome.org/gtkmm/stable/classGtk_1_1Widget.html#a63912f0924724c78298882b7c85139b4)呼び出すようにする。そして次の`on_draw`メソッドの実行時に描画内容を変更すれば良い

### signal
editing...

