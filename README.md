# Welcome!

<img src="images/photo.jpg" alt="I don't know what it is" height="50%"/>

## 測試過的

* [sac_debian_packager](https://github.com/sean0921/sac_debian_packager): 幫你自動將 SAC ([Seismic Analysis Code](http://ds.iris.edu/ds/nodes/dmc/software/downloads/sac/)) 編譯完成，並使用 Debian 套件管理指令打包，使你的 Ubuntu 等 Linux 發行版可以超乾淨的安裝/解除安裝 SAC 程式。別再把 SAC 裝到 `/usr/local` 底下還要自己想辦法 `source` 了！

* [timeseries_process](https://github.com/sean0921/timeseries_process): 把 Fortran 古董用 [**GNU Build System**](https://en.wikipedia.org/wiki/GNU_Build_System) 打包並用 `gfortran` 開源編譯器重新編譯成跨平台程式。以研究之名研究如何使用 GNU Autotools 這個還不錯用的工具，雖然 CMake 感覺也很厲害(之後再說)，以及如何在 Linux 下輕鬆編譯你們常在 Windows 上看到的「綠色軟體」。

* [hello_gmt_docker](https://github.com/sean0921/hello_gmt_docker): 利用 Debian GNU/Linux 建立不同 GMT 版本的腳本執行環境。

* [TEQC shell script](https://github.com/sean0921/teqc_sh_script): 在 POSIX-Compatible Shell 下，執行 TEQC 轉檔指令，通常拿來轉 GPS 資料。

* [dotfile](https://github.com/sean0921/dotfile): 在 Unix-like 環境下的個人化設定腳本，包含 vim、bash、zsh、libinput-gesture (筆電觸控板神器)。

## 隨便寫的

* [布穀鳥](articles/cuckoo): 應有文學性的暴力兇宅要求寫的文章。

* [OpenSSH on Win32](articles/openssh_win32.md): 在 Windows 7 或是 10 上使用遠端安全連線指令 `ssh` 與架設 OpenSSH 伺服器

* [口罩預購與健保卡讀卡機程式使用心得](articles/nhiicc_exp_note.md): 記錄自己在開源作業系統上實測讀卡機驅動是否能運作的心得，以健保卡讀卡機元件在口罩預購上的應用為例。

* FreeBSD Jail: 不想只會跟風用 Docker 嗎? 你有另一個明智的選擇：FreeBSD Jail，它是比 Docker 更悠久，更有趣的容器化(Containerization)解決方案，除了能在 FreeBSD、FreeNAS 等 BSD-like 系統安裝不同版本的子系統外，甚至還能在上面安裝 Debian 9、CentOS 6~7 等 Linux 環境喔！ 當然也能跟後來 Linux 發行板跟風引進的 ZFS 完美結合呢！ 是個只要 \*BSD 系統就必須體驗的一大有趣賣點喔～ (相關紀錄敬請期待!)
