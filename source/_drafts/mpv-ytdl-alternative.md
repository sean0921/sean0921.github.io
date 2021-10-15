---
title: 如何在串流播放器使用 youtube-dl 的替代品 - 以 mpv 為例
tags:
- youtube-dl
- mpv
- yt-dlp
- video-streaming
---

本篇文章將以 mpv 影音播放器為例，介紹如何在該軟體 0.33 之後的版本快速替換 youtube-dl 外掛所使用的路徑

<!--more-->

## youtube-dl 與 mpv
不想被既有影音平臺使用規則限制的人都知道，除了坊間上流傳來來去去的影音下載網站以外，最知名、自己來用起來最方便的工具就是 `youtube-dl` 了。而 `youtube-dl` 除了直接下載各影音網站的內容儲存外，也可以用將它整合到你喜愛的影音播放軟體，直接播放串流影音。這類軟體包含但不限於 [mpv](https://mpv.io/) 以及[以它為基礎的播放器](https://github.com/mpv-player/mpv/wiki/Applications-using-mpv)。

## 如何下載 mpv
在 Linux/FreeBSD 平臺上安裝 mpv 是一件極度容易的事情，通常透過套件管理員安裝即可，以 Ubuntu 為例就是: `sudo apt install mpv`。而 Windows 使用者則是可以到 mpv 官網列的[下載列表](https://mpv.io/installation/)看哪個安裝方式最符合你的需求，通常我會去[第一個](https://sourceforge.net/projects/mpv-player-windows/files/)選「Download Latest Version」來安裝，解壓縮 `bootstraper` 資料夾點選 `update.bat` 安裝後，對於所有問題一律按「y」回答，就會把 `mpv` 和 `youtube-dl` 都幫你裝好了。剩下如何操作的部分，

## 如何用 mpv 播放串流影音
若你想透過 `mpv` 播放你喜歡的影音網址，你可以在安裝完畢後開啟命令列，輸入 `mpv <影音網址>` 後即可播放，例如嘗試以下指令:
```
mpv https://youtu.be/dQw4w9WgXcQ   ## 支援 YouTube
mpv https://vimeo.com/148751763    ## 支援 Vimeo
```
你也可以將瀏覽器上的 youtube 連結拖入已經開啟的 mpv 視窗中:
![](https://i.imgur.com/Ap5bYB1.png)

當然如果你有可以存取的 [HLS 串流網址](https://zh.wikipedia.org/wiki/HTTP_Live_Streaming)，也可以直接丟給 mpv 來播放。

這樣子當我們不確定 youtube 影片的內容，又不想幫連廢片都不如的內容衝流量，就可以用這樣的方式來觀看，除此之外也仍然有阻擋廣告、硬體解碼(節省電腦運算資源)、一鍵截圖...等結合這兩個軟體各種方便的特性。

## youtube-dl 的爭議與沒落
但這個專案首先歷經了一連串 DMCA 爭議後一度被 GitHub 下架，後來雖然恢復，但開發能量已大不如前，最後一次更新的時間是三個月前，上一次釋出版號的時間也已經是四個月前，回不去以往隔數天~一週就釋出新版號的開發節奏了。這也意味著當各大影音平臺改變策略，嘗試阻擋第三方存取的方式時，這個套件將難以再迅速因應。同時也無法再新增其他影音網站平臺的支援。

## youtube-dl 的替代方案
youtube-dl 是基於 [Unlicence](https://unlicense.org/) 授權，因此也開始陸續出現許多分支來嘗試繼承開發能量，像 DMCA 下架事件就開始出現 [`youtube-dlc`](https://github.com/blackjack4494/yt-dlc)，但後來也無疾而終。

而個人關注，且最近還有持續的開發的分支則是 [yt-dlp](https://github.com/yt-dlp/yt-dlp), 目前有維持一定的開發節奏且持續釋出新版本，是現階段值得使用的替代方案，也可以改善原版 youtube-dl 偶發性無法存取的問題。

## 如何在 mpv 使用 youtube-dl 替代方案
以下以 `yt-dlp` 取代 `youtube-dl` 為例

### 重新命名執行檔(不推薦)
這個方法滿單純的，就是找出 `youtube-dl` 的執行路徑，Windows 平臺你是在哪個路徑執行 `youtube-dl.exe` 這個程式，通常會跟 `mpv.exe` 安裝在同一個資料夾下。Linux/FreeBSD 等平臺則是在一個優先度比較高的 `$PATH` 路徑下，建立一個 symbolic link 連結到 `yt-dlp` 的執行檔路徑。例如:
```bash
ln -rsv "${HOME}/.local/bin/yt-dlp" "${HOME}/.local/bin/youtube-dl"   ## 如果路徑衝突，請自行解決或解除安裝原版 youtube-dl
```

### 更改 mpv 設定檔 (適用於 >= 0.33 版本)
上一個方法看似單純，卻容易在更新套件時把你的環境搞炸。因此我會建議能將 mpv 更新到此版本的使用者都使用此方法。

首先我們可以先找出 mpv 設定檔的存放路徑，相關說明可以在官方文件找到。Windows 的使用者可以將它放在 `%USERPROFILE%\AppData\Roaming\mpv\mpv.conf` 或是同資料夾下的 `portable_config/mpv.conf`，Linux/FreeBSD 使用者則是可以去 `~/.config/mpv/mpv.conf` 新增或更改自己的設定檔。

然後我們只需要在設定檔新增以下一行:
```
script-opts=ytdl_hook-ytdl_path=/home/user/.local/bin/yt-dlp
```
(`/home/user/.local/bin/yt-dlp` 改成你放 yt-dlp 執行檔的路徑)

這樣就不用更改任何環境變數、改檔名或是建立軟連結囉！

## 結語
當我們觀看著影音的同時，我們也正在被平台提供商用各種意想不到的方式觀看著！
當然我們也可以積極關注更為開放、重視分享精神的影音平臺: 像是 [wiwi](https://wiwi.video/about/instance) 推薦的 [PeerTube](https://zh.wikipedia.org/wiki/PeerTube) 或 [LBRY](https://zh.wikipedia.org/wiki/LBRY)。

但短時間內我們無法構造改革，因此我們使用這些工具，來讓我們仍然保有不被平臺限制使用存取方式的選擇，也能讓觀看影片的體驗更加彈性，應用到更多層面。
