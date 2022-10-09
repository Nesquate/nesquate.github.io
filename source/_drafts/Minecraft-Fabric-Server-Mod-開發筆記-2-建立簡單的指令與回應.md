---
title: Minecraft Fabric Server Mod 開發筆記 (2) - 建立簡單的指令與回應
tags: []
id: '105'
categories:
  - - uncategorized
comments: false
---

寫在開頭：Mod 開發語言使用為 Kotlin，且請先自行引入 [Brigadier](https://github.com/Mojang/brigadier)。

或許是各路高手神通廣大不需要 Wiki，[Fabric Wiki](https://fabricmc.net/wiki/doku.php) 的內容基本上很舊，雖然還是有部份內容可以參考，但還是要頻繁去看 Library 程式碼以及 JavaDocs。
<!-- more -->
### 註冊指令

Fabric API 有提供 `CommandRegistrationCallback` 新的指令相關 API 給開發者使用，可以讓創建指令方便了一點，大概。

你可以使用 `CommandRegistrationCallback.EVENT.register()` 來註冊指令，要注意的是傳入的東西必須為 Callback，在 Kotlin 可以使用 Lambda 來處理。

大致上的程式碼如下：

```kotlin
CommandRegistrationCallback.EVENT.register { dispatcher, registryAccess, environment ->

            if (environment.dedicated) {
                var dimRootCommand: LiteralCommandNode<ServerCommandSource> =
                    dispatcher.register(CommandManager.literal("dim").executes { context ->
                        println("Hello")
                        0
                    })

                // Add redirect root name
                var dmRootCommand: LiteralCommandNode<ServerCommandSource> =
                    dispatcher.register(CommandManager.literal("dm").redirect(dimRootCommand))
                var dimmanager: LiteralCommandNode<ServerCommandSource> =
                    dispatcher.register(CommandManager.literal("dimmanager").redirect(dimRootCommand))

                dispatcher.root.addChild(dimRootCommand)
                dispatcher.root.addChild(dmRootCommand)
                dispatcher.root.addChild(dimmanager)

                // Add sub-commands here
                dimRootCommand.addChild(Create().register())
                dimRootCommand.addChild(Delete().register())
            }
        }
```

(最近我在寫麥塊多維度管理的 Mod，大家可以期待一下XD)

這邊稍微解釋一下：

*   `register()` 會傳入 dispatcher (`CommandDispatcher<S>`)、registryAccess (`CommandRegistryAccess`) 和 environment (`CommandManager.RegistrationEnvironment`) ，這已經與當前 Wiki 上撰寫的內容有不小的差異，不過好佳在是自己傳入而不是要自己定義後傳入
*   可以先用 `environment.dedicated` 來判斷是不是獨立伺服器，如果去看 `CommandManager.RegistrationEnvironment` 的原始碼可以看到有 ALL、DEDICATED 和 INTEGRATED 三種。關於這部份的解釋 [Fabric Wiki](https://fabricmc.net/wiki/tutorial:side) 可以代勞
*   dispatcher 是主要操作指令樹的物件，你可以使用 `register()` 註冊一道指令。
    *   一道指令也必須傳遞 literal 進來。Fabric 官方建議使用 `CommandManager.literal()` 來處理，`CommandManager` 是 Minecraft 內建的指令管理，但好像也可以直接用 Brigadier 的 `LiteralArgumentBuilder.literal<T>()` ，但無論哪一個，都是要丟指令名字進去就是了
    *   可以直接在 `register()` 後面接 `excutes()` 來指定在指令被下之後該執行什麼操作，這邊一樣使用 Lambda。要注意的是它會傳入 context (`CommandContext`)，可以幫助你取得指令背後的各種資訊（包含來源是誰、指令參數內容等等）
    *   _回傳值的數字可以代表紅石訊號強度喔！_，如果你有要用指令方塊觸發其他紅石元件，這回傳值很重要
*   `dispatcher.register()` 會建立類別 `LiteralCommandNode<ServerCommandSource>` 的物件
    *   **如果這是你的根指令，請務必將它加入到 dispatcher.root 上面，這樣 Minecraft 才能夠認得。使用 `dispatcher.root.addChild()` 加入**
    *   你可以用該物件來繼續延伸其他子指令，在該物件上使用 `addChild()` 把子指令丟進來，**但要注意它必須丟 `LiteralCommandNode<ServerCommandSource>` 的物件進去**
*   如果你需要別名 (Alias)，可以在 `CommandManager.literal()` 後面接 `redirect()` ，並且把類別 `LiteralCommandNode<ServerCommandSource>` 的物件丟進去裡面，所以要先用 `dispatcher.register()` 產出物件

最後，關於這些程式碼的生命週期，請把它放到 `ServerLifecycleEvents` 任一週期**以外**的地方（也就是在 datafixer 載入以後就要被執行了！），否則無法註冊。

### 回傳提示訊息

還記得前面講到，`CommandManager.literal().excutes()` 其實會傳遞 `CommandContext` 的物件，我們就利用 context 來找出誰傳訊息，並且丟給玩家提示訊息吧！

在 Kotlin 當中，可以使用 `context.source` 來把來源對象的物件丟出來（如果是 Java 的話就要寫 `getSource()` 了），不過要注意的是，由於觸發指令的來源不一定是玩家（有可能是 Server console、指令方塊等），所以如果有必要的話，可以自己留意一下傳遞的 Class 是什麼。

之後就可以傳送提示訊息給玩家了，有以下傳送方式：

*   `sendChatMessage()`: 就像玩家聊天那樣的傳遞，不過這方法自 1.19 開始要求訊息簽章之後，情況變得頗複雜，因為你必須將訊息包進去`SentMessage`類別的物件內
*   `sendFeedback()`：就像玩家下指令那樣，只有自己看得到的 Hint，不過有另外一個布林值可以控制該訊息是否 OP 也要收到，這在一些重大指令需要 OP 監視時可以使用
*   `method_45068()`: _目前不確定這有什麼效果_
*   `sendError()`: 發生錯誤時的訊息，可以預期應該就是紅字訊息

以下程式碼範例，這邊不示範 `sendChatMessage()` 因為我還沒弄出來：

```kotlin
var commandUser = context.source
commandUser.sendFeedback(Text.of("Feedback message"), false)
commandUser.sendError(Text.of("Error message"))
```

呈現效果如下： [![](/2022/08/Screenshot_20220801_151444.png)](https://blog.im-nex.one/wp-content/uploads/2022/08/Screenshot_20220801_151444.png)

請注意要用 `Text.of()` 把文字包起來！

### 簡單接個參數