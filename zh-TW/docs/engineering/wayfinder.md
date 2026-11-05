## 它做什麼

`wayfinder` 處理一項對一個 agent [session](https://www.aihero.dev/ai-coding-dictionary/session) 來說太大的工作。這是一個你能說出**目的地（destination）**、卻還看不見路線的想法。Wayfinder 把它畫成你 issue tracker 上一張由**決策 ticket** 組成的共用**地圖（map）**，然後一次解決一張 ticket，直到路線變得清楚。

它只規劃，不建置。每一張 ticket 都問一個問題，而答案是一個決策，而不是建置的一個切片。當在有人去建造那個東西之前已經沒有任何需要決定的事時，地圖就完成了。這條規則把 wayfinder ticket 與一般的實作 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 區分開來，而它也是 agent 最常違反的規則。當地圖變得清楚時，wayfinder 會交接出去，而不會繼續寫程式碼。

## 何時使用

你透過輸入 `/wayfinder` 來觸發它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不會自己去用它。

它是這套 skill 中最重量級的流程，所以觸發條件很狹窄。這項工作必須大過一個 agent session 能容納的量，而且通往目的地的路線必須還不清楚。區分的依據是 session 數量：單一 session 的規劃用 `/grill-with-docs`，多 session 的規劃用 `/wayfinder`。

| 你眼前有什麼 | 要執行什麼 |
| --- | --- |
| 一個範圍明確、坐一次就能確定的功能 | [grill-me](https://aihero.dev/skills-grill-me)，或在有程式碼庫時用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 一個全新專案，或一個橫跨許多 session 的建置，而路線仍然不清楚 | `/wayfinder` |
| 一個決策已經做完的討論串 | [to-spec](https://aihero.dev/skills-to-spec)：直接跳過地圖 |
| 一張已經清楚的 wayfinder 地圖 | [to-spec](https://aihero.dev/skills-to-spec)，然後 [to-tickets](https://aihero.dev/skills-to-tickets) 與 [implement](https://aihero.dev/skills-implement) |
| 一個已經變得太大的既有 session | 說「hand off to `/wayfinder`（交接給 `/wayfinder`）」（[handoff](https://aihero.dev/skills-handoff) 既能把工作帶出地圖，也能帶進地圖） |

全新專案不是必要條件。人們經常在舊的與做到一半的程式碼庫上使用 wayfinder，而它在那裡可能更有用，因為大部分的迷霧是「這裡已經成立的是什麼」，而不是「我們應該做什麼」。

## 前置條件

地圖與它的 ticket 存在 repo 的 issue tracker 上，所以 wayfinder 需要 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 的 tracker 設定。那一步會寫入一個「Wayfinding operations」段落。這個段落描述如何在 GitHub、GitLab 或本地 markdown 上表達地圖、它的子 ticket、阻擋關係與前緣查詢。Wayfinder 是透過你 `CLAUDE.md` / `AGENTS.md` 中的指標找到那份文件，而不是在一個固定的路徑。如果沒有設定 tracker，它會退回使用本地 markdown 檔案。

tracker 做了真正的工作。它原生的阻擋連結會在 tracker 自己的 UI 中顯示前緣。在沒有原生依賴連結的 tracker 上（例如一個自架的 Gitea），wayfinder 會從地圖文字推斷阻擋者。那可以運作，但你必須更密切地監督它。

## 地圖、迷霧與前緣

**地圖**是單一個標上 `wayfinder:map` 的 issue，而它的 ticket 是它的子 issue。它是一份**索引，而不是一個倉庫**。一個決策只存在於一個地方，也就是它的 ticket，而地圖只給一行摘要和一個連結。session 以低解析度載入地圖，並在需要時才打開個別的 ticket。這讓地圖可以持續成長，而不需要每個 session 都載入它的完整歷史。

地圖有四個段落：

- **Destination（目的地）。** 這張地圖的終點長什麼樣子。你在任何 ticket 存在之前就先為它命名，因為目的地決定了你衡量每張 ticket 所依據的範圍。
- **Decisions so far（目前為止的決策）。** 每張已關閉的 ticket 一行，每一行都附有指向細節所在處的連結。
- **Not yet specified（尚未具體化）。** 這就是**戰爭迷霧（fog of war）**：你能感覺到即將到來、但還無法清楚表達的決策。判斷是迷霧還是 ticket 的標準，是你*現在*能不能精確地說出這個問題，而不是你能不能回答它。當你解決一張 ticket 時，它前方的迷霧就會散開，而任何你現在可以具體化的東西都會升級成一張新的 ticket。
- **Out of scope（範圍外）。** 超出目的地的工作。迷霧只會*朝向*目的地聚集，所以範圍外的工作會保持關閉，永遠不會升級。

**前緣（frontier）**是開放、未被阻擋、未被認領的 ticket 集合（已知世界的邊緣）。session 在做任何工作之前，會先把一張 ticket 指派給自己來認領它。指派者*就是*認領，所以同時進行的其他 session 會跳過那張 ticket。session 用名稱來指稱 ticket，從不用單純的 `#42`，因為一整排 issue 編號在敘述中很難閱讀。

## 四種決策 ticket 類型

每張 ticket 都帶有一個 `wayfinder:<type>` 標籤。每張 ticket 不是 **[HITL](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)**（與一位為自己發言的人一起處理），就是 **[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)**（由 agent 獨自驅動）。HITL ticket 只能透過即時的交流來解決。一個自問自答其[拷問](https://www.aihero.dev/ai-coding-dictionary/grilling)問題的 agent 就破壞了它。

| 類型 | 模式 | 何時使用 | 由什麼解決 |
| --- | --- | --- | --- |
| `grilling` | HITL | 預設。這個問題可以透過談話來解決。 | [grilling](https://aihero.dev/skills-grilling) 加上 [domain-modeling](https://aihero.dev/skills-domain-modeling)，在一個新的 session 中 |
| `prototype` | HITL | 「這應該長什麼樣子」或「這應該怎麼運作」：一個談話無法解決的問題。 | [prototype](https://aihero.dev/skills-prototype)，並從 ticket 連結到建好的產出物 |
| `research` | AFK | 一個工作目錄之外的事實卡住了某個決策。 | 一個 [research](https://aihero.dev/skills-research) [子代理（subagent）](https://www.aihero.dev/ai-coding-dictionary/subagent)，在你繪製地圖時啟動，並在一個 `research/<name>` 分支上平行執行 |
| `task` | 兩者皆可 | 沒有東西要決定，但手動工作卡住了某個決策，例如開通存取權限、註冊某個服務，或搬移資料讓你能看清它的形狀。 | 能做的地方由 agent 獨自完成，否則就給人類一份精確的檢查清單 |

`task` 是唯一一種*執行*而不是決策的類型。它之所以屬於地圖，只是因為它能解除某個決策的阻擋，而從不是因為它交付了目的地的某一部分。這種類型在實務上最常出錯。agent 會把它讀成一個實作步驟，並開始在地圖中撰寫產品程式碼。

研究是「每個 session 一張 ticket」的唯一例外。

## 常見問題

**這和 `/grill-with-docs` 有什麼不同？我應該從哪一個開始？**
看 session 數量，而不是專案大小。`/grill-with-docs` 是單一 session 的規劃；wayfinder 是多 session 的規劃。如果你能在一段對話中掌握整件事，拷問更便宜也更好，而 wayfinder 在那種情況下更慢、更密集。社群的簡略說法是，只有當工作放不進單一 session 時，wayfinder 才有意義。這是到目前為止關於 wayfinder 被問最多的問題。人們一直問，是因為 skill 的描述並沒有告訴你你自己的任務位於那條線的哪一邊。你必須自己判斷 session 數量。

**當它要求「目的地」時，指的是這個 session 的終點，還是整件事的終點？**
整張地圖的終點，而不只是第一個 session。這個問題讀起來有歧義，但 wayfinder 依定義就是一個多 session 的工具，所以一個限定在 session 範圍內的答案永遠沒有意義。典型的目的地是一份要交出去的 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)、一個在規劃開始前要鎖定的決策、一個概念驗證，或一個就地進行的變更，例如資料遷移。

**地圖清楚了。wayfinder 不是已經寫好 spec、也做好 ticket 了嗎？為什麼我還需要 `/to-spec` 與 `/to-tickets`？**
不是。Wayfinder 的 ticket 是決策 ticket，到地圖關閉時它們也全都關閉了。剩下的是一張充滿連結決策的地圖，而那不是一份建置計畫。[to-spec](https://aihero.dev/skills-to-spec) 把那些連結的決策收斂成一份 spec（`/to-spec #<map_issue>`），而 [to-tickets](https://aihero.dev/skills-to-tickets) 把那份 spec 切成曳光彈式的實作 ticket。如果你把地圖直接接到 [implement](https://aihero.dev/skills-implement)，你就跳過了收斂，並失去了連結的細節。只有在工作最後證明很小時，才直接進入實作。有些人執行較短的管線，並回報它可以運作。那兩個額外的步驟給你一份明確的 spec，讓審查者或同事可以閱讀，而這在你不是獨自工作時更重要。

**我的 agent 在 wayfinder session 進行到一半時開始寫正式程式碼。**
這是這個 skill 被回報最多的失敗，而它是由 skill 中一個真正的缺口所造成的。你可以在地圖的 **Notes** 中覆寫 wayfinder「只規劃，不執行」的預設。但寫 Notes 的是 agent，所以這個限制以及它的豁免，存在於一個由受限制的 agent 所擁有的檔案中。一位使用者看著一個 agent 在它自己的 Notes 中寫下「this map carries execution（這張地圖包含執行）」。在之後的 session 中，agent 把那一行讀回來當作許可，然後在一台正式運作的伺服器上動手建置。這個 skill 沒有針對「我指的是預設值」的硬性停止。在它加上之前，閱讀任何不是你自己繪製之地圖的 Notes、把實作留在獨立的 session 中，並把任何看起來像建置切片的 `wayfinder:task` 視為類型標錯了。

**我畫了 27 張 ticket，等我做到第十三張時，剩下的都已經沒有意義了。**
這個問題是逐字取自一位使用者的回報，而其他人也回報了同樣的結果。預設上，wayfinder 會做全面的規劃。當後面的 ticket 建立在前面 ticket 所推翻的假設之上時，地圖就會掉進批評者指控這個 skill 會掉入的瀑布式陷阱。有兩件事有幫助。第一，把地圖的範圍限定在一個有界的目的地，而不是整個產品。使用者回報，範圍限定在一個明確 epic 的地圖，表現比一個龐大的「實作 V1」更好。目標是交付小的增量，而不是規劃一個非常大的東西。第二，積極地[做原型](https://www.aihero.dev/ai-coding-dictionary/prototyping)。路線之所以能保持最新，是因為便宜的具體產出物能在實作依賴它之前就暴露出不確定性。Wayfinder 是「把原型做到極致」，而不是「把規劃做到極致」。

**我可以平行處理好幾張 ticket 嗎？**
前緣會告訴你哪些 ticket 可以接手，而阻擋關係讓平行工作在紙面上是安全的。實務上，一次一張 ticket 是比較安全的預設。如果你同時處理兩張拷問 ticket，一個 session 可能會問你一個你剛在另一個 session 中回答過的問題，因為兩個 session 不共用任何[情境](https://www.aihero.dev/ai-coding-dictionary/context)。原型 ticket 也有一個已知的缺口。一位使用者回報一個 agent 做了三個 UI 變體、自己選了一個，然後關閉了 ticket。那個選擇是你的，而這個 skill 還沒有把這一點說得夠清楚。如果你確實要平行執行 ticket，先自己審查依賴圖。

**我一定要用 GitHub Issues 嗎？**
不用。任何 issue tracker 都可以。GitHub 的支援最好，因為它原生的子 issue 與阻擋關係讓你不用打開地圖就能看到前緣。人們也使用 GitLab、Linear、Jira 與本地 markdown。有兩個注意事項。在沒有原生阻擋功能的 tracker 上，wayfinder 會從文字推斷依賴圖，而你必須手動修正它。本地 markdown 會把產出物放進你的 repo，這並不建議，因為存放在 repo 中的資料往往會意外地被保留下來。開源維護者碰到的是相反的問題（公開的 tracker 被 agent 產生的規劃 ticket 塞滿），所以他們常常還是選擇本地 markdown。

**拷問讓人精疲力盡。每個問題都有三段那麼長。**
這是對 wayfinder 最尖銳、仍未解決的抱怨，至今沒有人修正它。一位使用者這樣拆解：冗長本身就造成決策疲勞，而且長度隱藏了 agent *為什麼*要問某個問題，所以隨著地圖成長，你會失去決策與決策之間的關聯。這種冗長看起來比較像是目前這批[模型](https://www.aihero.dev/ai-coding-dictionary/model)的特性，而不是這個 skill 的特性。使用者嘗試兩種緩解方式：較低的[推理強度（reasoning effort）](https://www.aihero.dev/ai-coding-dictionary/effort)，以及在你全域 `CLAUDE.md` 中加一條使用白話的指示。無論如何都要預期在這裡要用力思考。Wayfinder 要求你大量思考，而那是它大部分的目的，而不是一個缺陷。

**一個我已經關閉的決策後來被證明是錯的。我要編輯舊的 ticket，還是開一張新的？**
沒有官方的指引，而 agent 的預設行為沒什麼幫助。它傾向於繞過那個錯誤的決策來設計，而不是質疑它，所以你必須自己引導它。有效的做法是直白地告訴 wayfinder 什麼改變了。它接著會更新地圖、修訂受影響的 ticket，並在已關閉的 ticket 上留言。你可以在地圖進行中從範圍變更中恢復過來。但如果你*設計*了一張預期會改變的地圖，那就是範圍錯了的跡象。

**`decision-mapping` 去哪了？**
就是這個 skill。v1.1 把它改名為 `wayfinder`，你以 `/wayfinder` 觸發它。「Decision map（決策地圖）」是術語，而且也不準確，因為四種 ticket 類型中只有一種本身就是決策。新名稱讓這個 skill 擁有一套一致的詞彙（目的地、戰爭迷霧、前緣、地圖），而不是在上面再加一個自創的術語。單位保留了「decision（決策）」這個詞：wayfinder ticket 是一張**決策 ticket**，這樣人們就不會把它讀成實作 ticket。

## 它有效的跡象

- 在任何一張 ticket 存在之前，目的地就已經寫下來並取得共識。
- 每一張開放的 ticket 讀起來都是一個問題。任何讀起來像「建置 X」的 ticket，要嘛是類型標錯了，要嘛屬於地圖的下游。
- 你可以看著你的 tracker，不用打開地圖就看到哪些 ticket 可以接手，因為原生的阻擋關係會顯示前緣。
- 一個 session 解決一張 ticket、把答案發布為一則解決留言、關閉它，並在地圖的 *Decisions so far* 中加一行。然後就停止。
- **Not yet specified** 會隨著時間縮小。當迷霧升級成一張 ticket 時，它就會離開那個段落，而不會同時出現在兩個地方。
- 當開頭的廣度優先拷問完全沒發現任何迷霧時，這個 skill 會停下來，告訴你這項工作小到可以跳過地圖。
- 完成地圖的那個 session 會把你指向一份 spec，而不是一個 pull request。

## 它的定位

`wayfinder` 是一條**視情況使用的匝道**，而不是預設的起點。大部分的工作仍然從以拷問帶領的「想法 → 出貨」鏈開始。當想法大到一個 session 裝不下時，你才使用 wayfinder。它在 [to-spec](https://aihero.dev/skills-to-spec) 重新加入那條鏈，因為一張已經清楚的地圖負責交接，而不負責建置。

大部分的工作發生在 wayfinder 所排程的其他 skill 中。[grilling](https://aihero.dev/skills-grilling) 與 [domain-modeling](https://aihero.dev/skills-domain-modeling) 解決預設的 ticket 類型，[prototype](https://aihero.dev/skills-prototype) 解決談話無法解決的 ticket，而 [research](https://aihero.dev/skills-research) 以子代理的形式執行，讓它的閱讀不會進入你的 session。[handoff](https://aihero.dev/skills-handoff) 負責把工作移進移出：從一段變得太大的對話移進地圖，以及在 session 中途出現支線任務時移出地圖。其他任何情況，[ask-matt](https://aihero.dev/skills-ask-matt) 都會在整套 skill 中為你指路。
