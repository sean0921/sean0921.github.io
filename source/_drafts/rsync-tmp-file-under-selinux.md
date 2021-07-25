---
title: 在 SELinux 環境下建立發行版鏡像站需要處理的權限問題
tags:
- rsync
- selinux
- linux
- sysadmin
- mirroring
---

SELinux 是讓 Linux 系統管理者又愛又恨的安全性模組之一，它可以保護我們避免自己或他人不恰當的資源誤用，但更多時候我們則是被它複雜的權限設定所困惑，進而將其設定為 permissive 甚至是 disabled 來逃避現實。這篇筆記主要描述如何從 rsync 運作原理，來理解在 SELinux 啟用的情況下，如何正確設定 rsync 暫存區路徑參數與暫存區權限，節省往後摸索的時間。

<!--more-->

## 會用到的 SELinux 觀念

從鳥哥的中文文件 [^1] 以及 Red Hat 的官方文件 [^2] 我們可以大致理解 SELinux 主要想管理重點包含「程序是否能正確讀取『對應』的檔案資源」。

以使用 nginx 架設單純的靜態網頁伺服器為例，我們只預期外部使用者 nginx 只會存取到: `/var/www/html/` 或是 `/usr/share/nginx/html` 的內容，其他不小心或刻意讀取到其他路徑下的檔案內容的行為都是「邪魔歪道」，因此我們會發現利用 `ls -Z` 列出類似路徑下的檔案參數內容，多會顯示帶有 `httpd_sys_content_t` 這個標籤，表示這些檔案可以被特定的 http 服務存取，例如 Nginx 或 Apache。

因此照這樣來看，如果想要透過 http server 把檔案分享給別人，讓其他路徑下的檔案也能被 http server 存取的方法就是透過 selinux 的管理指令: `semanage`，將該路徑底下的預設檔案標籤也設定為 `httpd_sys_content_t`，甚至如果我們想讓檔案能透過其他的公開服務來存取 (ftp, rsync..)，我們還可以把標籤設定成 `public_content_t`。

這大概就是我們設定 SELinux 相關內容前，大概理解的觀念

## 會用到的 rsync 觀念

把 rsync 用到的觀念講得最簡明扼要的文件大概就是它自己的官方文件了 [^3]

文件把該 rsync 套件會用到的角色(或程序)分成: client, server, daemon, remote shell, sender, receiver, generator

以我們想要同步上游 rsync 鏡像站為例，我們是 client，他們是 server，他們同時也有 daemon 的角色(持續地在背景跑 rsyncd)，也因此我們不會用到 remote shell，也就是不會透過 ssh, rsh.. 等存取他們的 rsync 服務。他們接受我們要求傳檔案給我們，我們接收檔案，所以 sender 會跑在他們那邊，我們是則會跑 receiver，並在接收到檔案列表後 fork 出 generator 和 receiver，進行檔案傳輸的工作。

其中我比較在意的部分在 The Receiver 那邊提到的:
> The file's checksum is generated as the temp-file is built. At the end of the file, this checksum is compared with the file checksum from the sender. If the file checksums do not match the temp-file is deleted.
> ...
> After the temp-file has been completed, its ownership and permissions and modification time are set. It is then renamed to replace the basis file.

以及 `rsync(1)` man page [^4] 提到的:

> This changes the way rsync checks if the files have been changed and are in need of  a  transfer.   Without  this  option, rsync  uses a  "quick check" that (by default) checks if each file’s size and time of last modification match between the sender and receiver.

也就是說

1. 如果有啟用 `--checksum`，在確認 checksum (聽說較新版本 rsync 是用 MD5) 沒問題後，temp file 就會移過去取代原始檔案。
2. 如果沒有啟用 checksum 機制，rsync 會使用「快速檢查行為」("quick check" behavior)，確認最後修改時間和檔案大小沒問題後，temp file 就會「移過去」取代原始檔案。

[^1] http://linux.vbird.org/linux_basic/0440processcontrol.php#selinux
[^2] https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/index
[^3] https://rsync.samba.org/how-rsync-works.html
[^4] https://man7.org/linux/man-pages/man1/rsync.1.html
