## 它做什麼

`improve-codebase-architecture` 勘查一個程式碼庫，尋找**加深的機會（deepening opportunities）**。這些是一個淺模組（介面幾乎和它所隱藏的東西一樣複雜）可以變成深模組的地方。它把這些機會寫成一份自成一體的 HTML 報告，然後針對你挑選的那一個[拷問（grill）](https://www.aihero.dev/ai-coding-dictionary/grilling)你。

它從不改變程式碼。整次執行產出的是一段對話，以及你作業系統暫存目錄中的一個 HTML 檔。重構由你之後在另一個 [session](https://www.aihero.dev/ai-coding-dictionary/session) 中，透過正常的建置流程來做。這讓它成為一次勘查，而不是一個重構工具，所以你可以在一個你還沒準備好要動的程式碼庫上執行它。

有兩道過濾器讓報告不會變成通用的清理建議。第一，每個候選項目都必須通過**刪除測試**：如果你移除這個模組，它的複雜度會移到一個更小的介面後面，還是會擴散到各個呼叫者？只有前者會得到一張卡片。第二，除非你把它指向某個特定區域，否則它會先讀取最近的 commit 歷史，並把掃描集中在經常變動的路徑上。在沒人碰的程式碼上做加深，是一次永遠不會回本的重構。

## 何時使用

你透過輸入 `/improve-codebase-architecture` 來觸發它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不會自己去用它。

它不是主要建置迴圈中的一步。你定期執行它，為改善程式碼庫排入更多工作。人們在四種情況下使用它：

| 情況 | 如何使用 |
| --- | --- |
| 例行維護 | 每隔幾天執行一次，或在你有空檔時執行，讓結構不會在功能之間腐壞。 |
| 大型建置之前 | 把它指向 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，然後問「我們要怎麼讓這個變更變得容易？」這是對它最有效的提示。 |
| 既有專案稽核 | 在一個大型、沒有結構或[憑感覺寫出來（vibe-coded）](https://www.aihero.dev/ai-coding-dictionary/vibe-coding)的 repo 上執行它，了解它目前的狀態。 |
| 舊程式碼的測試工作 | 在為難以測試的程式碼撰寫測試之前，用它找出缺少的接縫。 |

它容易與兄弟 skill 混淆的地方：

- 要設計一個你已經選定的模組，請使用 [codebase-design](https://aihero.dev/skills-codebase-design)。這個 skill 找出要處理的模組，而 `codebase-design` 是你設計它的地方。
- 對於一整項大到一個 session 裝不下的工作，請使用 [wayfinder](https://aihero.dev/skills-wayfinder)。
- 對於「這個特定的東西壞了」，請使用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。當真正的發現是沒有好的接縫可以把 bug 鎖住時，它會把你送回這裡。

## 前置條件

執行它不需要任何東西。如果 `GLOSSARY.md` 或 `docs/adr/` 中的 ADR 存在，它會讀取它們，並使用你領域自己的名詞。候選項目讀起來就會是「加深 Order 進件模組」，而不是「重構 FooBarHandler」。

它會寫入兩個地方。報告寫到 `<tmpdir>/architecture-review-<timestamp>.html`，在 repo 之外。在拷問迴圈中，它會在 `GLOSSARY.md` 中新增或磨利術語，如果那個檔案不存在就建立它。它也會提議把一個被否決的候選項目記錄成一份 ADR，讓未來的執行不會再建議它。

## 深度，以及尋找深度的報告

這個 skill 建立在一個概念上：**深度（depth）**。深模組在一個小而穩定的介面後面放大量的行為。淺模組則透過一個幾乎和底下程式碼一樣寬的介面暴露它的實作。報告會尋找三種形式的淺薄：

- 只是為了可測試性而抽出來的純函式，而真正的 bug 卻在呼叫者如何使用它們之中（沒有**局部性**）。
- 跨過它們的**接縫**洩漏的模組。
- 一個不打開五個檔案就無法理解的概念。

對每一個，它都會提出修正它的加深方式。

每個候選項目都是一張卡片，包含涉及的檔案、摩擦點、一個淺白英文的解法、以**局部性**與**槓桿**表達的好處、一張之前 / 之後的圖表，以及一個強度徽章。

| 徽章 | 它對你的意義 |
| --- | --- |
| `Strong` | 刪除測試明確通過，而摩擦是真實的。認真對待這些。 |
| `Worth exploring` | 看似合理的加深，但回報取決於程式碼接下來的走向。 |
| `Speculative` | 為了完整而列入。這些大部分你可以忽略。 |

報告最後是一個**首要建議（Top recommendation）**，也就是它會最先做的候選項目。然後 skill 會停下來，問你想探索哪一個候選項目。在那個時間點你什麼都還沒決定，也沒有任何程式碼改變。

## 你挑選之後會發生什麼

當你挑選一個候選項目時，就會針對它開始一場 [grilling](https://aihero.dev/skills-grilling) session。它涵蓋限制、接縫後面放什麼、哪些測試會留下來，以及加深後的介面應該長什麼樣子。那個 session 的產出是一個決策，而不是一份 diff。從那裡開始就套用正常的流程：把決策帶進 [to-spec](https://aihero.dev/skills-to-spec)，然後 [to-tickets](https://aihero.dev/skills-to-tickets)，然後 [implement](https://aihero.dev/skills-implement)。

## 常見問題

**它針對一個想法拷問了我一個小時，而不是給我看選項。我可以關掉這個嗎？**

可以。在觸發時說出來（「不要拷問我，只要給我看報告」）。這是對這個 skill 最常見的抱怨。一位使用者喜歡它是「一個方便取得完整改善分析的方式」，但在加入拷問迴圈之後，發現它「幾乎無法使用」。在他們的 session 中，它提出了單一個解法，然後問了「幾十甚至幾百個問題」。設計的本意是報告優先，而拷問只在你選定的候選項目上開始。但比較弱的[模型](https://www.aihero.dev/ai-coding-dictionary/model)會直接跳到針對它們第一個想法的訪談。那個討論串中的結果因模型而有很大的差異。這是一個開放中的 issue，而這個 skill 還沒有一個有文件記載的不拷問模式。

**報告打開時是沒有樣式的原始 HTML，也沒有圖表。發生了什麼事？**

報告從 CDN 載入 Tailwind 與 Mermaid，所以你打開它時需要網路連線。如果有東西擋住了那些腳本，頁面就會在沒有錯誤的情況下壞掉。在被回報的案例中，一個安全 hook 要求 SRI 雜湊值。agent 加上了它們，但 CDN 給瀏覽器的位元組與給計算雜湊值的 `curl` 的不同，所以瀏覽器擋住了腳本。離線與管制嚴格的環境也有同樣的問題。agent 看不到這個問題，因為它從不渲染頁面。作為變通方法，請它改用內嵌的 CSS 與手工打造的 SVG 圖表，而不是 CDN 的設定。這是一個開放中的 issue。

**它給了我十二個候選項目。我要在同一個 session 中處理它們，還是開一個新的？**

每個 session 一個候選項目。如果你在一段對話中處理好幾個，[context window](https://www.aihero.dev/ai-coding-dictionary/context-window) 會同時被報告、拷問、領域模型的編輯與程式碼變更塞滿。報告只是一個暫存檔，所以要帶著往前走的是候選項目，而不是檔案。挑一個、拷問它，然後把決策帶進 `/to-spec`。把其餘的變成之後可以分別接手的 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)。把選定的改善放進一份 spec，而不是直接進入實作。人們常常問這個問題，而 skill 本身並沒有記載任何對應的工作流程。

**我應該怎麼提示它？**

用你接下來要建的東西來提示它。如果有一個大型建置即將到來，把它指向 spec，然後問「我們要怎麼讓這個變更變得容易？」沒有提示的執行會自己去找熱點。這對例行維護來說沒問題，但給它一個方向，會讓報告變得可以付諸行動。

**它在大型的舊程式碼庫上有用嗎？**

部分有用。它在缺乏一致結構的大型既有程式碼庫上效果很好，也是任何一次性結構設定之後建議使用的維護工具。但專案已經失控的使用者回報它「幫了一點，但似乎還是不夠」。一位手上有一個八年舊程式碼庫的開發者回報，模型一直在原地打轉，而同一個 skill 在一個整潔的 repo 上卻能產出一張乾淨的圖。目前還沒有針對那種情況的專用 `/refactor` skill。如果程式碼庫沒有共用的詞彙，先執行 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 來建立一套。那通常會讓這個 skill 的產出好很多。

**這和 `/codebase-design` 有什麼不同？**

`/codebase-design` 是一份參考資料，而不是一個 session 的驅動者。它提供詞彙（module、interface、depth、seam、adapter、leverage、locality），而這個 skill 使用它。如果你把 `/codebase-design` 當作任務交給一個全新的 agent，它會以一種已知的方式失敗。它沒有自己的流程可以遵循，所以 agent 會自己發明一個、重新探索程式碼，並執行非常久才問你任何問題。執行這個 skill，讓它去使用那一個。

**它會告訴我程式碼庫沒問題嗎？**

很少，所以在開始之前要知道這一點。這個 skill 存在的目的就是產出發現，所以它傾向於產出候選項目，而不是得出「沒有任何問題」的結論。用強度徽章來修正這一點。如果一份報告中的每個候選項目都是 `Speculative`，代表 skill 什麼都沒找到。

**它在 Codex 或其他 harness 中能運作嗎？**

部分可以。探索步驟直接指名了 Claude Code 帶有 `subagent_type=Explore` 的 `Agent` 工具。一個沒有那個工具的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)，可能會跳過平行探索，而不是使用它自己的對等工具。skill 仍然會執行，但掃描會比較不徹底。有人提議過一個與 harness 無關的改寫版本，但還沒有合併。

**我實際上要怎麼在 TypeScript 中實作深模組？**

這個 skill 沒有附上好的答案。人們常常要求一份 `TYPESCRIPT.md`，為這些原則提供具體的檔案與模組配置，而它並不存在。這個 skill 告訴你加深該放在哪裡、接縫後面放什麼。你必須自己把它轉換成套件或目錄結構。

## 它有效的跡象

- 候選項目指名的是你領域的概念，而不是編造的類別名稱：「Order 進件模組」，而不是「FooBarHandler」。
- 候選項目位於你最近編輯過的檔案中，而不是 repo 中沒人碰的部分。
- 執行期間沒有任何程式碼改變。唯一的新檔案是你暫存目錄中的 HTML 報告。
- 它在報告之後停下來，問你想要哪一個候選項目。它不會自己繼續下去。
- 每張卡片都以局部性或槓桿來解釋回報，並說明哪些測試會變簡單，而不只是「這樣比較乾淨」。
- 當你以一個長期的理由否決一個候選項目時，它會提議記錄一份 ADR，讓下一次執行不會再建議它。

## 它的定位

`improve-codebase-architecture` 是**定期維護**。你每隔幾天在任何鏈之外執行它，用來排入工作，而不是做工作。它的鄰居：

- [codebase-design](https://aihero.dev/skills-codebase-design) 擁有每個候選項目都會使用的深度與接縫詞彙。
- [grilling](https://aihero.dev/skills-grilling) 在你選定一個候選項目之後走過決策樹。
- [domain-modeling](https://aihero.dev/skills-domain-modeling) 在你做決策時讓 `GLOSSARY.md` 與 ADR 保持最新。

它的產出是一個想法，會在 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 或 [to-spec](https://aihero.dev/skills-to-spec) 回到主要建置流程。它在主流程末端的對應者是 [retro](https://aihero.dev/skills-retro)。這個 skill 改善 agent 工作所在的程式碼，而 `retro` 在一次建置之後改善它周圍的環境（檢查、規範、引導檔案）。至於哪個 skill 適合哪種情況，[ask-matt](https://aihero.dev/skills-ask-matt) 是整套 skill 的路由器。
