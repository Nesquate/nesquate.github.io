---
title: 在 Arch Linux 安裝最新版的 LINE 總整理
tags: []
id: '81'
categories:
  - - 隨筆
comments: false
date: 2022-07-13 02:19:15
---

關於 Arch Linux 怎麼灌 LINE 的步驟，我已經在 PTT 上 ([https://www.ptt.cc/bbs/Linux/M.1657093067.A.B49.html](https://www.ptt.cc/bbs/Linux/M.1657093067.A.B49.html)) 有初步分享，經由鄉民們的補充後，誕生出這篇文章。

整理出這篇就是希望以後自己在安裝 Arch Linux 的時候有點脈落，而不是缺啥補啥，搞到時間都浪費掉。
<!-- more -->
### 安裝 Wine 以及套件庫

首先必須打開 [multilib](https://wiki.archlinux.org/title/official_repositories#multilib) 軟體庫，以 root 權限用文字編輯器開啟 `/etc/pacman.conf` 之後，取消註解下列內容：

```yaml
[multilib]
Include = /etc/pacman.d/mirrorlist
```

存檔之後，下 `sudo pacman -Syu` 重新同步軟體庫資料庫，之後安裝 Wine 與 Winetricks：

```shell
sudo pacman -S wine winetricks
```

據 PTT 鄉民的回報，還需要加裝以下兩個 Library 才可以讓 LINE 的媒體功能正常運作（看圖以及影片聲音）：

```shell
sudo pacman -S lib32-openal lib32-libpulse
```

### 設定 LINE 的執行環境

一樣是據 PTT 鄉民回報，最新版的 LINE 需要 64 位元的環境 + Visual C++ 2012 Runtime 才可以運行。

首先建立一個 LINE 專屬的 Wine 執行環境

```shell
WINEPREFIX=~/.wine-line winecfg
```

並順便進入設定。過程中若有提示要安裝 Wine Mono 的話，請不要安裝，因為用不到 .NET 框架執行環境。

進入設定之後，目前只要改 Windows 執行版本為 Windows 10 就好。

![](/2022/07/未儲存圖片-2.jpg)

### 安裝 Visual C++ 2012 Runtime

這時可以請 Winetricks 代勞下載：

```shell
WINEPREFIX=~/.wine-line winetricks vctun2012
```

如果不給下載，進去 Winetricks -> 選取預設的 Wine 容器 -> 安裝 Windows DLL 或套件，句選 vctun2012 也可以。

預設它會把 32 位元和 64 位元的 Runtime 下載下來，如果有彈出安裝視窗就把它們裝好吧。

### 安裝 LINE

搞定好 Runtime 之後，接下來可以直接下載[最新版的 LINE](https://desktop.line-scdn.net/win/new/LineInst.exe)然後下以下指令：

```shell
WINEPREFIX=~/.wine-line wine /path/to/LineInst.exe
```

`/path/to/LineInst.exe` 改成 LineInst.exe 實際放置的路徑就好。

安裝結束之後，基本上你的 LINE 已經可以正常運作了。

### 故障排除

#### 切換到其他視窗時，LINE 視窗的外框還在？

Wine 的問題，左岸網友在 WeChat 上也有遇到，後來找到視窗外框的 ID 就解決了，

可以嘗試使用此方式解決看看。

可參考：  
[https://www.ptt.cc/bbs/Linux/M.1643429963.A.510.html](https://www.ptt.cc/bbs/Linux/M.1643429963.A.510.html)  
[https://zhuanlan.zhihu.com/p/106926984](https://zhuanlan.zhihu.com/p/106926984)

##### 字體顯示怪怪的

安裝 [ttf-win10](https://aur.archlinux.org/packages/ttf-win10) 可能可以解決這個問題，不過你可能要先去安裝 AUR Helper，以及安裝時會下載 Windows 10 的 ISO 檔案，請準備好 10GB 的可用磁碟空間。