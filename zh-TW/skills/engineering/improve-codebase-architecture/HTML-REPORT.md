# HTML 報告格式

架構審查以單一個自成一體的 HTML 檔呈現，放在作業系統的暫存目錄中。Tailwind 與 Mermaid 都來自 CDN。Mermaid 能可靠地處理圖狀的圖表；手工打造的 div 與內嵌 SVG 則處理比較具編輯風格的視覺（質量圖、剖面圖）。兩者混用：不要什麼都靠 Mermaid，不然看起來會變得千篇一律。

## 骨架

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Architecture review for {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* 一小層自訂樣式，處理 Tailwind 不容易乾淨涵蓋的東西：
         虛線的接縫線、帶手繪感的箭頭等等。 */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Repo 名稱、日期，以及一個精簡的圖例：實線方框 = 模組、虛線 = 接縫、紅色箭頭 = 洩漏、粗黑框 = 深模組。沒有介紹段落。直接進入候選項目。

## 候選卡片

圖表承擔主要的份量。文字精簡、淺白，並且不拘泥形式地使用詞彙表中的術語（來自 `/codebase-design` skill）。

每個候選項目是一個 `<article>`：

- **標題**：簡短，指名這次加深（例如「Collapse the Order intake pipeline（收合 Order 進件管線）」）。
- **徽章列**：建議強度（`Strong` = 翡翠綠、`Worth exploring` = 琥珀色、`Speculative` = 石板灰），加上一個依賴類別的標籤（`in-process`、`local-substitutable`、`ports & adapters`、`mock`）。
- **檔案**：等寬字型的清單，`font-mono text-sm`。
- **之前 / 之後的圖表**：核心重點。兩欄並排。參見下方的模式。
- **問題**：一句話。哪裡痛。
- **解法**：一句話。改變什麼。
- **好處**：條列，每條不超過 6 個字。例如「Tests hit one interface（測試只打一個介面）」、「Pricing logic stops leaking（定價邏輯不再洩漏）」、「Delete 4 shallow wrappers（刪掉 4 個淺包裝）」。
- **ADR 提示框**（如果適用）：琥珀色底框中的一行字。

不要有解釋性的段落。如果圖表需要一段話才能看懂，就重畫圖表。

## 圖表模式

挑選適合該候選項目的模式。混著用。不要讓每張圖都長得一樣。多樣性本身就是重點的一部分。

### Mermaid 圖（處理依賴與呼叫流程的主力）

當重點是「X 呼叫 Y 呼叫 Z，看看這團亂」時，使用 Mermaid 的 `flowchart` 或 `graph`。用 Tailwind 樣式的卡片包起來，讓它看起來不像是空降進來的。用 classDef 設定樣式，把洩漏的邊塗成紅色、深模組塗成深色。序列圖很適合表達「之前：6 次往返；之後：1 次」。

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### 手工打造的方框與箭頭（當 Mermaid 的排版跟你作對時）

模組用帶邊框與標籤的 `<div>` 表示。箭頭用內嵌 SVG 的 `<line>` 或 `<path>` 元素，以絕對定位疊在一個相對定位的容器上。當你希望「之後」的圖看起來像一個粗框的深模組、內部元件以灰色淡化時，就用這種方式，因為 Mermaid 無法以恰當的份量呈現它。

### 剖面圖（適合呈現層層的淺薄）

堆疊水平的色帶（`h-12 border-l-4`），呈現一次呼叫所穿過的各層。之前：6 個什麼也沒做的薄層。之後：1 條標上整合後職責的粗色帶。

### 質量圖（適合「介面跟實作一樣寬」）

每個模組兩個矩形：一個代表介面的表面積，一個代表實作。之前：介面矩形幾乎跟實作矩形一樣高（淺）。之後：介面矩形很矮，實作矩形很高（深）。

### 呼叫圖收合

之前：一棵以巢狀方框呈現的函式呼叫樹。之後：同一棵樹收合成一個方框，原本的呼叫變成內部呼叫，以淡化的方式顯示在其中。

## 樣式指引

- 偏向編輯風格，而不是企業儀表板。寬裕的留白。標題可選用襯線字（`font-serif` 與 stone/slate 色系搭配得很好）。
- 節制用色：一個強調色（翡翠綠或靛藍），加上代表洩漏的紅色與代表警告的琥珀色。
- 圖表高度保持在約 320px，讓之前與之後能舒適地並排而不需捲動。
- 圖表中的模組標籤使用 `text-xs uppercase tracking-wider`，讓它們讀起來像示意圖，而不是 UI。
- 唯一的腳本是 Tailwind CDN 與 Mermaid ESM 匯入。報告除此之外是靜態的：沒有應用程式碼，除了 Mermaid 自身的渲染之外沒有互動性。

## 首要建議區塊

一張比較大的卡片。候選項目名稱、一句話說明原因、一個連到它卡片的錨點連結。就這樣。

## 語氣

淺白的英文、簡潔，但架構相關的名詞和動詞直接取自 `/codebase-design` skill。簡潔不是偏離用語的藉口。

**精確使用：** module、interface、implementation、depth、deep、shallow、seam、adapter、leverage、locality。

**絕不替換成：** component、service、unit（代替 module）· API、signature（代替 interface）· boundary（代替 seam）· layer、wrapper（當你指的是 module 時，代替 module）。

**符合風格的措辭：**

- 「Order intake module is shallow: interface nearly matches the implementation.（Order 進件模組很淺：介面幾乎等於實作。）」
- 「Pricing leaks across the seam.（定價邏輯跨過接縫洩漏。）」
- 「Deepen: one interface, one place to test.（加深：一個介面，一個測試的地方。）」
- 「Two adapters justify the seam: HTTP in prod, in-memory in tests.（兩個 adapter 證明這個接縫成立：正式環境用 HTTP，測試用記憶體內。）」

**好處條列**以詞彙表的術語指出收穫：*「locality：bug 集中在一個模組」*、*「leverage：一個介面，N 個呼叫處」*、*「介面縮小；實作吸收了包裝層」*。不要寫*「更容易維護」*或*「更乾淨的程式碼」*，因為這些詞不在詞彙表中，也配不上它們的位置。

不要含糊其辭、不要清喉嚨式的開場、不要「值得注意的是……」。如果一句話可以變成條列，就把它變成條列。如果一個條列可以刪掉，就刪掉。如果某個術語不在 `/codebase-design` 的詞彙表中，在發明新詞之前，先找一個詞彙表裡有的。
