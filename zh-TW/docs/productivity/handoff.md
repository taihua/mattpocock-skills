## 它做什麼

`handoff` 把你所在的對話壓縮成一份**交接文件**：一個 markdown 檔，寫到你作業系統的暫存目錄而不是工作區中，讓一個全新的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 可以讀取它來接手工作。

它給你的是**可攜性**，而不是壓縮，所以這個 skill 的用途比聽起來更狹窄。只有在工作必須*移動*時你才需要一個檔案：移到一個新的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)、一個新的目錄、一位同事，或一個你想分岔出去的支線任務。如果沒有任何東西要移動，你就不需要交接。留在 [session](https://www.aihero.dev/ai-coding-dictionary/session) 中、`/clear`、[子代理（subagent）](https://www.aihero.dev/ai-coding-dictionary/subagent)與 `/compact` 就能涵蓋一般的階段結束情況，而且 `/compact` 涵蓋的次數比這個 skill 更多。

## 何時使用

你透過輸入 `/handoff` 來觸發它；agent 不會自己去用它。傳入一段說明下一個 session 要做什麼的註記，skill 就會為那個目的撰寫文件。

觸發條件有四個，而且只有四個：

| 情況 | 為什麼需要一個檔案 |
| --- | --- |
| 換 harness（Claude → Codex） | 新的 harness 看不到舊的[情境（context）](https://www.aihero.dev/ai-coding-dictionary/context) |
| 移到一個不同的目錄或 repo | 原型目錄是常見的例子 |
| 把工作交給一位同事 | 他需要一個能讀的東西 |
| 分岔出在階段中途發現的支線任務 | 你繼續工作；由第二個 agent 接手那條支線 |

其他任何情況（同一個 harness、同一個目錄，你剛結束[拷問](https://www.aihero.dev/ai-coding-dictionary/grilling)要進入實作），都用 `/compact`。[ask-matt](https://aihero.dev/skills-ask-matt) 有一棵在階段邊界上涵蓋全部五個選項的排序決策樹。

## 分岔是人們會略過的用法

這個 skill 的描述讀起來像是 session 的接續：寫一份摘要，在這裡結束，在那裡繼續。這樣讀的話，它看起來像是一個比較差的 `/compact`，所以人們會略過它。分岔的情況才是值得知道的。你**留在你的 session 中**，把累積的情境複製一份交給第二個平行工作的 agent。

這就是經由 [prototype](https://aihero.dev/skills-prototype) 繞道時所使用的方式。你正深入一段設計對話，碰到一個只有可執行的程式碼才能解決的問題，而你不想把你建立起來的討論串花在找答案上。交接給一個原型 session，拿到答案，把答案交接回來，然後從原本的討論串引用它。工作跨過去兩次，你原本的對話保持有效，你也不需要重新解釋任何東西。

階段邊界上的五個選項中，有三個保留的是不同的東西：`/compact` 保留你的意圖，`/clear` 什麼都不保留，`/handoff` 保留工作移動的能力。

## 什麼會被帶走，什麼不會

文件帶著仍在進行中的脈絡（什麼正在進行、為什麼、接下來是什麼），加上一個**建議使用的 skill** 段落，指出下一個 agent 應該使用什麼。skill 在寫入檔案之前會遮蔽機密。

它刻意不帶走任何已經寫下來的東西。文件以路徑或 URL 引用 spec、計畫、ADR、issue、commit 與 diff，從不複製它們。這讓檔案保持小巧，也讓已確定的細節只存在一個地方，而不是兩份會逐漸分歧的副本。

## 常見問題

**Handoff 還是 compact？**
除非有東西要移動，否則用 `/compact`。待在同一個任務上就是 compact，而不是 handoff。當 harness 與目錄都不變、而你需要留在迴圈中時，階段邊界決策樹在大多數日子裡都會落在 `/compact`。`/handoff` 並不會摘要得比較好。它的優勢在於結果是一個檔案，你可以把它帶到 `/compact` 到不了的地方。

**那麼 compact、clear 與 handoff 實際上有什麼差別？**
每一個保留的東西不同。`/compact` 壓縮這個情境，並在一個新的視窗中繼續，所以你的意圖得以保存。`/clear` 清空視窗，從零開始。當你身後的一切都可以丟掉時，這是正確的，但如果不是，你就無法復原。`/handoff` 寫出一個可攜的檔案，所以工作能在移到別處後存活下來。三者都會把一份**[第一手資料（primary source）](https://www.aihero.dev/ai-coding-dictionary/primary-source)**（實際發生過的對話）變成一份**[二手資料（secondary source）](https://www.aihero.dev/ai-coding-dictionary/secondary-source)**（它的摘要）。繼續是唯一不會這樣做的選項，這就是為什麼你要最先排除它。

**我的交接檔案跑到哪裡去了？**
它會被放到暫存目錄，這是這個 skill 被回報最多的問題。skill 把它解析為 `$TMPDIR`，否則是 `/tmp`（Windows 上是 `%TEMP%`），但路徑很長。在繼續之前，請它把路徑告訴你並記下來。放在暫存目錄是刻意的，因為交接是一份在傳遞中的文件，而不是你要維護的產出物。暫存目錄也不是持久的，如下一個問題所說明。

**我的交接檔案在兩個 session 之間消失了。**
有些環境會在 session 之間清空暫存目錄（Codex 是被回報的例子），而重新開機會清空 `/private/tmp`。如果下一個 session 不會在一小時內開始，或要在不同的 harness 下開始，就在檔案寫好後立刻自己把它複製到某個持久的地方。同樣的道理也適用於文件*指向*的任何東西：如果一份交接引用了暫存目錄中的其他檔案，下一個 agent 就無法跟著過去。

**我實際上要怎麼把它交給下一個 agent？**
打開新的 session，並把它指向那個路徑：讀這個檔案，然後繼續。指向檔案，而不是把摘要貼進 shell 指令。當你把一份含有反引號或 `$(...)` 的摘要內插到 `claude "<summary>"` 中時，shell 會把它弄亂。通常的結果是無聲的截斷，而不是錯誤，所以新的 agent 拿到的是不完整的簡報，而沒有任何東西告訴你。

**這和 `/branch`、`--fork-session` 或內建的 `/handoff` 一樣嗎？**
相似，但不完全相同。`/branch` 不是這裡附帶的 skill；`/handoff` 才是標準名稱。分岔（fork）會繼承一份完全相同的情境副本；這個 skill 產出的則是針對一個明確的下一個任務、*有目標*的壓縮，並寫成一個檔案。在分岔就能做到的地方（同一台機器、同一個 harness、同一個目錄），分岔比較省事。當目的地是分岔到不了的地方時，檔案是比較好的選擇。

**什麼時候某件事應該改放在 `CLAUDE.md`？**
問問它下個月是否仍然成立。`CLAUDE.md` 是關於專案的常駐情境，不論相不相關，都會載入每一個 session。交接是關於一項進行中的工作，一旦那項工作落地就沒用了。如果你一直在重新解釋某個事實，它就屬於 `CLAUDE.md`。一個做到一半的任務則屬於交接。

**它記錄了做什麼，卻沒記錄為什麼。**
這是一個合理的批評，人們也常常提出。有兩件事有幫助。傳入參數（告訴它下一個 session 要做什麼），讓 skill 保留與*那件事*相關的推理，而不是把它壓扁。並留意那些 session 從未驗證過、卻說得很肯定的主張，例如「X 還沒建好」或「Y 已經完成了」。下一個 agent 會相信這份文件，而不會重新檢查，所以一個被寫成事實的信念，會成為之後所有事情的錯誤前提。在交出去之前讀一遍文件，並把任何你只是假設的東西降級。

**為什麼它是一個 skill，而不是一個 slash command？**
兩者都可行；它們適合不同的情況。作為 skill，它與這裡其他一切都透過同一條安裝路徑發布與更新，這讓它可以分享。阻止 agent 自己觸發它的，是它的 frontmatter，而不是機制本身。

## 它有效的跡象

- 文件只有對話的一小部分，而 spec、issue 與 diff 是以路徑與 URL 的形式出現，而不是被複製的文字。
- 你可以在沒有打開原本 session 的情況下直接閱讀它，並知道下一步要做什麼。
- 新的 agent 直接開始工作，而不是要你重新解釋設定。
- 在分岔的情況下，當你回到原本的 session 時，它沒有任何改變。
- 建議使用的 skill 段落，指名的正是你自己也會使用的那個 skill。
- 裡面沒有任何金鑰、token 或密碼。

## 它的定位

`handoff` 是一個**隨時都能使用的獨立 skill**。它在 session 之間運作，而不是在建置鏈之中。它的用途很狹窄，而且你使用它的頻率會比階段邊界上的其他四個選項更低。它最接近的鄰居是 [prototype](https://aihero.dev/skills-prototype)，因為原型存在於它自己的目錄中，而一來一回的往返正是這個 skill 的用途。當你處在一個邊界上，不確定要繼續、清空、交接、委派還是壓縮時，[ask-matt](https://aihero.dev/skills-ask-matt) 有一棵為這五者排序的決策樹，並會在整套 skill 中為你指路。
