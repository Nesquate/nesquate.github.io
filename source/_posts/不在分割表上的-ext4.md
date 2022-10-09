---
title: 不在分割表上的 ext4
tags: []
id: '117'
categories:
  - - 隨筆
date: 2022-08-21 20:07:13
---

我已經忘記我當初是怎麼分割資料碟了，只知道是跟 EndeavourOS 安裝的時候一起格式化。

最近因為遊戲需求逐漸增加而把 Windows 灌回自己的 Laptop，把兩顆硬碟都分一半，結果在 Windows 操作磁碟工具時提示「磁碟初始化」的訊息。

因為自己不記得自己曾經幹了啥事，所以直接回到 Linux 去看發生什麼事。

首先是 GParted，發現怎麼縮小分割，最後都會變成最大（嘗試 e2fsck 後都會跑 resize2fs，超怪），然後就跑去 Google 一下 ext4 without partition table 的狀況，找到這篇 [StackExchange](https://unix.stackexchange.com/questions/346826/is-it-ok-to-mkfs-without-partition-number "StackExchange")，裡面的最佳解答如下：

> Creating a filesystem on a whole disk rather than a partition is possible, but unusual. The documentation only explicitly mentions the partition because that's the most usual case (it does say usually). You can create a filesystem on anything that acts sufficiently like a fixed-size file, i.e. something where if you write data at a certain location and read back from the same location then you get back the same data. This includes whole disks, disk partitions, and other kinds of block devices, as well as regular files (disk images).
> 
> After doing mkfs.fat -n A /dev/sdb, you no longer have a partition on that disk. Beware that the kernel still thinks that the disk has a partition, because it keeps the partition table cached in memory. But you shouldn't try to use /dev/sdb1 anymore, since it no longer exists; writing to it would corrupt the filesystem you created on /dev/sdb since /dev/sdb1 is a part of /dev/sdb (everything except a few hundred bytes at the beginning). Run the command partprobe as root to tell the kernel to re-read the partition table.
> 
> While creating a filesystem on a whole disk is possible, I don't recommend it. Some operating systems may have problems with it (I think Windows would cope but some devices such as cameras might not), and you lose the possibility of creating other partitions. See also [The merits of a partitionless filesystem](https://unix.stackexchange.com/questions/14010/the-merits-of-a-partitionless-filesystem)

再點進去解答者提供的連結，看了一下，原來 Oracle VM 也這樣幹...。

後來去 `lsblk` 看一下，`sda` 後面沒號碼是直接整個掛起來的：

[![](/2022/08/Screenshot_20220821_200036.png)](/2022/08/Screenshot_20220821_200036.png)

然後再去 `/etc/fstab` 看一下，因為我當初直接用 GNOME 提供的磁碟工具來掛載，不知道裡面寫了啥，所以要檢查一下：

[![](/2022/08/Screenshot_20220821_200154.png)](/2022/08/Screenshot_20220821_200154.png)

好吧，當初用 UUID 的方式掛起來。不過我對我當初怎麼分割成這樣已經完全沒有任何印象了 lol