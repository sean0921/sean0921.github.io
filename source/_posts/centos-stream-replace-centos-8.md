---
title: CentOS 再見，但，CentOS Stream 真有那麼不堪嗎?
tags:
  - operating-system
  - linux
  - redhat
  - centos
  - centos-stream
  - rhel
date: 2020-12-10 01:04:27
---


自從 12/8 CentOS blog 和 Mailing list 相繼宣佈將要轉移開發重心到 CentOS Stream 上，並計劃要讓 CentOS 8.x (有版號的 release) 提前退休後

https://blog.centos.org/2020/12/future-is-centos-stream/
https://lists.centos.org/pipermail/centos-announce/2020-December/048208.html

甚至出現[報導](https://www.cyberciti.biz/linux-news/centos-linux-8-will-end-in-2021-and-shifts-focus-to-centos-stream/)指出：CentOS Stream 不會比 RHEL 早拿到安全更新，又要幫 RHEL 先測試新套件，同時因為 CentOS Stream 的環境比較新，你在 CentOS Stream 上面構建的套件還無法移植回 RHEL 上

但我認為這個論點有一些不充分甚至是誤導的地方。

<!--more-->

## 基本觀念

在講下去之前，先記住兩件事實：

1. RHEL/CentOS/CentOS Stream 在同一個大版號 (major release, 6/7/8) 內的 ABI/API 是保持穩定不變的 [^1]
2. CentOS 的安全性更新本來就是晚於 RHEL 的，且相較其他主流發行版晚很多

## 討論

> * How will CVEs be handled in CentOS Stream?
> ...
> In other words, CentOS Streams users will test RHEL ahead of everyone and
> report bugs, but they won’ t get security updates till resolved in RHEL. Very
> tricky situation.

這邊沒提到的是，傳統的 CentOS 本來就是這樣:
* https://lists.centos.org/pipermail/centos-announce/2020-November/035873.html
* https://lists.centos.org/pipermail/centos-announce/2020-November/035814.html

以這幾篇為例，傳統的 CentOS 本來就要等上一個禮拜（年代久遠的 release 甚至要到一個月）以上的時間才會接到來自 RHEL 的 Security patch，改成 CentOS Stream 之後不但不會比較慢，也有機會在更短的時間內接到 patch

> * Does this mean that CentOS Stream is the RHEL BETA test platform now?
> ...
> There is no option if you use CentOS for CI
> because you couldn't use RHEL developer licenses.
> Also, note that CentOS Stream will have different ABI/API at times,
> so you can no longer test or build EPEL packages locally.

但 FAQ 其他地方已經講明了 *CentOS Stream is focused on the next RHEL minor release*.

然後 RHEL/CentOS 的 minor release 就是擺明不會變這些東西，想要用新套件/新東西，就是抓 software collection、module stream、epel、第三方repo 來用，不會也不應該因為 `dnf update`/`yum update` 之後一切都炸了

所以 *no longer test or build EPEL packages locally* 根本就偏離事實，再說如果真的是「packaging」的話，RHEL 和 CentOS 的套件相依性本來就完全對不起來了，換成 CentOS Stream 不會因此有什麼改變

## 社群反應

但現在的風向都一面倒認為不應該限制別人用 RHEL 吃剩的 CentOS (8.x/9.x...)，連當初 CentOS 的 Founder 都說要來 fork 出 Rocky Linux

好吧，我懷疑我的想法太天真了

但過往的 CentOS 的社群風氣、使用者/貢獻者比例，相對於其他發行版真的差太多了，RHEL 8 出來時[追 CentOS 8 開發進度](https://www.centos.org/forums/viewtopic.php?f=10&t=71468)時才在 [mailing list](https://lists.centos.org/pipermail/centos-devel/2019-September/017694.html) 發現維運這個專案的工作人力只有 3 個人，不做出這樣策略性的改變，懷疑 CentOS 發行版還能撐多久? (IBM/RedHat 也不可能無限制地投入資源)，所以即使 Rocky Linux 真的出來了，我也懷疑這樣的動力能維持多久 (或許它能夠號召到更多有同樣需求的贊助商吧)


## 參考資源

[^1]: https://access.redhat.com/articles/rhel8-abi-compatibility#Scope
