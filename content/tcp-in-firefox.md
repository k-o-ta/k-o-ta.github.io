+++
title = "TCP/IP in Firefox"
date = 2020-07-18
tags = []
+++

### 目的
HTTPとTCP/IPとの関係を述べよという問題を見た。
HTTPはTCP/IPを前提として動いており、階層構造になっている、というような答えなのだが、HTTPのヘッダのhostとportはIPヘッダの宛先IPアドレスとTCPヘッダの宛先ポート番号と情報が被るな、と思った。それぞれ階層が違うので別に不思議ではないのだが。

ブラウザがTCP/IPの宛先を設定する処理と、HTTPの宛先を設定する処理を見たくなった。
#### h2
sentence

#### rfc2626
>  HTTP communication usually takes place over TCP/IP connections. The
   default port is TCP 80 [19], but other ports can be used. This does
   not preclude HTTP from being implemented on top of any other protocol
   on the Internet, or on other networks. HTTP only presumes a reliable
   transport; any protocol that provides such guarantees can be used;
   the mapping of the HTTP/1.1 request and response structures onto the
   transport data units of the protocol in question is outside the scope
   of this specification.
