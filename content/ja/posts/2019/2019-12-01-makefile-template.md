---
title: "Makefileテンプレート作った"
date: 2019-12-01
draft: false
tags:
- makefile
- gcc
---

Makefileって作るときはいろいろ調べながら作るけど、一度作るとそれ以上触ることないので、よく忘れる。備忘録も兼ねてテンプレとして残しておきます。

{{< alert theme="warning" >}}
全ての環境での動作を保証するものではない。使用する際は、環境に合わせて使う。
{{< /alert >}}

成果物はGitHubにプッシュ済み。  
https://github.com/tommylife88/makefile-template.git

# Makefileテンプレート

```Makefile
# Makefile Template.

# .
# |-- Makefile (This file)
# |-- build
# |   `-- target file.
#     |-- obj
#         `-- object files.
# |-- include
# |   `-- header files.
# `-- source
#     `-- source files.

# Variables
## Directory defines
BUILDDIR = ./build
OBJDIR = $(BUILDDIR)/obj
SRCDIR = ./source
INCDIRS = ./include
LIBDIRS = #-L

## Target name
TARGET = $(BUILDDIR)/a.out

## Compiler options
CC = gcc
CFLAGS = -O2 -Wall
CXX = g++
CXXFLAGS = -O2 -Wall
LDFLAGS =

SRCS := $(shell find $(SRCDIR) -name *.cpp -or -name *.c -or -name *.s)
OBJS := $(SRCS:%=$(OBJDIR)/%.o)
DEPS := $(OBJS:.o=.d)
LIBS = #-lboost_system -lboost_thread

INCLUDE := $(shell find $(INCDIRS) -type d)
INCLUDE := $(addprefix -I,$(INCLUDE))

CPPFLAGS := $(INCLUDE) -MMD -MP
LDFLAGS += $(LIBDIRS) $(LIBS)

# Target
default:
	make all

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CXX) -o $@ $^ $(LDFLAGS)

# assembly
$(OBJDIR)/%.s.o: %.s
	$(MKDIR_P) $(dir $@)
	$(AS) $(ASFLAGS) -c $< -o $@

# c source
$(OBJDIR)/%.c.o: %.c
	$(MKDIR_P) $(dir $@)
	$(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

# c++ source
$(OBJDIR)/%.cpp.o: %.cpp
	$(MKDIR_P) $(dir $@)
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $< -o $@
	
.PHONY: all clean rebuild

clean:
	$(RM) -r $(BUILDDIR)

rebuild:
	make clean && make

-include $(DEPS)

MKDIR_P = mkdir -p
```

## ポイント

### out-of-source ビルド

本Makefileはout-of-sourceビルドとしています。`build`ディレクトリ直下に実行ファイル、`build/obj`ディレクトリにオブジェクトファイルと依存関係リストファイルを生成しています。
out-of-sourceビルドする利点として、

* ソースディレクトリが汚れない
* cleanするときに楽

があります。

### ヘッダファイルの依存関係の解決（-MMD、-MPオプション）

Makefileの依存関係の記述の仕方によっては、ヘッダーファイルのみを編集した場合で、そのヘッダーファイルをインクルードしているソースファイルがコンパイルされないことがあります。しかもたちが悪いことにビルドが通ってしまう可能性もあるため致命的な欠陥になりかねないのです。

これは`-MMD`や`-MP`オプションで解決できます。

* -MMD … 依存関係にあるファイルリストを拡張子.dのファイルに保存しコンパイルを行う。
* -MP … ヘッダーファイルを削除した場合、もともと依存関係にあるファイルはそのヘッダーファイルを探してしまいコンパイルが通らなくなる。本オプションは偽のターゲットを定義することで、削除されたはずのヘッダーファイルを探しても大丈夫なようにしてあげる。

```Makefile
（省略）
SRCS := $(shell find $(SRCDIR) -name *.cpp -or -name *.c -or -name *.s)
OBJS := $(SRCS:%=$(OBJDIR)/%.o)
DEPS := $(OBJS:.o=.d)
（省略）
CPPFLAGS := $(INCLUDE) -MMD -MP
（省略）
-include $(DEPS)
（省略）
```

本Makefileでは`DEPS`変数に全オブジェクトファイルの依存ファイルである拡張子.dファイルを生成し、`-include $(DEPS)`で依存関係の解消を行っています。
