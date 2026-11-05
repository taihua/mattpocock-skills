## 它做什麼

`tdd` 以測試先行的方式建立功能或修復 bug：一個失敗的測試，然後剛好足以讓它通過的程式碼，然後是下一個行為。它帶有讓這個迴圈產出值得保留之測試的標準：什麼是好的測試、測試該放在哪裡、mock 是用來做什麼的，以及會讓一整套測試毫無價值的三種反模式。

在一個你沒有先同意的接縫上，它不會寫任何測試。在任何測試存在之前，它會指名它打算在哪些公開邊界上測試，並停下來等你確認。測試的心力是有限的，而這一步讓心力花在關鍵路徑上，而不是每一個邊界情況。`tdd` 也是一份**參考資料**，而不是一個驅動者。它包含迴圈的規則，而由其他東西（你，或 [implement](https://aihero.dev/skills-implement)）來執行套用這些規則的 [session](https://www.aihero.dev/ai-coding-dictionary/session)。

## 何時使用

輸入 `/tdd`，或者當任務適合時，[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 會自動使用它：以測試先行的方式建立功能或修復 bug，或當你說「red-green-refactor」時。

當有一個具體的行為要建立，它有輸入與可觀察的輸出，而你想要能撐過重構的測試時，就使用它。

| 你的情況 | 去哪裡 |
| --- | --- |
| 一個有明確輸入與輸出的行為（商業邏輯、請求 / 回應合約、資料轉換、驗證） | `tdd` |
| 行為還沒確定下來 | [to-spec](https://aihero.dev/skills-to-spec)，它也會在任何程式碼寫下之前約定測試接縫 |
| 問題其實是介面的形狀，而不是測試 | [codebase-design](https://aihero.dev/skills-codebase-design) |
| 你有一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 或一些 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)，想讓整個建置替你執行 | [implement](https://aihero.dev/skills-implement)，它會每張 ticket 驅動一次 `tdd` |
| 設定、接線、膠水程式碼、型別註記、單純的 CRUD 委派 | 這裡沒有適合的；參見下方的開放缺口 |

最後一列是一個真正的缺口。這個 skill 決定接縫要放*在哪裡*，但其中沒有任何東西決定一個變更*是否*值得跑這個迴圈。如果你在一個沒有獨立事實來源可以斷言的變更上執行它，你會得到一個重述實作的測試。那就是這個 skill 所警告的同義反覆反模式，只是從另一個方向抵達。它是 [issue #746](https://github.com/mattpocock/skills/issues/746)，目前仍然開放。在它關閉之前，由你自己做這個判斷，或把規則寫進你的 `CLAUDE.md`。

## 前置條件

需要安裝 [codebase-design](https://aihero.dev/skills-codebase-design)。`tdd` 以前帶有自己的深模組與介面設計筆記；v1.0 刪除了它們，改用共用的 skill，現在 `tdd` 使用它的介面設計詞彙。除此之外沒有其他需求；這個 skill 是[無狀態的](https://www.aihero.dev/ai-coding-dictionary/stateless)，不會寫任何自己的檔案。

## 迴圈，以及它所在的接縫

這個 skill 建立在三個術語之上。

**紅燈、綠燈（Red-green）。** 寫一個失敗的測試，然後只寫剛好足以讓它通過的程式碼。不要為下下個測試寫程式碼。沒有重構階段。這個 skill 在 2026 年 6 月拿掉了它，因為 agent 幾乎從不執行它，也因為審查與實作分成不同的 session 效果更好。重構屬於 [code-review](https://aihero.dev/skills-code-review)。

**垂直切片（Vertical slice）。** 在一個接縫上寫一個測試，然後是最小的實作，然後重複。第一個循環是一顆**曳光彈（tracer bullet）**，證明單一條路徑從頭到尾可以走通。相反的做法是水平切片：先寫完所有測試，再寫完所有程式碼。大量寫出的測試驗證的是*想像中*的行為。它們檢查的是東西的形狀，而不是使用者做什麼，而且它們會讓你在理解實作之前就把測試結構定死。

**事先約定的接縫（Pre-agreed seam）。** 接縫是你在不伸手進入內部的情況下觀察行為的公開邊界。這條規則沒有例外。不在未經確認的接縫上寫任何測試。在完整的鏈中，接縫會在更早的 [to-spec](https://aihero.dev/skills-to-spec) 期間約定：「`/tdd` 被告知只在事先約定的測試接縫上工作，`/code-review` 檢查是否只使用了約定好的測試接縫。」單獨觸發時，`tdd` 會直接問你。

它被寫來防止的三種反模式：

| 反模式 | 徵兆 |
| --- | --- |
| 與實作耦合 | 你重新命名一個內部函式時測試壞了，但行為並沒有改變。mock 了內部協作者、斷言呼叫次數、用資料庫查詢來驗證而不是透過介面。 |
| 同義反覆 | 期望值是用程式碼計算的同一種方式算出來的，所以測試在構造上就一定會通過。期望值必須來自別的地方：一個已知正確的字面值、一個算好的範例、spec。 |
| 水平切片 | 在任何實作之前，就落地了一批測試。 |

mock 只用於系統邊界：外部 API、時間、隨機性，有時候是檔案系統或資料庫。絕不 mock 你自己的模組。

## 常見問題

**它為什麼不重構？description 明明寫著「red-green-refactor」。**

因為重構步驟被拿掉了，description 卻沒有改。拿掉是刻意的。agent 幾乎從不執行這一步，而且讓實作與審查分在不同的 session 效果更好。結果按照教科書是否仍算是 TDD，不如這個迴圈是否產出更好的程式碼來得重要。觸發語句與內文之間的不一致已經回報為 [issue #589](https://github.com/mattpocock/skills/issues/589)，目前仍然開放，所以「red-green-refactor」仍然可以作為觸發這個 skill 的語句。你得到的是紅燈、綠燈，而重構在 [code-review](https://aihero.dev/skills-code-review) 中進行。

**它要我選一個測試接縫，但我完全不知道該選哪個。**

這是這個 skill 被回報最多的摩擦點（[issue #607](https://github.com/mattpocock/skills/issues/607)）。提示只列出候選接縫的名稱，完全沒說明每一個能抓到什麼、會漏掉什麼，所以你是在標籤之間做選擇。目前還沒有發布任何修正。實際的變通方法是在回答之前先請 agent 說明取捨：元件層級的接縫會漏掉哪些整合接縫能抓到的東西，以及它慢多少。這也是為什麼整條鏈要在 `to-spec` 中事先約定接縫，因為在那裡你能看到整個功能，而不只是一個提示。

**它先寫了實作才寫測試，即使 skill 說要先紅燈。**

這確實會發生。一位使用者追問[模型](https://www.aihero.dev/ai-coding-dictionary/model)這件事，得到了一個異常誠實的回答：「我知道 skill 說『一次一個測試，看著它因為正確的理由失敗』。我讀過了。我只是預設回到了我平常的習慣。」這個 skill 接受這一點。沒有任何指示能讓 agent 100% 遵守，而更嚴格的措辭會為了很小的收穫而限制 agent 的創造力。即使 agent 沒有嚴格遵守，這個迴圈仍然值得執行，因為整體的結果還是比較好。如果嚴格遵守對某個特定切片很重要，就盯著執行過程，而不是信任 skill 會強制執行。

**它應該先寫瀏覽器測試或端到端測試嗎？**

通常不應該，而這個 skill 不會阻止它。一位使用者回報 agent 先寫了一個 Playwright 測試，然後花了很長的迴圈一再重新執行它，最後斷定是*測試*壞了，而那個功能其實根本還不存在。瀏覽器測試慢到讓紅燈、綠燈回饋迴圈不再划算。在你 repo 的 `CLAUDE.md` 中說明，瀏覽器測試要在行為可以運作之後才寫。

**`/tdd` 會取代 `/implement`，或課程中的 `/do-work` 嗎？**

不會。`/tdd` 記錄的是方法論；`/implement` 是一個簡單的「工作 → 回饋 → commit」迴圈，是 `/do-work` 的直接替代品。課程中單一的 `/do-work` 步驟現在被拆成 `/implement`、`/tdd` 與 `/code-review`。如果你在問要對一張 ticket 執行哪一個，答案幾乎永遠是 `/implement`。

**深模組與介面設計的指引去哪了？**

在 v1.0 移到了 [codebase-design](https://aihero.dev/skills-codebase-design)，並加以通用化，讓好幾個 skill 共用同一套詞彙。`refactoring.md` 也在同時被移除；重構現在是 [code-review](https://aihero.dev/skills-code-review) 的工作，而那個 skill 帶有 Fowler 的壞味道基準。

**它知道我的其他 ticket 嗎？**

不知道。對一張 ticket 執行時，它可能會提出屬於兄弟 ticket 的工作，因為它看不到 issue 圖的其他部分（[issue #129](https://github.com/mattpocock/skills/issues/129)）。這不是 `tdd` 的工作。把 spec 和 ticket 一起傳入會有幫助；一開始就把 ticket 調整到合適的大小，幫助更大。

## 它有效的跡象

- 在任何測試檔存在之前，它會停下來指名它打算測試的接縫，並等待。
- 出現一個測試、亮紅燈、得到剛好足以通過的程式碼，然後才出現下一個測試，而不是一批測試接著一批程式碼。
- 測試名稱讀起來像能力（「使用者可以用有效的購物車結帳」），而不是內部細節（「checkout 呼叫 paymentService.process」）。
- 斷言中的期望值是你能追溯到 spec 的字面值，而不是用程式碼計算的方式重新算出來的值。
- 重新命名一個內部函式，不會讓測試套件中的任何東西壞掉。
- mock 只出現在外部邊界上（金流 API、時鐘），從不出現在你自己的模組周圍。

## 它的定位

`tdd` 在主鏈的建置步驟內部執行；它本身不是一個步驟：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

[to-spec](https://aihero.dev/skills-to-spec) 事先約定測試接縫，[implement](https://aihero.dev/skills-implement) 每張 ticket 驅動一次 `tdd`，而 [code-review](https://aihero.dev/skills-code-review) 事後檢查是否只使用了約定好的接縫，並負責 `tdd` 不再做的重構。它的另一個鄰居是 [codebase-design](https://aihero.dev/skills-codebase-design)，也就是 `tdd` 所使用的接縫與深模組詞彙的共用來源。你也可以單獨使用它，只要有一個具體的行為要建立，而且沒有完整的 spec。當你不確定哪個 skill 適合你的情況時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
