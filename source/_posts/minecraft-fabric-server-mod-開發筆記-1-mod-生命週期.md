---
title: Minecraft Fabric Server Mod 開發筆記 - (1) Mod 生命週期
tags: []
id: '95'
categories:
  - - Minecraft
comments: false
date: 2022-07-28 14:15:19
---

Fabric API 新版 Lifecycle 的幾個比較重要的生命週期（依照 `println()` （Kotlin) 印出結果來記錄）
<!-- more -->
*   `ServerLifecycleEvents.SERVER_STARTING`:
    *   發生在 "Starting minecraft server version xxx" 之前，在 Fabric Loader 載入之後
    *   官方在相關 class 檔案內的註解為在 `PlayerManager` 載入之前 (`PlayerManager` 用來載入 OP、黑白名單等玩家相關名單)
    *   猜測適用於自訂方塊載入、可能對存檔內容初始化有影響的內容
*   `ServerLifecycleEvents.SERVER_STARTED`:
    *   發生在 "Done (x.xxxs)! For help, type "help""之後
    *   官方表示此時存檔已經載入完畢
    *   猜測適用於管理玩家、存檔後續設定、網路相關服務的啟動等內容
*   `ServerLifecycleEvents.SERVER_STOPPING`:
    *   在觸發 `/stop` 指令後的瞬間發生
    *   官方表示此時網路連線並未中斷，存檔、玩家資料等仍可以進行操作
    *   猜測適用於停止網路相關服務、針對伺服器停止時玩家的行為等內容
*   `ServerLifecycleEvents.SERVER_STOPPED`:
    *   發生於存檔已經儲存完畢之時
    *   官方表示此時存檔、玩家資料等已經無法再存取
    *   猜測適用於非常底層的卸載動作
*   `(不在任何生命週期內)`
    *   發生於 datafixer 載入後的瞬間
    *   目前我還不知道這週期可以做什麼

`onInitialize()` 的程式碼：

```kotlin
override fun onInitialize() {
        ServerLifecycleEvents.SERVER_STARTING.register { server: MinecraftServer ->
            println("Happened on server starting")
        }
        ServerLifecycleEvents.SERVER_STARTED.register { server: MinecraftServer ->
            println("Happened on server started")
        }
        ServerLifecycleEvents.SERVER_STOPPING.register { server: MinecraftServer ->
            println("Happened on server stopping")
        }
        ServerLifecycleEvents.SERVER_STOPPED.register { server: MinecraftServer ->
            println("Happened on server stopped")
        }

        println("Hello world!")
    }
```

輸出結果:

```log4j
[14:00:14] [main/INFO] (FabricLoader/GameProvider) Loading Minecraft 1.19 with Fabric Loader 0.14.8
[14:00:14] [main/INFO] (FabricLoader) Loading 37 mods:
    - dimmanager 0.1.0
    - fabric 0.58.0+1.19
    - fabric-api-base 0.4.9+e62f51a3a9
    - fabric-api-lookup-api-v1 1.6.7+9ff28f40a9
    - fabric-biome-api-v1 9.0.14+b2a4a624a9
    - fabric-command-api-v2 2.1.2+0d55f585a9
    - fabric-content-registries-v0 3.2.1+07df213ea9
    - fabric-convention-tags-v1 1.0.8+37622d24a9
    - fabric-crash-report-info-v1 0.2.3+bd0a0d4aa9
    - fabric-data-generation-api-v1 5.1.2+a680b9b4a9
    - fabric-dimensions-v1 2.1.28+a6d2f785a9
    - fabric-entity-events-v1 1.4.16+9ff28f40a9
    - fabric-events-interaction-v0 0.4.26+9ff28f40a9
    - fabric-game-rule-api-v1 1.0.19+18990361a9
    - fabric-gametest-api-v1 1.0.33+e62f51a3a9
    - fabric-item-api-v1 1.5.5+35a03c43a9
    - fabric-item-groups-v0 0.3.26+9ff28f40a9
    - fabric-language-kotlin 1.8.2+kotlin.1.7.10
    - fabric-lifecycle-events-v1 2.1.0+33fbc738a9
    - fabric-loot-api-v2 1.1.1+03a4e568a9
    - fabric-message-api-v1 1.0.1+513f4a59a9
    - fabric-mining-level-api-v1 2.1.11+33fbc738a9
    - fabric-networking-api-v1 1.1.0+442de8b8a9
    - fabric-object-builder-api-v1 4.0.8+9ff28f40a9
    - fabric-particles-v1 1.0.8+dc39553aa9
    - fabric-registry-sync-v0 0.9.18+23c4cfefa9
    - fabric-rendering-data-attachment-v1 0.3.12+9ff28f40a9
    - fabric-rendering-fluids-v1 3.0.5+9ff28f40a9
    - fabric-resource-conditions-api-v1 2.0.9+e62f51a3a9
    - fabric-resource-loader-v0 0.5.6+5f1a85e0a9
    - fabric-screen-handler-api-v1 1.2.7+9ff28f40a9
    - fabric-transfer-api-v1 2.0.9+e62f51a3a9
    - fabric-transitive-access-wideners-v1 1.1.1+9e7660c6a9
    - fabricloader 0.14.8
    - fantasy 0.4.5+1.19
    - java 17
    - minecraft 1.19
[14:00:14] [main/INFO] (FabricLoader/Mixin) SpongePowered MIXIN Subsystem Version=0.8.5 Source=file:/home/nesquate/.gradle/caches/modules-2/files-2.1/net.fabricmc/sponge-mixin/0.11.4+mixin.0.8.5/c1dc27696aa4006e492c2485c9ccbcb26045a971/sponge-mixin-0.11.4+mixin.0.8.5.jar Service=Knot/Fabric Env=SERVER
[14:00:15] [main/INFO] (FabricLoader/Mixin) Loaded Fabric development mappings for mixin remapper!
[14:00:15] [main/INFO] (FabricLoader/Mixin) Compatibility level set to JAVA_16
[14:00:15] [main/INFO] (FabricLoader/Mixin) Compatibility level set to JAVA_17
[14:00:16] [main/WARN] (FileUtil) Configuration conflict: there is more than one oshi.properties file on the classpath
[14:00:16] [main/WARN] (FileUtil) Configuration conflict: there is more than one oshi.architecture.properties file on the classpath
[14:00:19] [main/INFO] (Minecraft) Building unoptimized datafixer
[14:00:22] [main/INFO] (Minecraft) [STDOUT]: Hello world!
[14:00:22] [main/INFO] (Minecraft) Environment: authHost='https://authserver.mojang.com', accountsHost='https://api.mojang.com', sessionHost='https://sessionserver.mojang.com', servicesHost='https://api.minecraftservices.com', name='PROD'
[14:00:24] [main/INFO] (Minecraft) Loaded 7 recipes
[14:00:25] [main/INFO] (Minecraft) Loaded 1179 advancements
[14:00:26] [main/INFO] (BiomeModificationImpl) Applied 0 biome modifications to 0 of 63 new biomes in 3.226 ms
[14:00:26] [Server thread/INFO] (Minecraft) [STDOUT]: Happened on server starting
[14:00:26] [Server thread/INFO] (Minecraft) Starting minecraft server version 1.19
[14:00:26] [Server thread/INFO] (Minecraft) Loading properties
[14:00:26] [Server thread/INFO] (Minecraft) Default game type: SURVIVAL
[14:00:26] [Server thread/INFO] (Minecraft) Generating keypair
[14:00:26] [Server thread/INFO] (Minecraft) Starting Minecraft server on *:25565
[14:00:26] [Server thread/INFO] (Minecraft) Using epoll channel type
[14:00:26] [Server thread/INFO] (Minecraft) Preparing level "world"
[14:00:26] [Server thread/INFO] (Minecraft) Preparing start region for dimension minecraft:overworld
[14:00:30] [Worker-Main-1/INFO] (Minecraft) Preparing spawn area: 0%
[14:00:30] [Worker-Main-5/INFO] (Minecraft) Preparing spawn area: 0%
[14:00:30] [Worker-Main-5/INFO] (Minecraft) Preparing spawn area: 0%
[14:00:30] [Worker-Main-1/INFO] (Minecraft) Preparing spawn area: 0%
[14:00:30] [Worker-Main-4/INFO] (Minecraft) Preparing spawn area: 0%
[14:00:30] [Worker-Main-1/INFO] (Minecraft) Preparing spawn area: 0%
[14:00:30] [Worker-Main-1/INFO] (Minecraft) Preparing spawn area: 0%
[14:00:30] [Worker-Main-1/INFO] (Minecraft) Preparing spawn area: 0%
[14:00:30] [Worker-Main-3/INFO] (Minecraft) Preparing spawn area: 31%
[14:00:31] [Worker-Main-4/INFO] (Minecraft) Preparing spawn area: 93%
[14:00:31] [Server thread/INFO] (Minecraft) Time elapsed: 4557 ms
[14:00:31] [Server thread/INFO] (Minecraft) Done (4.925s)! For help, type "help"
[14:00:31] [Server thread/INFO] (Minecraft) [STDOUT]: Happened on server started
stop
[14:08:03] [Server thread/INFO] (Minecraft) Stopping the server
[14:08:03] [Server thread/INFO] (Minecraft) [STDOUT]: Happened on server stopping
[14:08:03] [Server thread/INFO] (Minecraft) Stopping server
[14:08:03] [Server thread/INFO] (Minecraft) Saving players
[14:08:03] [Server thread/INFO] (Minecraft) Saving worlds
[14:08:04] [Server thread/INFO] (Minecraft) Saving chunks for level 'ServerLevel[world]'/minecraft:overworld
[14:08:05] [Server thread/INFO] (Minecraft) Saving chunks for level 'ServerLevel[world]'/minecraft:the_end
[14:08:05] [Server thread/INFO] (Minecraft) Saving chunks for level 'ServerLevel[world]'/minecraft:the_nether
[14:08:05] [Server thread/INFO] (Minecraft) ThreadedAnvilChunkStorage (world): All chunks are saved
[14:08:05] [Server thread/INFO] (Minecraft) ThreadedAnvilChunkStorage (DIM1): All chunks are saved
[14:08:05] [Server thread/INFO] (Minecraft) ThreadedAnvilChunkStorage (DIM-1): All chunks are saved
[14:08:05] [Server thread/INFO] (Minecraft) ThreadedAnvilChunkStorage: All dimensions are saved
[14:08:05] [Server thread/INFO] (Minecraft) [STDOUT]: Happened on server stopped
```

了解這些生命週期可以讓我順利取得 Server 的實體，以便進行後續的 Coding。