---
title: OpenSSH on Windows 7/10
date: 2020-04-27 00:00:00
tags:
- sysadmin
---

雖然新版 Windows 10 已經有預裝 OpenSSH Client 了，也可以自行啟用 OpenSSH Server，不過版本都似乎停留在 7.7

如果想要在 Win10 體驗較新版本的 OpenSSH (~8.1) ，或是想在 Win7 上也體驗 OpenSSH 的話，可以去以下載點下載官方測試版

<!--more-->

* [OpenSSH-Win32 GitHub 入口頁面](https://github.com/PowerShell/Win32-OpenSSH)
* [OpenSSH-Win32 source code](https://github.com/PowerShell/openssh-portable)
* [OpenSSH-Win32 執行檔載點](https://github.com/PowerShell/Win32-OpenSSH/releases)

首先要先去設定那邊關閉 OpenSSH 功能，這樣系統才不會先讀到 System32 那邊的 OpenSSH

## Client 安裝

把解壓縮的 OpenSSH 放在 Program Files 底下，加進系統 `PATH`

## Server 安裝

參考 [GitHub wiki 頁面](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)

收工！

# 調校

* `sshd_config` 位置： `C:\ProgramData\ssh`，可以改 ssh port，限制登入的使用者等進階安全設定
* 理論上可以改 ssh 進去的 shell (預設是 cmd)

