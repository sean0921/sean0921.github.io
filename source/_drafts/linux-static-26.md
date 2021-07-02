---
title: linux-static-26
date: 2020-09-13 12:23:10
tags:
---

在 Debian Stretch 上編 GNU/Linux 2.6.32 的 statically linked binary

目標是放 CentOS 6 上，

裝的套件除了 build-essential 以外還要裝 libncurses5-dev 以及
libgpm-dev (才有包含 static linked 需要的 library 不然要自編)

最後用
env LIBS="-static -L/usr/lib/x86_64-linux-gnu -lncurses -ltinfo -lform \
-lgpm" ./configure

才能把 binary 生出來



感覺理論上應該不用加 -L/usr/lib/x86_64-linux-gnu 這個奇怪的東西

才能讓 GNU autotools 找到用 apt 裝的函式庫，

但如果只用 env LDFLAGS=-static 或 env LIBS=-static 都會 error

之後用 buildroot 試試看好了，如果可行的話? 反正應該擠得出資源?
