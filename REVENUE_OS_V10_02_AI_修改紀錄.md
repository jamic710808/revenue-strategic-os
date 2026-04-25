# Revenue Strategic OS V10.02 AI — 修改紀錄

**修改日期：** 2026-04-25
**修改檔案：** `Revenue_Strategic_OS_V10_02_AI.html`
**新增檔案：** `revenue_cors_proxy.py`
**參考來源：** `c:\Users\jamic\應收帳款\AR 量子智慧戰情室 V5.6_AI.html`

---

## 修改摘要

參照 AR 量子智慧戰情室 V5.6 的 AI 設定，對 Revenue OS V10.02 的 AI 功能進行全面升級，新增供應商支援、CORS 代理機制、端點 URL 管理強化，以及更完善的錯誤處理。

---

## 詳細變更內容

### 1. 新增 AI 供應商

| 供應商 | Base URL | 預設模型 | 說明 |
|--------|---------|---------|------|
| **OpenRouter** | `https://openrouter.ai/api/v1` | `deepseek/deepseek-chat` | 聚合多家 LLM，支援 claude-sonnet-4-6、gpt-4o、gemini-2.5-flash 等 |
| **Ollama（本地）** | `http://localhost:11434/v1` | 自訂輸入 | 本地端模型，無需 API Key |

HTML 供應商選單與 `AI_PROVIDERS` 物件同步新增。

---

### 2. Anthropic 模型更新

| 項目 | 修改前 | 修改後 |
|------|-------|-------|
| 模型清單 | `claude-opus-4-5`, `claude-sonnet-4-5`, `claude-3-5-sonnet-20241022`, `claude-3-5-haiku-20241022`, `claude-3-opus-20240229` | `claude-opus-4-7`, `claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001` |
| 預設模型 | `claude-sonnet-4-5` | `claude-sonnet-4-6` |

---

### 3. CORS Proxy 支援

**HTML 新增（進階設定內）**
```html
<div class="ai-config-row" id="ai-proxy-row">
  CORS 代理 / 本機代理開關 / Port 輸入 / 環境提示
</div>
```

**JS 新增函式 `onProxyToggle()`**
- 開啟時自動偵測環境：
  - 本機（`localhost`）→ Python Proxy `http://localhost:PORT/ar-proxy`
  - 遠端（Vercel 部署）→ Cloudflare Worker `https://tiny-cell-0a82.jamic710808.workers.dev`
- 設定同步至 `aiState.proxyEnabled` / `aiState.proxyPort`

**`aiState` 新增欄位**
```javascript
proxyEnabled: false,   // 是否啟用 CORS 代理
proxyPort: 8765        // 本機代理 Port
```

---

### 4. 端點 URL 管理強化

**HTML 修改**
- URL 輸入欄改為**所有供應商常駐顯示**（原本僅 custom 顯示）
- 加入 ↺ 重置按鈕，可一鍵還原至供應商預設 URL
- Placeholder 更新為更清楚的說明文字

**JS 新增函式**
| 函式 | 功能 |
|------|------|
| `_fillUrlForProvider(provider)` | 切換供應商時自動填入對應 URL（優先用已儲存的覆寫值） |
| `_markUrlModified()` | 偵測 URL 是否已被修改，更新重置按鈕樣式（紫色醒目提示） |
| `_refreshUrlResetBtn(provider)` | 更新重置按鈕外觀 |
| `_resetUrlToDefault()` | 還原為該供應商的預設端點 URL |

**`aiState` 新增欄位**
```javascript
customBaseUrls: {}     // 各供應商的 URL 覆寫記錄（{ openai: '...', deepseek: '...' }）
```

---

### 5. API Key 管理強化

**JS 新增函式 `_fillKeyForProvider(provider)`**
- 切換供應商時自動從 `aiState.keys` 讀出並填入對應的已儲存 Key
- 同步更新連線狀態指示（就緒 / 未連線）

原本 `onProviderChange` 改為呼叫此函式，邏輯更集中。

---

### 6. Ollama 支援

`_toggleCustomRows()` 和 `updateModelOptions()` 均新增 `isOllama` 判斷：
- Ollama 與 custom 相同：隱藏下拉模型選單，改顯示自由輸入欄
- Ollama 呼叫時不需要 API Key（`callAI` 加入豁免判斷）

---

### 7. 錯誤處理集中化

**JS 新增函式 `_handleAIError(err, msgEl)`**

取代原本散落在 `callAI` catch 區段的錯誤邏輯，統一處理：

| 錯誤類型 | 處理方式 |
|---------|---------|
| `AbortError` | 顯示「（已中止）」提示 |
| CORS / `Failed to fetch` | 顯示中文四點建議（CORS 設定、Key、URL、file:// 問題） |
| HTTP 401 / 402 / 429 / 404 / 500 / 503 | 對應的中文友善訊息 |

---

### 8. URL 清理工具函式

**JS 新增函式 `_normalizeBaseUrl(url)`**

自動清理使用者輸入的端點 URL：
- 移除空白字元（含全形空白）
- 去除尾端多餘路徑後綴（`/chat/completions`、`/messages`、`/completions`）
- 去除尾端斜線

例：`https://api.x.com/v1/chat/completions ` → `https://api.x.com/v1`

---

### 9. 設定持久化更新

`saveAISettings()` 和 `loadAISettings()` 同步支援新欄位：

**新增儲存/還原項目**
- `proxyEnabled`
- `proxyPort`
- `customBaseUrls`（各供應商的 URL 覆寫記錄）

`loadAISettings()` 改為 `applyToUI()` 內部函式模式，確保 DOM 就緒後才套用 UI。

---

### 10. 串流模式 value 修正

`initAIPanelEvents` 中的 stream 判斷由 `=== 'stream'` 修正為 `!== '0'`，與 HTML select option value（`'1'` / `'0'`）一致。

---

## 新增檔案

### `revenue_cors_proxy.py`

本機 CORS 中轉代理伺服器，解決瀏覽器直接呼叫外部 API 的跨域封鎖問題。

**啟動方式**
```bash
cd c:\Users\jamic\收入
python revenue_cors_proxy.py        # Port 8765（預設）
python revenue_cors_proxy.py 9000   # 自訂 Port
```

**功能**
- 靜態檔案服務：`http://localhost:8765` 列出目錄，可直接點擊開啟 HTML
- CORS Proxy：攔截 `/ar-proxy` 路徑，轉發至 `X-Proxy-Target` header 指定的目標 API

**使用情境**

| 開啟方式 | 是否需要 Proxy |
|---------|--------------|
| 雙擊 `.html`（`file://`） | 需要啟動此腳本並開啟代理開關 |
| `python revenue_cors_proxy.py` 後用 `localhost` 開啟 | 不需要開關（直連 API） |
| 部署至 Vercel | 不需要，自動走 Cloudflare Worker |

---

## 與 AR V5.6 的對應關係

| AR V5.6 元素 | Revenue OS V10.02 對應 |
|-------------|----------------------|
| `AR_AI_PROVIDERS` | `AI_PROVIDERS` |
| `arAiState` | `aiState` |
| `AR_AI_STORAGE_KEY = 'arV5AiSettings'` | `AI_STORAGE_KEY = 'revenueOS_aiSettings'` |
| `AR_AI_CHAT_CACHE_KEY = 'arV5AiChatCache'` | `AI_CACHE_KEY = 'revenueOS_aiChatCache'` |
| `ar_cors_proxy.py` | `revenue_cors_proxy.py` |
| `_arNormalizeBaseUrl` | `_normalizeBaseUrl` |
| `_arHandleFetchError` + catch block | `_handleAIError` |
| `_arFillKeyForProvider` | `_fillKeyForProvider` |
| `_arFillUrlForProvider` | `_fillUrlForProvider` |

System Prompt 與 TAB_PROMPTS 維持原 Revenue OS 的營收分析版本，未替換為 AR 版本。
