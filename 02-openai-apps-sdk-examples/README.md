# OpenAI Apps SDK Examples

## 範圍說明
- 本文件涵蓋實驗 2.1～2.2，聚焦 `openai/openai-apps-sdk-examples` 的 Solar system Python server，並驗證 UI 渲染是否正常。

## 語言/版本
- Python 3.12
- WSL（bash）

## 實驗 2.1：啟動 Solar system Python server
### 目的
- 以最小步驟啟動 Solar system Python server，確認範例服務可正常運作。

### 前置需求
- 已安裝 Python 3.12
- 已安裝 `uv`
- 已安裝 Node.js 18+
- 已安裝 `pnpm`
- 已安裝 Git
- 已安裝 `cloudflared`
- 已設定 `OPENAI_API_KEY` 環境變數
- 可連線網路以取得依賴

### 步驟
```bash
git clone https://github.com/openai/openai-apps-sdk-examples.git
cd openai-apps-sdk-examples
```

```bash
pnpm install
```

```bash
# Terminal A（WSL）：啟動靜態資源服務
pnpm run serve
```

```bash
# Terminal B（WSL）：建立前端 tunnel（4444）
cloudflared tunnel --url http://localhost:4444
# 記下輸出的公開網址，作為 <FRONTEND_TUNNEL_URL>
```

```bash
# Terminal C（WSL）：用前端公開網址重建 assets，讓 ChatGPT 可抓到靜態資源
BASE_URL=<FRONTEND_TUNNEL_URL> pnpm run build
```

```bash
# Terminal D（WSL）：啟動 Solar system MCP server
cd solar-system_server_python
uv sync
uv run uvicorn main:app --port 8000
```

```bash
# Terminal E（WSL）：建立 MCP tunnel（8000）
cloudflared tunnel --url http://localhost:8000
# 記下輸出的公開網址，作為 <MCP_TUNNEL_URL>
```

### 成功判斷
- `pnpm run serve` 可提供靜態資源（本機 `http://localhost:4444`）
- `<FRONTEND_TUNNEL_URL>` 可成功取到前端資源
- MCP server 啟動後無立即錯誤中止，且監聽 `http://127.0.0.1:8000`
- `cloudflared` 產生 `<MCP_TUNNEL_URL>`
- 在 ChatGPT Connector 端設定 `<MCP_TUNNEL_URL>`
- 若 `<FRONTEND_TUNNEL_URL>` 改變，已重新執行 `BASE_URL=<FRONTEND_TUNNEL_URL> pnpm run build`

## 實驗 2.2：驗證特定範例功能與 UI 渲染
### 目的
- 驗證 Solar system 範例在實際互動中可正常渲染 UI，並確認關鍵元件與更新行為。

### 前置需求
- 已完成實驗 2.1，且 `pnpm run serve` 與 MCP server 持續運行
- `cloudflared` tunnel 持續運行，並取得 `<FRONTEND_TUNNEL_URL>` 與 `<MCP_TUNNEL_URL>`
- 已在 ChatGPT 端設定連到 `<MCP_TUNNEL_URL>` 的 App/Connector

### 步驟
```bash
# 保持 pnpm 靜態資源服務、MCP server、雙 tunnel 都持續執行
# 在 ChatGPT 載入 Solar system 對應 UI
# 在 UI 觸發一次會回傳/更新行星資料的操作
```

```bash
# 再觸發至少一次互動，觀察畫面是否完成二次渲染
```

### 成功判斷
- UI 可正常載入且無空白或崩潰
- 可見 Solar system 主要視覺元件（例如行星/軌道相關元素）
- 互動後畫面有可觀察的更新，且無明顯錯誤
- 已保存測試截圖作為 UI 渲染與互動成功證據

## 失敗排查
- 進入路徑錯誤：確認進入 `openai-apps-sdk-examples/solar-system_server_python`，且啟動檔為 `main.py`
- `pnpm run build` 或 `pnpm run serve` 失敗：確認 Node.js / pnpm 版本與依賴安裝狀態
- `uv sync` 失敗：確認 Python 版本、網路與套件來源設定
- MCP server 啟動失敗：確認 `OPENAI_API_KEY`、依賴是否完整安裝
- ChatGPT 連不到 MCP：確認 `cloudflared tunnel --url http://localhost:8000` 正常，且 Connector 使用 `<MCP_TUNNEL_URL>`
- UI 無法渲染：確認 `cloudflared tunnel --url http://localhost:4444` 正常，且已用 `BASE_URL=<FRONTEND_TUNNEL_URL> pnpm run build` 重建
- 互動後無更新：確認請求是否送達 server，並檢查回應格式是否符合範例預期

## 參考連結
- https://github.com/openai/openai-apps-sdk-examples

## 測試截圖
- UI 渲染結果：`./solar-system-ui-render-chatgpt.png`
- MCP 連線與回應結果：`./solar-system-mcp-connector-response.png`
