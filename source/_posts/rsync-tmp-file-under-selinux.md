---
title: 在 SELinux 環境下建立發行版鏡像站需要處理的權限問題
tags:
  - rsync
  - selinux
  - linux
  - sysadmin
  - mirroring
date: 2021-07-25 18:34:20
---

這篇筆記主要描述如何從 rsync 運作原理，來理解在 SELinux 啟用的情況下，如何正確設定 rsync 暫存區路徑參數與暫存目錄權限。rsync 接收同步資料時，會先將抓取未驗證的檔案放在暫存目錄，經校驗無誤後再移動到同步目的地。如果 rsync 設定 `--temp-file` 目錄路徑的預設 context 與目的地目錄的不同，則會發生目的地出現檔案實際 context 與該路徑下預設內容不符的情況，進而造成服務讀取時發生權限錯誤問題。解決方法有二: 1. 將暫存目錄的 context 設定成和同步目的地的一樣。 2. 取消設置 `--temp-file` 參數，此時未驗證的暫存檔就會存放在同步目的地所在的目錄底下。

<!--more-->

## 簡介
SELinux 是讓 Linux 系統管理者又愛又恨的安全性模組之一，它可以保護我們避免自己或他人不恰當的資源誤用，但更多時候我們則是被它複雜的權限設定所困惑，進而將其設定為 Permissive 甚至是 Disabled 來逃避現實。這篇筆記主要描述如何從 rsync 運作原理，來理解在 SELinux 啟用的情況下，如何正確設定 rsync 暫存區路徑參數與暫存區權限。

## 會用到的 SELinux 觀念
從鳥哥的中文文件<sub>[1]</sub> 以及 Red Hat 的官方文件<sub>[2]</sub> 我們可以大致理解 SELinux 主要想管理重點包含「程序是否能正確讀取『對應』的檔案資源」。

以使用 [nginx](https://zh.wikipedia.org/wiki/Nginx) 架設單純的靜態網頁伺服器為例，我們只預期外部使用者 [nginx](https://zh.wikipedia.org/wiki/Nginx) 只會存取到: `/var/www/html/` 或是 `/usr/share/nginx/html` 的內容，其他非經管理者允許，不小心或刻意讀取到其他路徑下的檔案內容的行為可能都是「邪魔歪道」。

因此我們會發現利用 `ls -Z` 列出類似路徑下的檔案參數內容，多會顯示帶有 `httpd_sys_content_t` 這個標籤，表示這些檔案可以被特定的 http 服務存取，例如 Nginx 或 Apache。

照這樣來看，如果想要透過 http server 把檔案分享給別人，讓其他路徑下的檔案也能被 http server 存取的方法就是透過 selinux 的管理指令: `semanage`，將該路徑底下的預設檔案標籤也設定為 `httpd_sys_content_t`

如果我們想讓檔案能透過其他的公開服務來存取 (ftp, rsync..)，我們還可以把標籤設定成 `public_content_t`。

這大概就是我們設定 SELinux 相關內容前，大概要理解的觀念


## 會用到的 rsync 觀念
把 rsync 用到的觀念講得最簡明扼要的文件大概就是它自己的官方文件了<sub>[3]</sub>

文件把該 rsync 套件會用到的角色(或程序)分成: client, server, daemon, remote shell, sender, receiver, generator

以我們想要同步上游 rsync 鏡像站為例，我們是 client，他們是 server，他們同時也有 daemon 的角色(持續地在背景跑 `rsyncd`)，也因此我們不會用到 remote shell，也就是不會透過 [`ssh`](https://zh.wikipedia.org/wiki/Secure_Shell), [`rsh`](https://zh.wikipedia.org/wiki/%E8%BF%9C%E7%A8%8B%E5%A4%96%E5%A3%B3)... 等傳輸協定來存取他們的 rsync 服務，而是使用 rsync 自己的協定。他們接受我們要求傳檔案給我們，我們接收檔案，所以 sender 會跑在他們那邊，我們是則會跑 receiver，並在接收到檔案列表後 fork 出 generator 和 receiver，分別進行檔案檢查與接收檔案的工作。

其中我比較在意的部分在 The Receiver 那邊提到的:
> The file's checksum is generated as the temp-file is built. At the end of the file, ***this checksum is compared with the file checksum from the sender. If the file checksums do not match the temp-file is deleted.***
> ...
> After the temp-file has been completed, its ownership and permissions and modification time are set. It is then ***renamed to replace the basis file***.

以及 `rsync(1)` man page<sub>[4]</sub> 提到的:

> This (`--checksum`) changes the way rsync checks if the files have been changed and are in need of  a  transfer.   Without  this  option, rsync  uses a  "quick check" that (by default) checks if each ***file’s size*** and ***time of last modification*** match between the sender and receiver.

也就是說

1. 如果有啟用 `--checksum`，在確認 checksum (聽說較新版本 rsync 是用 MD5) 沒問題後，temp file 就會從 **暫存路徑** *移過去* 取代原始檔案。
2. 如果沒有啟用 checksum 機制，rsync 會使用「快速檢查行為」("quick check" behavior)，確認最後修改時間和檔案大小沒問題後，temp file 就會從 **暫存路徑** *移過去* 取代原始檔案。


## 實例解說: 在 SELinux 啟用的系統上鏡像 Arch Linux 套件庫
Arch Linux 是一個相當新穎、簡潔的 **x86_64** 發行版，注意這邊也因為它官方只支援 x86\_64 這個架構，因此實際把它官方套件庫的全部檔案抓下來，會發現不含 iso 檔大概只有 70 GiB 不到 ( Ubuntu 官方套件庫不含 iso 檔大概要佔用將近 1.5T 的容量)。也因此較適合拿來用一般的硬體資源來練習架設鏡像站。

但如果我們使用 Arch Linux 官方附的建議腳本<sub>[5]</sub> 來同步上游 rsync mirror 的套件庫，並把每個留白的參數都根據自己需求填入資訊，會發生什麼事情呢?

首先，假設我們要存放 mirror 內容的路徑是 `/mnt/mirror`，我們仍先把要共享 mirror 檔案內容的目錄及其下所有檔案的標籤都設爲 `public_content_t`:

```
semanage fcontext -a -t public_content_t "/mnt/mirror(/.*)?"
restorecon -Rv /mnt/mirror
```

接著再來執行我們修改過後的同步腳本，然後回去 `ls -Z` 看看:

```
unconfined_u:object_r:unlabeled_t:s0 community
unconfined_u:object_r:unlabeled_t:s0 community-staging
unconfined_u:object_r:unlabeled_t:s0 community-testing
unconfined_u:object_r:unlabeled_t:s0 core
unconfined_u:object_r:unlabeled_t:s0 extra
unconfined_u:object_r:unlabeled_t:s0 gnome-unstable
unconfined_u:object_r:unlabeled_t:s0 images
unconfined_u:object_r:unlabeled_t:s0 kde-unstable
unconfined_u:object_r:unlabeled_t:s0 lastsync
unconfined_u:object_r:unlabeled_t:s0 lastupdate
unconfined_u:object_r:unlabeled_t:s0 multilib
unconfined_u:object_r:unlabeled_t:s0 multilib-staging
unconfined_u:object_r:unlabeled_t:s0 multilib-testing
unconfined_u:object_r:unlabeled_t:s0 pool
unconfined_u:object_r:unlabeled_t:s0 staging
unconfined_u:object_r:unlabeled_t:s0 testing
```

咦? 我們沒看錯吧? 於是我們再次 `restorecon -Rv /mnt/mirror` 後，等待下次更新:

```
unconfined_u:object_r:public_content_t:s0 community
unconfined_u:object_r:public_content_t:s0 community-staging
unconfined_u:object_r:public_content_t:s0 community-testing
unconfined_u:object_r:public_content_t:s0 core
unconfined_u:object_r:public_content_t:s0 extra
unconfined_u:object_r:public_content_t:s0 gnome-unstable
unconfined_u:object_r:public_content_t:s0 images
unconfined_u:object_r:public_content_t:s0 kde-unstable
unconfined_u:object_r:unlabeled_t:s0 lastsync
unconfined_u:object_r:unlabeled_t:s0 lastupdate
unconfined_u:object_r:public_content_t:s0 multilib
unconfined_u:object_r:public_content_t:s0 multilib-staging
unconfined_u:object_r:public_content_t:s0 multilib-testing
unconfined_u:object_r:public_content_t:s0 pool
unconfined_u:object_r:public_content_t:s0 staging
unconfined_u:object_r:public_content_t:s0 testing
```

現在變成有被更新的檔案都會被 unlabeled 了，這是為什麼呢?

後來經歷一連串的測試之後，終於確定了問題出在 `--temp-file` 的這個參數，假設我們藉由該參數設定 rsync 暫存路徑為 `/mnt/mirror_tmp`，接下來我們來進行一個小實驗，任意設定該路徑的 selinux 標籤，改成對 rsync client 沒任何用途的 `rsync_tmp_t`，並把 `/mnt/mirror` 的標籤回復後，等待下次進行同步:

```
unconfined_u:object_r:public_content_t:s0 community
unconfined_u:object_r:public_content_t:s0 community-staging
unconfined_u:object_r:public_content_t:s0 community-testing
unconfined_u:object_r:public_content_t:s0 core
unconfined_u:object_r:public_content_t:s0 extra
unconfined_u:object_r:public_content_t:s0 gnome-unstable
unconfined_u:object_r:public_content_t:s0 images
unconfined_u:object_r:public_content_t:s0 kde-unstable
unconfined_u:object_r:rsync_tmp_t:s0 lastsync
unconfined_u:object_r:rsync_tmp_t:s0 lastupdate
unconfined_u:object_r:public_content_t:s0 multilib
unconfined_u:object_r:public_content_t:s0 multilib-staging
unconfined_u:object_r:public_content_t:s0 multilib-testing
unconfined_u:object_r:public_content_t:s0 pool
unconfined_u:object_r:public_content_t:s0 staging
unconfined_u:object_r:public_content_t:s0 testing
```

此時有被更新過檔案的 selinux 標籤就變成了 `rsync_tmp_t` 了

### 小結
於是我們可以確定：

如果給 rsync 用的暫存目錄 ( `--temp-dir` ) 放在 selinux context 設定路徑以外的地方，就會使得 rsync 同步完後的 context 變成預期外的內容，進而造成 selinux 的權限錯誤。

### 解決方案
1. 把 temp dir 的 context 設定成和同步目的地一樣的內容
2. 不要設定 `--temp-dir` 參數 (這樣暫存檔的位置就會同步目的地一樣的目錄底下)


## 總結

雖然 SELinux 對於系統管理者在資源的運用上給予了相當多綁手綁腳的限制，但釐清相關運作流程的觀念與細節之後，就可以讓 SELinux 成為使得 Linux 系統服務安全的重要夥伴。這次在 rsync 同步雖然也踩到這個一開始令人困惑的問題，但同樣也藉此進一步地理解了 rsync 相關運作流程和 SELinux 觀念，給予了我們不小啟發。

[1]: <http://linux.vbird.org/linux_basic/0440processcontrol.php#selinux>

[2]: <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/index>

[3]: <https://rsync.samba.org/how-rsync-works.html>

[4]: <https://man7.org/linux/man-pages/man1/rsync.1.html>

[5]: <https://gitlab.archlinux.org/archlinux/infrastructure/-/blob/master/roles/syncrepo/files/syncrepo-template.sh>
