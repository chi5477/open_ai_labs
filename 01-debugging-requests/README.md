# Debugging requests

## 範圍說明
- 本文件涵蓋實驗 1.1～1.2，聚焦 API 回應 headers、request ID、rate limit headers 與自訂 X-Client-Request-Id。

## 語言/版本
- Shell（curl）

## 實驗 1.1：檢視回應 headers 與 request ID
### 目的
- 觀察回應 headers 中的 `x-request-id` 與 API meta / rate limit 相關欄位（實際欄位以回應為準）。

### 前置需求
- 已設定 `OPENAI_API_KEY` 環境變數
- 可連線到網路

### 步驟
```bash
curl -i https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

```bash
curl -D headers.txt -o /dev/null https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

### 成功判斷
- 回應 headers 中出現 `x-request-id`
- 若出現 `x-ratelimit-*` 或 `openai-*` 欄位，已記錄其值

## 實驗 1.2：自訂 X-Client-Request-Id
### 目的
- 送出自訂 request ID，驗證格式限制並保留可追蹤的識別值。

### 前置需求
- 已設定 `OPENAI_API_KEY` 環境變數
- 可連線到網路

### 步驟
```bash
curl -i https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "X-Client-Request-Id: 123e4567-e89b-12d3-a456-426614174000"
```

### 成功判斷
- 回應非 400，表示自訂 ID 格式可被接受
- 已在本地記錄自訂 ID 以便後續比對或支援排查

## 失敗排查
- 401：API key 無效或未設定
- 429：觸發速率限制，請查看 `x-ratelimit-*` headers
- 400：`X-Client-Request-Id` 含非 ASCII 字元或長度超過 512

## 參考連結
- https://platform.openai.com/docs/
