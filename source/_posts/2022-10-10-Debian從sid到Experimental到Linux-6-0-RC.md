---
title: Debian 從 sid 到 Experimental 到 Linux 6.0 RC
date: 2022-10-10 16:16:12
tags: [Linux]
categories: 
    - 紀錄
---

為了安裝 Linux 6.0 的紀錄。

這邊只介紹官方作法，也只推薦官方作法，因為單獨安裝首先會碰到套件依賴問題。由於 Debian 需要將更新頻道切換到 `experimental` 才可以安裝最新版的 Linux Kernel 6.0，且要使用 `experimental` 頻道之前系統必須要在 `sid` 更新頻道的狀態，等於說要切換兩次更新頻道。

首先先修改 `/etc/apt/source.list`，把 `bullseye` 換成 `sid`，只要改第一個 `deb` 和 `deb-src` 就好。

```text
deb http://debian.cs.nctu.edu.tw/debian/ sid main
deb-src http://debian.cs.nctu.edu.tw/debian/ sid main
```

然後直接做系統全更新。

```shell
# apt update
# apt full-upgrade
```

讓 `apt` 把更新一次跑完後重開機，就會是 `sid` 更新頻道的狀態了。

接著再次修改 `/etc/apt/source.list`，把 `sid` 換成 `experimental`，同樣也是只要改第一個 `deb` 和 `deb-src`。

```text
deb http://debian.cs.nctu.edu.tw/debian/ experimental main
deb-src http://debian.cs.nctu.edu.tw/debian/ experimental main
```

然後安裝最新版 Linux Kernel，截至本文時間 Debian 最新版本為 Linux 6.0-rc7。

```shell
# apt update
# apt -t experimental install linux-image-6.0.0-rc7-amd64
```

`-t` 參數讓 `apt` 可以在指定的更新頻道上搜尋 or 安裝套件，雖然 Debian 官方 Wiki 上表示可以修改權重來達成以後都不需要帶入 `-t` 參數，但個人不建議，主要是因為 `experimental` 更新頻道比起 `sid` 更不穩定，隨時都會有系統爆炸的危險。

如果要搜尋到 `experimental` 更新頻道上的套件，那在下 `apt search` 的時候也要帶上 `-t` 參數。

### 參考
- [DebianExperimental (Debian Wiki)](https://wiki.debian.org/DebianExperimental)