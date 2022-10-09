---
title: Minecraft 改存檔名稱，請留意一下 level.dat...
tags: []
id: '102'
categories:
  - - Minecraft
  - - 隨筆
comments: false
date: 2022-07-31 01:30:26
---

會有這問題，主要是因為我經營的伺服器所使用的 [MCDiscordChat](https://github.com/Xujiayao/MCDiscordChat) 在我更改存檔資料夾的名稱之後經常噴錯：

```log4j
[01:13:37] [Timer-2/ERROR]: Uncaught exception in thread "Timer-2"
java.io.UncheckedIOException: RealmsMain/stats
    at org.apache.commons.io.FileUtils.listFiles(FileUtils.java:2153) ~[commons-io-2.11.0.jar:?]
    at top.xujiayao.mcdiscordchat.utils.Utils$2.run(Utils.java:255) ~[MCDiscordChat-1.19-2.0.0-alpha.5.jar:?]
    at java.util.TimerThread.mainLoop(Unknown Source) ~[?:?]
    at java.util.TimerThread.run(Unknown Source) ~[?:?]
Caused by: java.nio.file.NoSuchFileException: RealmsMain/stats
    at sun.nio.fs.UnixException.translateToIOException(Unknown Source) ~[?:?]
    at sun.nio.fs.UnixException.rethrowAsIOException(Unknown Source) ~[?:?]
    at sun.nio.fs.UnixException.rethrowAsIOException(Unknown Source) ~[?:?]
    at sun.nio.fs.UnixFileAttributeViews$Basic.readAttributes(Unknown Source) ~[?:?]
    at sun.nio.fs.UnixFileSystemProvider.readAttributes(Unknown Source) ~[?:?]
    at sun.nio.fs.LinuxFileSystemProvider.readAttributes(Unknown Source) ~[?:?]
    at java.nio.file.Files.readAttributes(Unknown Source) ~[?:?]
    at java.nio.file.FileTreeWalker.getAttributes(Unknown Source) ~[?:?]
    at java.nio.file.FileTreeWalker.visit(Unknown Source) ~[?:?]
    at java.nio.file.FileTreeWalker.walk(Unknown Source) ~[?:?]
    at java.nio.file.FileTreeIterator.<init>(Unknown Source) ~[?:?]
    at java.nio.file.Files.walk(Unknown Source) ~[?:?]
    at org.apache.commons.io.file.PathUtils.walk(PathUtils.java:1044) ~[commons-io-2.11.0.jar:?]
    at org.apache.commons.io.FileUtils.streamFiles(FileUtils.java:2971) ~[commons-io-2.11.0.jar:?]
    at org.apache.commons.io.FileUtils.listFiles(FileUtils.java:2151) ~[commons-io-2.11.0.jar:?]
    ... 3 more
```

雖然不影響伺服器啟動過程，但有點礙眼。

奇怪，我明明就把存檔名稱從 `RealmsMain` 改叫做 `Main` 了，怎麼還跑去找 `RealmsMain`？

根據 Exception 訊息，跑去看 `top.xujiayao.mcdiscordchat.utils.Utils` 的第 255 行左右的程式碼，內容是這樣的：

```java
String topic = TEXTS.onlineChannelTopic()
                        .replace("%onlinePlayerCount%", Integer.toString(SERVER.getPlayerManager().getPlayerList().size()))
                        .replace("%maxPlayerCount%", Integer.toString(SERVER.getPlayerManager().getMaxPlayerCount()))
                        //#if MC >= 11600
                        .replace("%uniquePlayerCount%", Integer.toString(FileUtils.listFiles(new File((SERVER.getSaveProperties().getLevelName() + "/stats/")), null, false).size()))
                        //#else
                        //$$ .replace("%uniquePlayerCount%", Integer.toString(FileUtils.listFiles(new File((SERVER.getLevelName() + "/stats/")), null, false).size()))
                        //#endif
                        .replace("%serverStartedTime%", SERVER_STARTED_TIME)
                        .replace("%lastUpdateTime%", Long.toString(Instant.now().getEpochSecond()));
```

其中第 255 行為：

```java
Integer.toString(FileUtils.listFiles(new File((SERVER.getSaveProperties().getLevelName() + "/stats/")), null, false).size()))
```

`SERVER` 是 `MinecraftServer` 類別的實體，所以可以斷言 `getSaveProperties()` 以及其底下的 `getLevelName()` 都是原生 Minecraft 的 API，應該不是 Mod 作者的鍋。

後來直接往 NBT 資訊有存檔名稱的方向猜，打開 level.dat，果然還真的猜對了...

[![](/2022/07/螢幕快照-2022-07-31-00-34-23.png)](/2022/07/螢幕快照-2022-07-31-00-34-23.png)

停機改掉之後再開啟，就沒再跳錯誤了。