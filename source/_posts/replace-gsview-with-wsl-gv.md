---
title: 使用 WSL/WSL2 的 gv 取代過時的 gsview
tags:
- freesoftware
- linux
date: 2020-12-04 12:24:08
---


很多人在 Windows 上觀看從 [GMT](https://www.generic-mapping-tools.org/) 或 其他繪圖軟體產生的 postscript 檔 (`*.ps`) 時，都會選擇下載 Ghostgum GSview 5.0 或破解版 GSview 6.0，然而無論何種版本，該軟體早已不再被官方或第三方維護。有任何軟體問題或是安全性問題基本上等於無解。但 Linux 或其他 Unix-Like 平臺仍有 [GNU gv](https://www.gnu.org/software/gv/) 可以使用與回報相關問題，甚至提供 [git repository](https://git.savannah.gnu.org/cgit/gv.git) 追蹤開發進度。該如何在 Windows 10 使用呢? 現在我們有比較穩定的方法可以解決這個問題了

<!--more-->

最簡單的方法就是[安裝 WSL](https://docs.microsoft.com/zh-tw/windows/wsl/install-win10)，相當於開一個更輕量化的虛擬機，直接進入 WSL 後，在檔案目錄下對要檢視的 `gv filename.ps` 就搞定了，但如果是一些依賴 `gsview32`/`gsview64` 指令的程式呢?

事實上很單純，主要分成以下步驟

## 0. 安裝 WSL, VcXsrv

[VcXsrv](https://sourceforge.net/projects/vcxsrv/) 是個有持續被維護且開放原始碼的 Windows X-server 軟體，安裝完啟動後記得選勾選 `Disable access control`，該程式相關防火牆設定都先打開，若機器長期暴露在公共網路 (有外網ip)，可自行研究 Xorg 的 xauth 怎麼設定，這邊先不細講。

當然 WSL 要自行安裝 `gv` 這個套件，也要檢查你的發行版有沒有順便幫你裝 `ghostscript` 等套件

## 1. 在自己使用者選項設定客製化執行路徑 (`%PATH%`)

相關教學很多，這邊有 [Java 的教學](https://java.com/zh-TW/download/help/path_zh-tw.html) 可以參考

## 2. 在 `%PATH%` 底下創建 batch 腳本

以 WSL 2 為例:

```cmd
@echo on
wsl.exe -d debian env DISPLAY="$(ip route|grep 'default via'|awk '{print $3}'):0.0" gv %*
```

在 DOS batch 腳本的 `%*` 是指吃所有餵進去的參數 (例如 `gv -eof test.ps` 的 `-eof test.ps`)
取成你喜歡的指令名，像我取成 `gsviewwsl.bat`

## 3. 在 Windows 開 cmd，對你要開的檔案進行測試

```
gsviewwsl hello.ps
```

或許 Cygwin 也可以做一樣的事情，但考量 Cygwin 速度沒有比較快且擴充性差很多，這邊就不研究與介紹了，若有 Windows 7 電腦有類似需求，請再用一樣的概念自己整理方法。
