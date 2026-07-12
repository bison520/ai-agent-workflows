# LINE 官方帳號自動回覆 + AI 客服 workflow

## 說明
結合 LINE Messaging API 與 AI（如 OpenAI），讓 LINE 官方帳號能自動回覆常見問題，並在無法回答時轉接真人客服，提升顧客服務效率。

## 前置需求
- LINE Official Account 帳號（已升級為開發者模式）
- LINE Messaging API Channel Access Token
- OpenAI API Key（或其他 LLM 服務）
- n8n 自動化平台

## 設定步驟

1. **LINE 開發者設定**：至 [LINE Developers Console](https://developers.line.biz/) 建立 Messaging API channel，取得 Channel Access Token 與 Channel Secret
2. **設定 Webhook URL**：將 n8n 的 Webhook URL 填入 LINE 的 Webhook 設定（格式：`https://your-n8n.com/webhook/line-reply`）
3. **準備 AI 知識庫**：整理 20-30 條常見問答，作為 AI 的 system prompt
4. **匯入 Workflow**：匯入下方 JSON，填入 LINE Token 與 OpenAI Key
5. **測試對話**：用另一個 LINE 帳號發送訊息測試

## Workflow JSON

```json
{
  "name": "LINE AI自動回覆客服",
  "nodes": [
    {"name": "LINE Webhook", "type": "n8n-nodes-base.webhook", "params": {"path": "line-reply", "method": "POST"}},
    {"name": "解析訊息", "type": "n8n-nodes-base.set", "params": {"userId": "{{$json.events[0].source.userId}}", "text": "{{$json.events[0].message.text}}"}},
    {"name": "AI判斷", "type": "n8n-nodes-base.openAi", "params": {"model": "gpt-4o-mini", "system": "你是台灣小商家的客服，用繁體中文回答產品相關問題。若無法回答，回覆『需要人工客服』", "prompt": "{{$json.text}}"}},
    {"name": "判斷轉接", "type": "n8n-nodes-base.if", "params": {"condition": "{{$json.aiReply}} contains 人工客服"}},
    {"name": "回覆AI答案", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://api.line.me/v2/bot/message/push", "method": "POST"}},
    {"name": "通知真人客服", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://api.line.me/v2/bot/message/push", "method": "POST", "body": {"message": "⚠️ 客戶需要人工協助：{{$json.text}}"}}
  ],
  "connections": {"LINE Webhook": [{"node": "解析訊息"}], "解析訊息": [{"node": "AI判斷"}], "AI判斷": [{"node": "判斷轉接"}], "判斷轉接": [{"true": "通知真人客服", "false": "回覆AI答案"}]}
}
```

## 自訂建議
- 將產品目錄加入 AI 的 system prompt，回答更精準
- 設定營業時間外自動回覆「已收到訊息，明日回覆」
- 整合訂單查詢，讓客戶直接查物流狀態

> 💡 建議先準備至少 20 組 QA 測試，確保 AI 回覆品質穩定
