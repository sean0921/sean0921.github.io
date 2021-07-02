---
title: Debian 最新版 nonfree 韌體安裝筆記
date: 2020-09-12 16:15:44
tags:
- debian
- firmware
- packaging
- salsa
- sysadmin
---

## 介紹

在使用 Debian GNU/Linux 這個發行版時，我們想要在兼顧 Debian Stable 穩定的特性下使用版本較新的套件，這時候 Backports 就是一個被官方社群支援的選項。然而有時我們還是會遇到裝了新版 backport kernel, non-free 韌體內容跟不上或找不到的小問題 (不影響系統運作)。除了 Bug Report 回報給相關社群之外，在等待的期間還可以做哪些處置來兼顧新版套件安裝與後續套件管理(apt)支援? 我們可以利用 Debian Salsa 的 GitLab 資源來協助我們參閱或修改測試中或已經釋出的打包腳本，讓 Debian 及其衍生發行版的套件資源變得更加健全。

<!--more-->

## 筆記

每次在 Debian Stable 想裝 [backports](https://backports.debian.org) 的 [kernel](https://kernel.org) (通常版本是次新釋出的穩定分支) 遇到了缺某些韌體的訊息

雖然看不出來系統會因此發生什麼問題，還是記一下更新的方式


首先要用 `apt` 先安裝 `linux-support-<kernel version>` 套件

以及: [^1]

- `git` (要從 [Debian Salsa](https://salsa.debian.org) 抓 repo)
- `build-essential`
- `fakeroot` 
- `devscripts` (利用 debhelper 建標準 Debian 套件)


然後

```bash
git clone https://salsa.debian.org/kernel-team/firmware-nonfree.git \
          /path/to/workdir/debian_firmware_nonfree/firmware-nonfree

cd /path/to/workdir/debian_firmware_nonfree/firmware-nonfree  #自己改路徑
```

再 `git clone` [kernel.org](https://kernel.org) 維護的 `linux-firmware` 專案:

```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git \
          /path/to/workdir/debian_firmware_nonfree/linux-firmware
```

進去 `/path/to/workdir/debian_firmware_nonfree/firmware-nonfree`:

```bash
PYTHONPATH=/usr/share/linux-support-<kernel version>/lib/python debian/rules \
debian/control   #自己改 kernel 版本
```

或是改 `debian/rule.defs` 確認自己的 kernel 版號對不對後，`debian/rules debian/control`
(通常不對，因為沒有 debian 後綴版號)

然後

```bash
PYTHONPATH=/usr/share/linux-support-<kernel version>/lib/python \
            debian/bin/release-update \
            /path/to/workdir/debian_firmware_nonfree/linux-firmware
```

然後忘記會不會幫你解包了

之後就把 `debian/` 目錄放到解好的地方 或是在有放 `debian/` 的目錄解包解過去

然後就照常

```bash
mk-build-deps debian/control  ## 通常東西都裝完了，不會裝新的東西
debuild -b -us -uc  ## 就叫 debhelper 系列的工具幫你把套件生出來
```

然後再往上層目錄找包好的套件

然後再選你要裝的 linux 韌體 sudo apt install ./<套件名稱>.deb

完工！ 重開機


通常應該不會出事.... 吧...


## 結論

Debian Salsa 和 Debian backports 都是很好用的東西，嫌 Debian 太舊的人都該先試試這些東西並且嘗試 Bug Report 看 Maintainer 態度再來評論好壞。

### Note


說到這個想到有時在裝完 backport kernel 時且把 `quiet splash` 關掉之後

就會在 systemd 訊息看到 `networking.service` 跟 [`NetworkManager.service`](https://en.wikipedia.org/wiki/NetworkManager) 打架的訊息
(`networking.service` 本質上還是 [SysV init](https://en.wikipedia.org/wiki/Init) 的東西只是 [systemd](https://en.wikipedia.org/wiki/Systemd) 向下相容它)

這時都是選擇留 `NetworkManager` 砍 `networking` 相關的 script 套件

做完之後確定 `NetworkManager` 有在做事就搞定了


## 參考資料

[^1] https://wiki.debian.org/BuildingTutorial
