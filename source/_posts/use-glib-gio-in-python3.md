---
title: 在 Ubuntu 桌面中使用 Python 將檔案丟到資源回收桶
tags:
  - GLib
  - Gio
  - Gtk
  - GNOME
  - python
  - data operation
  - linux
  - Ubuntu
  - opensource
date: 2022-05-05 23:02:56
---


假如你在 Ubuntu 使用 Python 進行資料處理，對於一些檔案懷抱著想移出視線又怕刪掉就回不去的情感，怎麼辦? 這篇文章介紹的工具與概念，或許可以提供你一個對於整合桌面環境更加友善的方向。

<!--more-->

## 初始情境
- 我們在用 Python 進行資料處理的時候，會想將不要的檔案移出工作目錄，但又不想要不小心誤刪檔案而救不回來。類似 mv 的操作也會有同名檔案處理起來相對複雜的問題。
- Python 的標準函式庫的檔案相關操作，無法滿足在桌面使用情境下對應的操作需求，以這邊為例就是*把檔案丟到資源回收桶*。

## 可利用方法
- 我們想要在 GNOME 桌面環境將檔案丟到資源回收桶，有哪些方式呢:
    1. 使用檔案總管(Nautilus)將檔案拖拉到資源回收桶資料夾
    2. 透過 `gio` 對檔案進行操作
    3. 透過呼叫 Glib 中 Gio 的相關函式對檔案進行操作
- 這邊要介紹的主要是第三種，而第二種為第三種的簡易應用。

## 甚麼是 Glib、在 Gtk 家族裡的關係?
- GLib 是 GNOME 將原本 gtk 函式庫裡面跟圖形介面無關的底層程式碼獨立出來的一個專案, 它們和其他 gtk 相關專案的關係圖如下:
![](https://upload.wikimedia.org/wikipedia/commons/thumb/a/ab/GTK%2B_software_architecture.svg/2560px-GTK%2B_software_architecture.svg.png)


## 如何使用 Python 將檔案丟到資源回收桶
- 首先，作業系統環境一定要安裝 GLib 函式庫，Python 套件則是需要安裝 PyGObject
- 若想透過 `gio` 等 shell 指令直接對 GLib 進行利用，在 Debian/Ubuntu 下需要確認是否有安裝到 `libglib2-bin` (該類發行版套件會切比較細)

### 相關程式碼
```python
#!/usr/bin/env python3

file_path = '/path/to/filename'
from gi.repository import Gio
a=Gio.File.new_for_path(file_path)
if a.trash() == True:
    print(f'{file_path} is moved to trash!')
```

以上程式碼大致可以對應到 `gio trash /path/to/filename` 這個指令，若是對應圖形介面操作，就是把 `/path/to/filename` 丟到資源回收桶，完工

## API Reference
這邊直接附上 GLib 的函式庫(1)，至於 PyGObject 那一側則還沒找到官方詳細文件，因此先拿它恰巧有出沒過的文件連結(2)替代，若有找到更完整的資料歡迎提供！
1. <https://lazka.github.io/pgi-docs/Gio-2.0/classes/File.html#Gio.File>
2. <https://pygobject.readthedocs.io/en/latest/guide/threading.html?highlight=Gio.File#asynchronous-operations>

## 應用: send2trash
於是這樣簡潔方便的介面使得 [send2trash](https://github.com/arsenetar/send2trash/) 這個第三方 Python 套件於 2013 年即開始支援以 Gio 來進行檔案移至資源回收筒的[實作](https://github.com/arsenetar/send2trash/blob/master/send2trash/plat_gio.py)方式。是不是相當單純呢?

## 結語與討論
GLib 除了本文介紹的範例外，也有許多有趣的模組與函式可以讓我們探索與應用，理解其概念，也將對我們開發、協助除錯 GNOME、GTK 相關開源桌面程式有莫大幫助，是個值得一探究竟的專案！
