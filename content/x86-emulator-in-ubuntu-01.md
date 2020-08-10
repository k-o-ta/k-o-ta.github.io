+++
title = "自作エミュレータで学ぶx86アーキテクチャコンピュータが動く仕組みを徹底理解！をubuntuで進める"
description = "x86-emulator-in-ubuntu"
date = 2018-05-27
tags = ["x86-emulator"]
+++

* OS: ubuntu16.04
* gcc: (Ubuntu/Linaro 6.3.0-18ubuntu2~16.04) 6.3.0 20170519

## Chapter1
### 1.3
gccコマンドは以下。`-m32` で32bitバイナリにコンパイルされる
~~~
gcc -Wl,--entry=func,--oformat=binary -nostdlib -fno-asynchronous-unwind-tables -m32 -o casm-c-sample.bin casm-c-sample.c
~~~

ndisasmはapt-getでnasmをinstallすると一緒に入ってくる
~~~
sudo apt-get install nasm
~~~

## Chapter2
### 2.3
`emu2.3`のMakefileを修正する必要がある
- `TARGET`は`.exe`である必要がないので、`TARGET = px86`
- `CC`でコンパイルに利用するgccを指定している。書籍だとサンプルコード内にあるgccを使うがそれはwindow用なので、ubuntuにinstallされているgccを使う。`CC = gcc`
- `CC`で`Z_TOOLS`を参照しなくなったので、`Z_TOOLS`を削除する

最終的に`Makefile`は以下のようになる
~~~Makefile
TARGET = px86
OBJS = main.o

CC = gcc
CFLAGS += -Wall

.PHONY: all
all :
	make $(TARGET)

%.o : %.c Makefile
	$(CC) $(CFLAGS) -c $<

$(TARGET) : $(OBJS) Makefile
	$(CC) -o $@ $(OBJS)
~~~


## Chapter3
### 3.2, 3.4, 3.7, 3.10, 3.12
`Makefile`をChapter2と同様に修正する

### 3.7
`emu3.7`のエミュレーターは`Makefile`をこれまで同様に修正すれば問題ないが、`exec-c-test`と`exec-arg-test`の`Makefile`はそのままではコンパイルできない。`CFLAGS`に`-m32`オプションを追加し、`LDFLAGS`に`-m elf_i386`オプションを追加する必要がある。
以下の様な`Makefile`になる
~~~
TARGET = test.bin
OBJS = crt0.o test.o
Z_TOOLS = ../z_tools

CC = gcc
LD = ld
AS = nasm
CFLAGS += -nostdlib -fno-asynchronous-unwind-tables \
	-I$(Z_TOOLS)/i386-elf-gcc/include -g -fno-stack-protector -m32
LDFLAGS += --entry=start --oformat=binary -Ttext 0x7c00 -m elf_i386

.PHONY: all
all :
	make $(TARGET)

%.o : %.c Makefile
	$(CC) $(CFLAGS) -c $<

%.o : %.asm Makefile
	$(AS) -f elf $<

$(TARGET) : $(OBJS) Makefile
	$(LD) $(LDFLAGS) -o $@ $(OBJS)
~~~

また、リスト3.29まで実装した段階で`exec-c-test`を試そうとすると、`add_rm32_imm8`が未実装のため`test.bin`が最後まで完走できない。`add_rm32_imm8`はリスト3.34で実装するので問題はないが、書籍の環境ではこの時点で`add_rm32_imm8`が必要だったかどうかはわからない。もしかしたら`inc_rm32`が使われていたのが、gccのバージョンが変わったせいで`add_rm32_imm8`が使われるようになったのかもしれないが、そこまでは確認していない。
