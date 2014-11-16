---
layout: posts
title: "eclipse 常用"
---

### 1.各个子目录相应的build-in.o链接为vmlinux
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
vmlinux: $(vmlinux-lds) $(vmlinux-init) $(vmlinux-main) vmlinux.o $(kallsyms.o) FORCE

vmlinux.o: $(modpost-init) $(vmlinux-main) FORCE

modpost-init := $(filter-out init/built-in.o, $(vmlinux-init))

vmlinux-init := $(head-y) $(init-y)
vmlinux-main := $(core-y) $(libs-y) $(drivers-y) $(net-y)

init-y		:= $(patsubst %/, %/built-in.o, $(init-y))
core-y		:= $(patsubst %/, %/built-in.o, $(core-y))
drivers-y	:= $(patsubst %/, %/built-in.o, $(drivers-y))
net-y		:= $(patsubst %/, %/built-in.o, $(net-y))
libs-y1		:= $(patsubst %/, %/lib.a, $(libs-y))
libs-y2		:= $(patsubst %/, %/built-in.o, $(libs-y))
libs-y		:= $(libs-y1) $(libs-y2)

init-y		:= init/
drivers-y	:= drivers/ sound/ firmware/
net-y		:= net/
libs-y		:= lib/
core-y		:= usr/
</xmp>

### 2.编译各个子目录生成build-in.o
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
$(vmlinux-dirs): prepare scripts
# make -f scripts/Makefile.build obj=$@
# Makefile.build会进行处理并递归到相应的子目录生成build-in.o
	$(Q)$(MAKE) $(build)=$@
</xmp>