---
title: Folding@Home 試用心得
tags:
  - sysadmin
  - foss
  - covid19
  - distributed computing
  - container
date: 2020-09-13 12:33:19
---

其實很久很久以前就好像聽過這東西了
只是 3 月的時候因為 COVID-19 所以又好像有媒體在宣傳這個專案

於是那時在終於有自己的機器可以運用的前提下
<!--more-->
(以前拿家裡公用的電腦挖這些，可能家人會有意見吧)

就去看官網看怎麼裝，能不能裝在 Linux 上，然後就拿[官方的 debian 包](https://foldingathome.org/alternative-downloads/)來用用看

發現 `fahcontrol_7.5.1-1_all.deb` 竟然還相依 [GNOME 2.x](https://en.wikipedia.org/wiki/GNOME#GNOME_2) 相關已經沒有人在維護的套件.... ( Python2 + Gtk2 的 [PyGTK](https://en.wikipedia.org/wiki/PyGTK) 就算了?)

於是在那時還沒有時間搞的前提下怒用 [Docker](https://www.docker.com) 的 [Debian Stretch (9.x)](https://wiki.debian.org/DebianStretch) 的映像來跑他的東西....

感覺這專案資助的單位還是滿有來頭的... 應該不至於沒經費繼續更新吧? 只是這部分的開發與維護剛好沒有被重視? 或是覺得不緊急不重要?
