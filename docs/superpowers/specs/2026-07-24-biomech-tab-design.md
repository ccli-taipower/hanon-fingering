# 生物力學獨立 tab — 設計 spec

**日期**：2026-07-24
**目標**：把「指法查詢」tab 瘦身成純指法工具，將所有生物力學/比較內容抽到一個獨立的「生物力學」tab，並把先前做的比較 artifact 完整移植進來（重繪為站方風格）。

## 動機

目前 `#tab-fingering` 一次塞了四種東西：兩套系統的長篇論述（`flat-system-wrap` + `sharp-system-wrap`）、統一 vs 哈農評比 + DP 分析、各調查詢表、指法公式 rules panel。使用者一打開想查指法，卻被論述打斷。把「為什麼」抽離，「指法查詢」只回答「怎麼按」。

## 範圍

### Tab 列（`nav.tab-nav`，目前 line 1277–1282）
新順序：`指法查詢 │ 生物力學(新) │ 五度圈 │ 練習指南 & 計劃 │ 進度總覽`
生物力學插在第 2 位（緊接指法查詢）。

### 指法查詢 tab（`#tab-fingering`）— 瘦身
- **保留**：各調查詢表（含調式鈕 大調/和聲/旋律、哈農/統一切換鈕）＋ 指法公式 & 規律 rules panel。
- **移出**：`<div id="flat-system-wrap">`（line 1290）、`<div id="sharp-system-wrap">`（line 1291）。
- 淨結果：打開即查表，無長篇論述。

### 生物力學 tab（`#tab-biomech`，新）— 由上而下
1. **命題 Hero**：兩軸勝負相反（雙 verdict 卡：生物力學→哈農贏／系統自洽→統一贏），改寫自 artifact，用站方色。
2. **兩套系統說明**（搬移）：`flat-system-wrap` + `sharp-system-wrap` 原樣移入本 tab（render 函式以 ID 注入，容器換 tab 即可，JS 邏輯不動）。內含降記號統一版（拇指對齊 C/F）+ 升記號鏡像對稱 + 現有「統一 vs 哈農」與「DP 評比」兩個 box。
3. **新增 artifact 內容**（新 render 函式 `renderBiomechExtra()` 注入新容器 `#biomech-extra-wrap`），全部重繪為站方 tokens：
   - **cost 長條**：哈農 677.2 vs 統一 760.0，多付 +82.8（集中在 B♭/E♭/A♭）。
   - **兩個力學量卡**：穿越淨空、跨越落點（各附小鍵盤黑鍵 vs 白鍵示意）。
   - **E♭ 左手逐音拆解**：音名列（黑/白鍵）＋哈農列＋統一列，拇指錨點 highlight、跨越 4 指圈框；底下兩行半音/全音註解。
   - **逐調建議**：入門用統一 / 熟練換哈農（B♭/E♭/A♭）＋迴轉爬指警告。

### 不動
- **調性識別公式**（`#decision-wrap`）：本來就在五度圈 tab，不搬、不改。
- 現有 `renderFlatSystem()` / `renderSharpSystem()` 內容與邏輯不改，只是容器換到新 tab。

## 樣式對映（artifact → 站方 tokens）

| artifact | 站方 token |
|---|---|
| 象牙/烏木地 | `--paper` / `--paper-warm` / `--ink` |
| 黃銅（哈農/力學贏）| `--accent-2` 鏽紅 #7a3a2c |
| 石板藍（統一/認知贏）| `--accent` 靛藍 #2c3e7a |
| cost 斜紋（多付）| `--accent-2` 半透明 |
| 等寬數字 | `--mono` Source Code Pro + `tabular-nums` |
| 圓角 | `--r-sm/md/lg` |

**不**引入 artifact 的獨立配色；全部走站方既有 tokens，與其他 tab 視覺一致。

## 實作要點

1. `nav.tab-nav`：加 `<button data-tab="biomech">生物力學</button>`（第 2 位）。
2. 新增 `<div class="tab-panel" id="tab-biomech">`，內含移入的 `flat-system-wrap` + `sharp-system-wrap` + 新 `biomech-extra-wrap`。
3. 從 `#tab-fingering` 移除 line 1290–1291 兩個 wrap。
4. 新 `renderBiomechExtra()`，加入初始 render 呼叫串（現 line 3623–3624 附近，與 renderFlatSystem/renderSharpSystem 並列）。
5. Tab 切換 JS（line 2117+）為 generic，新 tab 自動生效；biomech 內容在載入時 render（非 on-demand）。
6. 現有 hero 內文中「見上方 §頂音邊界規則」等跨區指涉：搬移後仍在同 tab 內，錨點語意保持成立。

## 驗收

- 指法查詢 tab 打開只有查詢表 + rules panel，無兩套系統論述。
- 生物力學 tab 順序正確、內容完整、全站方風格、亮暗一致（站方單一主題）。
- cost 長條、力學量卡、E♭ 拆解、逐調建議數據與 artifact 一致（哈農 677.2 / 統一 760.0；E♭ 哈農 3 2 1 4… / 統一 2 1 4 3…）。
- 其他 tab（五度圈/練習/進度）不受影響；調性公式仍在五度圈。
- 手機寬度不橫向溢出（cost 長條、E♭ grid 各自 `overflow-x:auto`）。

## 非目標

- 不改指法資料（DATA/SCALES）。
- 不改 harm/mel 其餘調的補齊（另案）。
- 不引入外部字型/資源（站方為純靜態單檔）。
