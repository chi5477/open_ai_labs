# OpenAI Apps SDK Examples

## 範圍說明
- 本文件涵蓋實驗 2.1～2.3，聚焦 `openai/openai-apps-sdk-examples` 的 Solar system 與 authenticated server Python 範例，並驗證 UI 渲染與 OAuth 連線流程。

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

## 實驗 2.3：Authenticated server OAuth 流程驗證
### 目的
- 驗證 `authenticated_server_python` 在 ChatGPT 中可觸發 OAuth 連線、完成授權並渲染受保護工具 UI。

### 前置需求
- 已完成實驗 2.1 的前端資源與雙 tunnel 設定
- 已設定 `authenticated_server_python/.env`
- 已有可用的 Auth0 tenant，且完成必要設定（DCR、callback URL、API audience）

### 步驟
```bash
cd openai-apps-sdk-examples/authenticated_server_python
uv sync
uv run uvicorn main:app --port 8000
```

```bash
# 在另一個終端啟動 MCP tunnel（如已啟動可略過）
cloudflared tunnel --url http://localhost:8000
```

```bash
# 以未帶 token 呼叫受保護工具，確認 mcp/www_authenticate 內容格式
curl -sS http://127.0.0.1:8000/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  --data '{"jsonrpc":"2.0","id":"test-1","method":"tools/call","params":{"name":"see_past_orders","arguments":{}}}' \
| sed -n 's/^data: //p' \
| jq -r '.result._meta["mcp/www_authenticate"][0]'
```

### 成功判斷
- ChatGPT 顯示 connector 連線/授權提示，且可進入授權頁
- 授權完成後可成功呼叫 `see_past_orders`
- 已渲染 past orders widget，畫面可見訂單資料
- `mcp/www_authenticate` 格式為 `error="...", error_description="...", resource_metadata="..."`

## 失敗排查
- 進入路徑錯誤：確認進入 `openai-apps-sdk-examples/solar-system_server_python`，且啟動檔為 `main.py`
- `pnpm run build` 或 `pnpm run serve` 失敗：確認 Node.js / pnpm 版本與依賴安裝狀態
- `uv sync` 失敗：確認 Python 版本、網路與套件來源設定
- MCP server 啟動失敗：確認 `OPENAI_API_KEY`、依賴是否完整安裝
- ChatGPT 連不到 MCP：確認 `cloudflared tunnel --url http://localhost:8000` 正常，且 Connector 使用 `<MCP_TUNNEL_URL>`
- UI 無法渲染：確認 `cloudflared tunnel --url http://localhost:4444` 正常，且已用 `BASE_URL=<FRONTEND_TUNNEL_URL> pnpm run build` 重建
- 互動後無更新：確認請求是否送達 server，並檢查回應格式是否符合範例預期
- Auth0 顯示 `Callback URL mismatch`：確認應用程式允許 `https://chatgpt.com/connector_platform_oauth_redirect`
- Auth0 顯示 `not authorized to access resource server`：確認 API audience 與 `RESOURCE_SERVER_URL` 對應一致
- tunnel URL 變動後登入失敗：更新 `.env` 中 `RESOURCE_SERVER_URL` 後重啟 server
- `mcp/www_authenticate` 解析失敗：確認 challenge 參數間有逗號分隔

## 參考連結
- https://github.com/openai/openai-apps-sdk-examples

## 測試截圖
- UI 渲染結果：`./solar-system-ui-render-chatgpt.png`
- MCP 連線與回應結果：`./solar-system-mcp-connector-response.png`
- Auth connector 連線提示：`./authenticated-server-connector-consent-prompt.png`
- Auth connector 權限資訊：`./authenticated-server-connector-permissions-dialog.png`
- Authenticated 搜尋 widget：`./authenticated-server-search-widget-rendered.png`
- Authenticated past orders widget：`./authenticated-server-past-orders-widget-rendered.png`
