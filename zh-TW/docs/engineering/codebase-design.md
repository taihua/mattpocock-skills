## 它做什麼

`codebase-design` 固定你用來設計模組的用詞：**module（模組）**、**interface（介面）**、**depth（深度）**、**seam（接縫）**、**adapter（轉接器）**、**leverage（槓桿）**、**locality（局部性）**。它精確地定義每一個詞、禁止鬆散的替代詞（「component」、「service」、「API」、「boundary」），並說明由它們推導出的少數幾條原則。

它是一份參考資料，而不是一個流程。它不執行任何迴圈、不產出任何產出物，也從不停下來問你問題。其他每一個涉及設計的 skill 都使用它的詞彙。單獨使用時，它給你用詞，然後就停止。在觸發它之前要知道這一點。如果你把一個 [session](https://www.aihero.dev/ai-coding-dictionary/session) 指向一個沒有流程、也沒有停止規則的 skill，然後說「開始」，agent 就會自己發明一個流程。下面的問題會說明那是什麼樣子。

## 何時使用

輸入 `/codebase-design`，或者當一個設計任務適合時，agent 會自動使用它。

當你已經知道要重新設計哪一段程式碼，而你需要思考它的形狀時使用它：接縫要放在哪裡、介面可以多小、一次抽取是否能把複雜度從呼叫者身上移除。也可以用它來解決關於某個設計用詞是什麼意思的爭論。

有好幾個 skill 跟它很接近。依你遇到的問題來選：

| 問題 | Skill |
|---|---|
| 一個模組的形狀：它的介面、它的接縫、它的深度 | `codebase-design` |
| *領域的用詞*：「account」有三種意思、兩個人說「cancellation」時指的是不同的東西 | [domain-modeling](https://aihero.dev/skills-domain-modeling) |
| 你還不知道要重新設計*哪一個*模組 | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)（找出候選項目的勘查） |
| 你想要有人和你爭論這個設計，而不只是命名它 | [grilling](https://aihero.dev/skills-grilling) |
| 有一個具體的行為要建立，而你想要能撐過重構的測試 | [tdd](https://aihero.dev/skills-tdd) |

## 詞彙

詞彙表就是這個 skill。skill 把每一個術語相對於其他術語來定義，並為每一個指出它所取代的詞。

| 術語 | 意思 | 不要說 |
|---|---|---|
| **Module（模組）** | 任何有介面與實作的東西。刻意不限定規模：一個函式、一個類別、一個套件、一個跨越多層的切片。 | unit、component、service |
| **Interface（介面）** | 呼叫者要正確使用它所必須知道的一切：型別簽章，加上不變式、順序限制、錯誤模式、必要的設定、效能特性。 | API、signature |
| **Depth（深度）** | 介面上的槓桿：呼叫者或測試每學一單位介面所能使用的行為量。**深**：在小介面後面有大量行為。**淺**：介面幾乎和實作一樣複雜。 | 無 |
| **Seam（接縫）** | Michael Feathers 的術語：一個你不必在該處編輯就能改變行為的地方。它是介面的*位置*，而要把它放在哪裡本身就是一個決策，與它後面放什麼是分開的。 | boundary |
| **Adapter（轉接器）** | 在接縫上滿足某個介面的具體東西。它指名的是一種角色，而不是一種東西：一個記憶體內的 fake 與一個 Postgres repo 都是 adapter。 | 無 |
| **Leverage（槓桿）** | 呼叫者從深度中得到的東西：每學一單位介面就能得到更多能力。 | 無 |
| **Locality（局部性）** | 維護者從深度中得到的東西：變更、bug 與驗證都留在一個地方，所以一次修正就能涵蓋每一個呼叫者。 | 無 |

這個 skill *不*把深度定義為實作行數與介面行數的比例，那是 Ousterhout 自己的定義。那個指標會獎勵灌水實作，所以這個 skill 改用「深度即槓桿」。

## 四條原則

- **深度是介面的特性，而不是實作的特性。** 一個深模組的內部可以由小的、可替換的部件組成。呼叫者只是看不到它們。一個模組可以有它自己的測試會用到的內部接縫，以及一個位於它介面上的外部接縫。
- **刪除測試。** 想像把這個模組刪掉。如果複雜度消失了，這個模組只是一個轉傳層。如果它在 N 個呼叫者中重新出現，這個模組就是在隱藏真正的複雜度。
- **介面就是測試面。** 呼叫者與測試跨越同一個接縫。如果你想測試*越過*介面的東西，這個模組的形狀就不對。
- **一個 adapter 代表假想的接縫。兩個 adapter 才代表真正的接縫。** 在有某個東西會在接縫兩側變化之前，不要切出接縫。只有一個 adapter 的接縫只是多一層間接。

有兩份輔助檔案會更進一步，而 skill 是依需要讀取它們，而不是一開始就讀。[DEEPENING.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DEEPENING.md) 把一個候選項目的依賴分成四類（程序內、可在本地替代、遠端但自有、真正的外部服務），因為類別決定了你要如何跨越接縫測試加深後的模組。[DESIGN-IT-TWICE.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DESIGN-IT-TWICE.md) 會啟動平行的[子代理（sub-agents）](https://www.aihero.dev/ai-coding-dictionary/subagent)，為同一個模組產出三個以上截然不同的介面，然後從深度、局部性與接縫位置來比較它們。

## 常見問題

**我實際上要怎麼在 TypeScript 中建立一個深模組？**

這是關於這個 skill 被問最多的問題，而這個 skill 並沒有回答它。它定義了深模組*是*什麼；但對於如何阻止一個亂入的 import 越過介面，它什麼都沒說。[Issue #458](https://github.com/mattpocock/skills/issues/458) 直白地說：「假設我們對介面很滿意，它隱藏了細節等等。但我們要怎麼強制執行它？我認為如果沒有 lint 或清楚的護欄，人類和 LLM 都會隨著時間把它弄得一團亂。」那個討論串中的回答給了三個選項：把它包在一個類別或 IIFE 中，並接受類別會變得非常龐大；把它做成 monorepo 中的一個套件，並接受 monorepo 的工具鏈；或使用像 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 這樣的 linter，禁止繞過介面的 import。在這些機制中，Effect 是最好的，dependency-cruiser 次之。repo 的 `in-progress/` 分類中有一個 `setup-ts-deep-modules` skill，它建立了 `src/packages/<name>/index.ts` 的慣例，但它是一個 beta 通道的 skill，沒有文件頁面，也沒有附帶任何 lint 規則。

**我把一個 session 指向它，結果它燒掉了 10 萬個 [token](https://www.aihero.dev/ai-coding-dictionary/token)，去重新設計我從沒問過的東西。**

這是一個已知的問題，回報為 [issue #449](https://github.com/mattpocock/skills/issues/449)。這個 skill 是模型觸發的，並把自己描述為詞彙，但它裡面沒有任何東西能硬性阻止 agent 把它當成一個可執行的流程。被告知「在 /codebase-design 中繼續，並推動開放中的決策」時，一個 agent 挑了這個 skill 中最接近流程的部分：`DESIGN-IT-TWICE.md` 中的平行子代理。它重新探索了先前一個 session 已經描繪過的程式碼，並工作了很久才問任何問題。這個 skill 沒有驅動型 skill 所擁有的任何護欄（檢查點、一次一個問題、不自動前進），因為它是一份參考資料。變通方法是觸發一個驅動型 skill（`/grill-with-docs`、`/improve-codebase-architecture` 或 `/tdd`），並把 `codebase-design` 當作它的詞彙。這個 issue 仍然開放。

**`design-an-interface` 去哪了？有沒有一個 `/interface-design` skill？**

這個 skill 取代了 `design-an-interface`，並接手了它的內容。沒有任何東西不見。它的「設計兩次」技巧（平行子代理產出截然不同的設計，出自 Ousterhout）在這裡以 `DESIGN-IT-TWICE.md` 的形式發布。另外，好幾個人要求一個專門給深模組 / 薄介面理念的 `/interface-design` skill；這個 skill 已經涵蓋了那個理念，而且沒有計畫做一個獨立的 skill。如果你是為了這兩個名稱之一而來，就是這一頁。

**這不就是一種檔案結構慣例嗎，例如資料夾、barrel 檔、功能切片？**

不是。人們反駁過很多次，而這個 skill 沒有改變。[Issue #95](https://github.com/mattpocock/skills/issues/95) 提議以一種正式化的碎形樹檔案結構，作為深模組的具體實作；回覆是兩者是獨立的：「深模組關乎的是介面的設計，以及透過一個嚴格的介面存取，不論檔案系統長什麼樣子。用這種做法，你完全有可能做出淺模組。」同樣的話題在 #458 中也出現過：「我認為你可能把模組的概念和檔案系統綁得太緊了。檔案系統當然可以是模組形狀的有用提示，但建構深模組並不需要用到檔案系統。」詞彙表刻意把 **module** 定義為不限定規模的。

**`tdd` 真的使用這套詞彙嗎？**

現在有，但有很長一段時間沒有。v1.0 從 `tdd` 中移除了內嵌的深模組筆記，改用這個共用的 skill，但沒有人加上一個指標來取代它們，所以 `tdd` 自己定義了「seam」，也沒有引用任何東西。現在 `tdd` 中已經有了指標，而當開放的問題是介面的形狀而不是測試時，agent 會跟著它走。`tdd` 仍然擁有「seam」作為你*測試*的邊界；這個 skill 擁有它後面的模組形狀。

**設計兩次模式在 Claude Code 之外能運作嗎？**

無法乾淨地運作。`DESIGN-IT-TWICE.md` 說「使用 Agent 工具平行派出 3 個以上的子代理」，而「Agent」是一個 Claude Code [工具](https://www.aihero.dev/ai-coding-dictionary/tool)的名稱。repo 為其他 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（包括 Codex）附帶了 metadata，而那些 harness 可能沒有叫這個名字的工具。所以平行設計階段的可攜性比 skill 的 metadata 所暗示的要低。[Issue #564](https://github.com/mattpocock/skills/issues/564) 在追蹤這件事，目前仍然開放。

**我可以把我自己的概念加進詞彙表嗎，例如 connascence（共生性）、module secrets（模組秘密）、[漸進式揭露](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)？**

有人提議過這些。[Issue #180](https://github.com/mattpocock/skills/issues/180) 加入了 Parnas 的模組秘密與 Page-Jones 的共生性，作為描述跨越接縫洩漏了*什麼*的命名層，並附上了一份可用的 diff；[issue #303](https://github.com/mattpocock/skills/issues/303) 提議在實作內部使用漸進式揭露，讓一個在公開介面上很深的模組內部也有結構。兩者都還開放、沒有合併。發布的詞彙表是刻意保持精簡的，而 skill 本身給出了理由：一致的語言就是重點所在，而一個沒有人一致使用的術語，比沒有術語還糟。

## 它有效的跡象

- 設計討論不再出現「component」、「service」與「boundary」這些詞，而開始出現「module」、「interface」與「seam」。
- 有人能指著一個提議的抽取，不含糊地說出它是否通過刪除測試。
- 一個提議的接縫會附帶指名第二個 adapter，而不只是第一個。
- 對一個介面的討論涵蓋了不變式、順序與錯誤模式，而不只是型別簽章。
- 觸發它不會啟動一個 session。如果 agent 光憑 `/codebase-design` 就開始讀檔案並提議重構，它就是把參考資料誤當成了驅動者。

## 它的定位

`codebase-design` 是一個**隨時都能使用的獨立 skill**，是工程類 skill 底下的詞彙層，而不是任何鏈中的一步。它最近的鄰居是 [domain-modeling](https://aihero.dev/skills-domain-modeling)，那是與它平行、負責*問題領域*用詞而不是模組形狀的參考資料。兩者通常會一起被需要，因為要為一個深模組取一個好名字，兩者都需要。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是另一個：它勘查程式碼庫以尋找加深的候選項目，並以這份詞彙表的用詞寫下每一個，所以由它找到模組，而你用這個 skill 的用詞來設計它。當你不確定哪個 skill 或流程適合時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
