## 它做什麼

當你看不懂某則訊息時，就輸入 `wait-what`。接著 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 會重新說明它剛才說的話。它會補上你缺少的背景、用淺白的英文寫，並使用你專案 `GLOSSARY.md` 中的詞彙。

這個 skill 只有三行。這就是它的設計，不是沒寫完的草稿。對抗囉嗦的 skill 通常會失敗，因為它們變得太長。一個四百行的「要簡潔」skill 仍然會讓[模型](https://www.aihero.dev/ai-coding-dictionary/model)很囉嗦，因為模型模仿的是 skill 的長度，而不是它的要求。這個 skill 只有一個精準的主導詞，其他什麼都沒有。

## 何時使用

你透過輸入 `/wait-what` 來觸發它。agent 不會自己去用它，也不應該。只有你知道自己是在什麼時候跟不上的。

一發現自己在跳著讀就使用它。例如，agent 開始使用它自己發明的術語、在一句話裡塞了五個縮寫，或解釋了一個你從沒看過其前提的決策。它修復的是你已經身在其中的這段對話。若要讓術語根本不會出現，請使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它會事先建立共用語言。

## 名稱就是機制

主導詞是 **wait（等等）**。「要簡潔」是對 agent 輸出的指示，模型遵守它的方式就是刪減字句，結果讓你更跟不上。**Wait** 講的是*你*的狀態。它表示你在這裡不懂了。聽到「簡短一點」的 agent 會寫出斷斷續續的片段。聽到「等等，你把我搞糊塗了」的 agent 會退一步並解釋。

這個差異就是整個 skill。每一個熱門的囉嗦解法都在指名*輸出*：`/tldr`、`/no-fluff`、`/talk-normal`。模型會過度修正成斷斷續續的片段，變短了，卻沒有更清楚。指名*聽者*則同時要求兩件事：更少的字**以及**你缺少的背景。

這個 skill 說的是重新說明**那個**，而不是「上一則訊息」。讓你跟不上的通常不只一段話，所以由 agent 決定要往回追溯多遠。

## 它接上你既有的語言

內文重複使用了你全域 `CLAUDE.md` 與專案 `GLOSSARY.md` 中已有的主導詞。ASD-STE100 簡化技術英文設定了語域。通用語言（ubiquitous language）提供了名詞。這個 skill、`CLAUDE.md` 與 `GLOSSARY.md` 使用相同的 [token](https://www.aihero.dev/ai-coding-dictionary/token)，所以觸發它並不是一個新的指示。它是在提醒 agent 一個它早已同意過的指示。

如果你沒有 `GLOSSARY.md`（也沒有為目前的 context 指向某份 glossary 的 `GLOSSARY-MAP.md`），這個 skill 仍然有效。你只會失去領域詞彙那一半。

## 它有效的跡象

- 重新說明的內容**更短也更清楚**，而不是更短但更生硬。
- 它補上了你缺少的前提，而不只是刪掉字。
- 專案的名詞取代了自創的名詞。你 `GLOSSARY.md` 中的術語回來了。
- 你可以連續使用兩次，它也不會退化成過度簡略。

## 它的定位

你可以在任何時候、任何對話中、任何其他 skill 之中使用 `wait-what`。它在事後修復一則訊息。真正的解法是事先約定好的共用語言，那就是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：一場在進行中同時執行 [domain-modeling](https://aihero.dev/skills-domain-modeling) 的[拷問（grilling）](https://www.aihero.dev/ai-coding-dictionary/grilling) session，它會把你們雙方使用的詞彙記錄在你的 `GLOSSARY.md` 中。如果你不確定哪個 skill 適合當下，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
