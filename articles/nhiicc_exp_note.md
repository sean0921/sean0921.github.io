# 口罩預購與健保卡讀卡機程式使用心得

1. 我知道便利商店可以直接插卡[預購口罩](https://www.nhi.gov.tw/Content_List.aspx?n=395F52D193F3B5C7)了，只是想說之前已經買的 **EZ100PU** 的讀卡機了不用浪費，所以還是想試試看在我平常用的作業系統上能不能用，故在本篇文章分享自己的使用心得

2. 我目前使用的作業系統環境是 [ArchLinux](https://www.archlinux.org)，相關[套件](http://linux.vbird.org/linux_basic/0520rpm_and_srpm.php#intro)也都更新到最新狀態了，有興趣去瞭解的人自己應該就可以查到這個時間點相關套件的版本是什麼了

3. 在 [Linux](https://zh.wikipedia.org/wiki/Linux) 核心的作業系統上，健保卡 + EZ100PU 讀卡機驅動安裝過程，雖然沒有點幾下滑鼠就能裝好那樣方便，但在 Arch Linux 上用第三方套件管理員(例: [`yay`](https://github.com/Jguer/yay)) 打幾行指令就能完成安裝，並利用 [`systemd`](http://linux.vbird.org/linux_basic/0560daemons.php#daemon) 這管理工具來啟動服務(類似去 Windows 工作管理員把服務開起來)，來取代官方手冊裡的繁瑣且讓使用者難以在未來解除安裝的步驟，裝起來還是比預期的順手一些。

4. 反而花比較多時間在設定瀏覽器憑證那邊，一開始只有按照 [官方安裝手冊](https://cloudicweb.nhi.gov.tw/cloudic/system/SMC/Document/%E5%81%A5%E4%BF%9D%E5%8D%A1%E5%85%83%E4%BB%B6_Linux(Ubuntu)%E5%AE%89%E8%A3%9D%E6%89%8B%E5%86%8A.pdf) 建議，把 `https://localhost:7777` (或是 `https://127.0.0.1:7777`，[一樣的東西](https://zh.wikipedia.org/wiki/Localhost) ) 加到例外，並把裝健保卡套件時，裡面附的根憑證 (Root CA) 安裝到瀏覽器 (我用 Firefox) 裡面。

5. 進去官方給的測試頁面還是顯示讀不到覺得微挫折，而後憑著腦海中前人分享的心得去 [`/etc/hosts`](https://zh.wikipedia.org/zh-tw/Hosts%E6%96%87%E4%BB%B6) ( [手動設定網域對應 ip 的檔案](https://zh.wikipedia.org/zh-tw/Hosts%E6%96%87%E4%BB%B6) ，Windows 也有 `C:\WINDOWS\system32\drivers\etc\hosts` 這個東西 ) 翻翻看內容，果然健保卡元件有加一行 `127.0.0.1       iccert.nhi.gov.tw` ，也就是說我在輸入 `https://iccert.nhi.gov.tw:7777`，我會藉此連線到「自己電腦」( `127.0.0.1` ) 的 port 7777 進行加密連線，但瀏覽器基於安全考量只認網域不認 ip ，因此驗證網頁要求連線過去的 `https://iccert.nhi.gov.tw:7777` 還是被瀏覽器當作不安全的連線而擋下來。 

6. 於是我再開瀏覽器去 `https://iccert.nhi.gov.tw:7777` 把警告消掉後加到安全例外，問題就解決了

7. 安裝部分能給的建議就是廠商有心開發給 Linux 用的驅動值得鼓勵，但希望往後打包的部分也能花點心思整理，至少讓 Ubuntu 的使用者可以用 `sudo apt install ./nhiicc-20200319-1.deb` 之類的方式一次安裝+設定的步驟也好 (其他發行版就看有沒有好心人士要順手包一下吧) ，節省更多使用者安裝與設定的時間。

8. 憑證手動增加「安全例外」的感想是....手動設定開自己機器的 port 7777 的服務，還要特別跟自己說「這是安全的服務」，這樣真的沒問題嗎? 只能不負責任希望更好的解決方案吧，而且在 Windows 平臺上似乎也是這樣搞的呢。

9. 繳費部分用到郵局金融卡，發現自己有些功能還沒開不能網路繳費，好笑的是郵局網路 ATM 的讀卡機程式只有 Windows/Mac，於是只好忍痛換回 Windows 設定完郵局金融卡，再回去原本作業系統完成剩下的繳費步驟了。因此對於健保卡有提供 Linux 讀卡機元件的部分，還是心存感激的。
