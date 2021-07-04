---
title: Ptt/Ptt2 近期對於 SSH 連線實作的更動
tags:
  - opensource
  - foss
  - ptt
  - ptt2
  - bbs
  - maplebbs
  - openssh
  - libssh
date: 2021-07-04 17:49:11
---


本篇文章將以使用者與 BBS 程式測試者的角度觀察，最近 PttBBS 對於 ssh 連線提供服務方式的更動，以及對於一般使用者可能造成的影響

近 10 年來，BBS 已然沒落，但批踢踢實業坊 ([PttBBS](https://www.ptt.cc)) 依然因為對於社會議題輿論產生重大影響、多位 app 開發者提供第三方手機連線服務...等因素，在尖峰時間維持數萬 ~ 十幾萬人次的同時上線人數。

然而近幾年以來卻開始愈來愈頻繁地發生尖峰時間 Ptt 無法正常登入的負載問題，尤其在近幾個月來更是出現一般使用者認為中規模的事件，如：小區域地震、三級警戒以來每天召開的防疫記者會，Ptt 也仍因過載無法登入的窘況。也開始引發各看板，包含 [SYSOP 站務板](https://www.ptt.cc/bbs/SYSOP/)大大小小的抱怨。

對此站方有什麼積極應對解決的方案呢？其實是有的。以下根據初步觀察大概分成四個階段：觀察、分析、嘗試方案、解決(正式上線)

<!--more-->

## 觀察

為了確認各種連線問題造成的原因，首先 Ptt 站務總監 okcool 先在理論上較多針對 Ptt 的 App 開發者所在看板：[AppsForBBS 板](https://www.ptt.cc/bbs/AppsForBBS/index.html)，籲請各 App 開發者在使用 websocket 連線登入時，配合加上 User-Agent 資訊，以方便統計各軟體使用 Websocket 的連線狀況。

至於其他連線方式(ssh, telnet)，根據既有的公開程式可以得知都會留下相關記錄，因此這方面就不再說明了。


## 分析與討論

另外在 Ptt、Ptt2 SYSOP 板爬梳一些討論資料，以及 [PttBBS 原始碼](https://github.com/ptt/pttbbs)、[PttCurrent 板](https://www.ptt.cc/bbs/PttCurrent/)提供的相關文件後，我們可以找到當初 PttBBS 可以承受同時間 15~6 萬人上線，關鍵在於 PttBBS 會經由 logind 程式接收大量 telnet 連線並為各 Client 顯示登入畫面，登入成功後再導入 mbbsd 進行主要的 bbs 指令操作。

然而，ssh 連線方式的實作則是直接在 Linux Server 開一個 `bbs` / `bbsu` 帳號，login shell 分別設爲 `/home/bbs/bin/bbsrf` 和 `/home/bbs/bin/utf8` ( symbolic link 到 `/home/bbs/bin/bbsrf` )，連線成功後即直接開啟 mbbsd 程式進行登入程序。

引用 BBS 開發者 IepIweidieng@github 在 [CCNS Discord](https://discord.ccns.io) #bbs-dev 的[描述](https://discord.com/channels/330361502643257345/370600485612290060/843869940427522099):

> `logind` 在使用者成功登入後才會將使用者的 `fd` 轉交給 `mbbsd` 並進行 `fork()`。`mbbsd` daemon 模式則是一連線就會 `fork()`。非 daemon 模式則是完全從頭啟動。

而 PttBBS 傳統上給 ssh 連線用的就是 `mbbsd` 的非 daemon 模式，`fork()` 的工作則是交給 OpenSSH Server 的 `sshd` 來進行。

![](https://i.imgur.com/P6w4fjq.png)

因此給 PttBBS 用的 `logind` 效能如何是 PttBBS 開發者可以研究如何改善的，但 OpenSSH Server 的效能調校能改的就相對有限了。

近年來加密連線防範封包側錄的資訊安全意識興起，ssh 連線被各使用者、手機 Apps 預設值，甚至是 BBS 爬蟲程式(如: [PyPtt](https://github.com/PttCodingMan/PyPtt)) 大量採用，因此也開始被懷疑是影響 PttBBS 近年處理大量連線的效能瓶頸。


## 嘗試方案: 暫停 ssh 連線 -> 觀察

今年 5 月，COVID-19 在臺灣的本土疫情爆發，因此 PttBBS 遇到的連線效能瓶頸也更加明顯，那段期間在每日 12pm 就會開始出現系統過載的畫面了，更別說是 2pm 召開防疫記者會的時候。那時候站方還有在 SYSOP 板發布[公告](https://www.ptt.cc/bbs/SYSOP/M.1621416649.A.5E7.html)，籲請使用者盡量改用 Websocket 加密連線方式來代替。但這樣的籲請措施能做的還是有限，一來 Websocket for BBS 在終端機上還沒有一個相對方便的指令連線方案，另外就是會有使用者反應在尖峯時段用 ssh 連線「搶」進去的機率反而比較高。

另外在該期間，Ptt 也[暫停](https://www.ptt.cc/bbs/PttNewhand/M.1621264236.A.23D.html)了 ssh 連線登入。我們目前認為是作為確認 ssh 造成連線壅塞原因的手段。


## 站方解決方案：以 logind 為基礎重新實作 SSH server

今年 5/31，Ptt 系統站長 robertabcd 在 [ptt2.cc](https://term.ptt2.cc) 發佈公告 ( `#1Wj4MGKt (SYSOP)` )，宣佈開始在 ptt2 測試新版 SSH。雖然該次以失敗(原因: 部分 client, 如 PuTTY, 登入畫面顯示仍有問題)，但也為 BBS 提供 SSH 連線兼顧效能的可行性，增添了不小的可能。

後來在 6/7，新版 SSH 再度上線(`#1WlJ2MXQ (SYSOP) [ptt2.cc]`)，這次使用者操作方面穩定性就提升了不少，也因此過了數天後就上線到 ptt.cc。這部分也可以從 `telnet` 或 `nc` 等工具程式針對 port 22 測試而得知。 (`telnet ptt.cc 22`)

以往會得到類似以下結果：
```
$ telnet ptt.cc 22                                                                                        [17:37:31]
Trying 140.112.172.2...
Connected to ptt.cc.
Escape character is '^]'.
SSH-2.0-OpenSSH_8.4p1
```

現在則是：
```
$ telnet ptt.cc 22                                                                                        [17:37:31]
Trying 140.112.172.2...
Connected to ptt.cc.
Escape character is '^]'.
SSH-2.0-bbs-sshd
```

由於 OpenSSH 的版本號碼顯示只能在編譯階段修改，且新版 SSH server 版號顯示也拿掉了 OpenSSH 字樣，因此我們推測新版程式是利用 [libssh](https://www.libssh.org/) 結合既有的 logind 提供服務，使透過 SSH 大量登入的連線可以順利交給 `logind` 處理，解決原本方法在這時產生的效能瓶頸。

![7/3 當日使用者上線統計](https://i.imgur.com/rphrOxQ.png)
![5/13 當日使用者上線統計](https://i.imgur.com/6aPiuHM.png)

目前從 *主選單 > (X) > (U) > (Y) > (U)* 進入昨日使用者統計來觀察 ptt 一天上線人次，已經不若以往每逢尖峰時間必過載的情形發生。但實際上而言能夠承受的密集大量登入程度為何，仍有待時間的考驗，以及是否有重大事件足以提供測試的契機了。


## 結語

在有限的資源下維護開源專案都是不容易的，尤其因為 Ptt 的特殊性質，更難以導入用別人錢(贊助)就可以解決問題的方案。在這樣情況下還能有這樣進展，我們或許該充分體認到這類的事情都不是理所當然的。
