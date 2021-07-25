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

[^1] http://linux.vbird.org/linux_basic/0440processcontrol.php#selinux
[^2] https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/index
