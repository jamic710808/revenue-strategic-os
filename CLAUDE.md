# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## 專案性質

這是一個**純靜態 HTML 專案**，無 build 工具、無套件管理器、無測試框架。所有邏輯、樣式、腳本均內嵌於單一 HTML 檔案中。直接用瀏覽器開啟即可運行，無需任何編譯步驟。

---

## 核心檔案

| 檔案 | 說明 |
|------|------|
| `Revenue_Strategic_OS_V7_0_Enhanced.html` | **主要版本**（732 行）— 完整 8 標籤頁看板，含真實 PDF 數據 |
| `Revenue_Strategic_OS_V6_2_Ultimate.html` | 原始版本（499 行）— 保留作為對照參考 |
| `PDF_CONTENT_REPORT.md` | 17 份 PDF 的完整內容提取報告 |
| `REVENUE_OS_V7_升級報告.md` | V6.2 → V7.0 升級作業記錄 |

PDF 原始資料（`03.pdf` ~ `10.pdf` 及對應精品投影片版）為唯讀參考資料，不需修改。

---

## 開啟方式

```bash
# 直接開啟（Windows）
start Revenue_Strategic_OS_V8_0_Ultimate.html

# 或用 Python 起一個本地伺服器（避免部分瀏覽器的 CORS 限制）
python -m http.server 8080
# 然後訪問 http://localhost:8080/Revenue_Strategic_OS_V8_0_Ultimate.html
```

---

## HTML 架構

V7.0 檔案結構為**單檔三層**：

```
HTML 檔案
├── <style>（約 200 行）— CSS 變數、元件樣式、響應式規則
├── <body>
│   ├── .sidebar — 固定寬度 260px 導航，含 8 個 .nav-item[data-tab]
│   └── .main-content — 8 個 .tab-pane，預設只顯示 active 的那個
└── <script>（約 50 行）— Tab 切換、DAX 搜尋、趨勢書籤切換
```

**8 個標籤頁對應的 ID：**

| data-tab | 內容 | 主要 PDF 來源 |
|----------|------|--------------|
| `tab-summary` | 營收指揮部 | 03.pdf |
| `tab-product` | 產品策略 & Pareto | 05.pdf |
| `tab-basket` | MBA 關聯分析 | 06.pdf |
| `tab-rfm` | RFM 價值矩陣 | 07.pdf |
| `tab-region` | 區域分析戰略 | 08.pdf |
| `tab-lifecycle` | 生命週期趨勢 | 04.pdf |
| `tab-forecast` | 預測 vs 預算 | 09.pdf + 10.pdf |
| `tab-dax` | DAX 知識引擎 | 全部 PDF |

---

## CSS 元件系統

修改樣式時，優先使用已定義的 CSS 變數（定義於 `:root`）：

```css
--accent-primary: #0ea5e9   /* 藍色，主要強調 */
--accent-success: #10b981   /* 綠色，正向指標 */
--accent-danger:  #f43f5e   /* 紅色，警示 */
--accent-warning: #f59e0b   /* 橙黃，觀察 */
--accent-purple:  #a855f7   /* 紫色，AI/特殊 */
```

常用元件 class：`.glass-card`、`.badge`（搭配 `.b-up/.b-down/.b-warn/.b-info`）、`.bar-chart`、`.progress-bar`、`.data-table`、`.dax-card`

---

## 新增 DAX 公式的方式

在 `#daxList` 內複製一個 `.dax-card` 區塊，調整以下欄位：

- `.dax-title` — 公式名稱
- `.tag` — 分類標籤（使用 `tag-blue/green/purple/orange`）
- `.code-box` — 公式內容，使用 `<span class="kw/fn/var/str/cm">` 做語法高亮

DAX 搜尋功能會自動對所有 `.dax-card` 的 `textContent` 做即時篩選，無需額外設定。

---

## 數據來源

所有看板數值均來自 PDF 提取，詳見 `PDF_CONTENT_REPORT.md`。修改數值時請對照該報告確認來源，避免與原始資料不符。

核心數據參考：營收 1,031.30 萬（+140.14% YoY）、淨利潤率 4.97%、客戶數 10,493、RFM 總計 9,986 人、區域總收入 31,238.80 萬。
