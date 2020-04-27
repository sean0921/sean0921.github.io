# Welcome!

<img src="images/photo.jpg" alt="I don't know what it is" height="50%"/>

## 測試過的

* [bbslist](https://bbslist.github.io): 中文電子佈告欄系統列表整合，主要工作是從各種奇怪的網站與 PCMan 保留下來的列表一個一個測試整理而成，靈感是來自 Ptt.cc 的大學(uni-talk)板。

* [bbsdocker](https://github.com/bbsdocker): BBS 過時了? 用 Docker 架 BBS 正夯，當然也可以參考裡面的指令自己弄一個電子佈告欄系統，只是這個專案把它自動化了。

* [timeseries_process](https://github.com/sean0921/timeseries_process): 把 Fortran 古董用 [**GNU Build System**](https://en.wikipedia.org/wiki/GNU_Build_System) 打包並用 `gfortran` 開源編譯器重新編譯成跨平台程式。以研究之名研究如何使用 GNU Autotools 這個還不錯用的工具，雖然 CMake 感覺也很厲害(之後再說)，以及如何在 Linux 下輕鬆編譯你們常在 Windows 上看到的「綠色軟體」。

* [PyGG](https://github.com/sean0921/PyGG): 希望可以用圖形化界面(Gtk+函式庫、利用 PyGObject )產生 GMT 繪圖腳本，進一步是利用 GMT 的 Python API 來實現。(尚未實現)

* [hello_gmt_docker](https://github.com/sean0921/hello_gmt_docker): 利用 Debian GNU/Linux 建立不同 GMT 版本的腳本執行環境。

* [gtk3-testing](https://github.com/sean0921/gtk3-testing): 大大都用 Qt 或 Visual C++ 在 Windows 開發 C++ 圖形界面，所以玩玩看 Gtk+ v3.x。

* [Earth Note](https://earthnote.github.io): 嗯...就相關筆記。

* [TEQC shell script](https://github.com/sean0921/teqc_sh_script): 在 POSIX-Compatible Shell 下，執行 TEQC 轉檔指令，通常拿來轉 GPS 資料。

* [docker-gipsy-centos7](https://github.com/sean0921/docker-gipsy-centos7): 在 Docker 底下裝 Gipsy (一種 GPS 解算軟體)，你覺得 CentOS 很棒，我覺得又老套件又少，那就包進 Docker 吧！

* [ctrain](https://github.com/sean0921/ctrain) : 就...你們也會的。

* [dotfile](https://github.com/sean0921/dotfile): 在 Unix-like 環境下的個人化設定腳本，包含 vim、bash、zsh、libinput-gesture (筆電觸控板神器)。

* **\*\*NEW!!\*\*** [FFmpeg-static](http://myweb.ncku.edu.tw/~c44046040/ffmpeg-static/): 跨平台影片抓取轉檔神器，學會用它和 [youtube-dl](https://yt-dl.org) 從此抓影片不求人，也不需要冒風險去使用不明第三方惡意網站！

* FreeBSD Jail: 不想只會跟風用 Docker 嗎? 你有另一個明智的選擇：FreeBSD Jail，它是比 Docker 更悠久，更有趣的容器化(Containerization)解決方案，除了能在 FreeBSD、FreeNAS 等 BSD-like 系統安裝不同版本的子系統外，甚至還能在上面安裝 Debian 9、CentOS 6~7 等 Linux 環境喔！ 當然也能跟後來 Linux 發行板跟風引進的 ZFS 完美結合呢！ 是個只要 \*BSD 系統就必須體驗的一大有趣賣點喔～ (相關紀錄敬請期待!)

## 隨便寫的

* [布穀鳥](articles/cuckoo): 應有文學性的暴力兇宅要求寫的文章。

* [OpenSSH on Win32](articles/openssh_win32.md): 在 Windows 7 或是 10 上使用遠端安全連線指令 `ssh` 與架設 OpenSSH 伺服器
