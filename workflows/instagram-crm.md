---
---
# IG 潛在客戶自動收集 + CRM workflow

## 說明
自動收集 Instagram 貼文留言和私訊中的潛在客戶資訊，整理到 Google Sheets 作為簡易 CRM，方便後續追蹤和行銷。

## 前置需求
- Instagram 商業帳號（已連結 Facebook 粉專）
- Facebook Developer App（Instagram Graph API 權限）
- Google Sheets
- n8n 自動化平台

## 設定步驟

1. **開通 IG API**：在 Facebook Developer Console 申請 `instagram_basic` 和 `instagram_manage_messages` 權限
2. **建立 CRM 表格**：在 Google Sheets 建立欄位：用戶名、來源（留言/私訊）、內容、日期、標籤、跟進狀態
3. **設定關鍵字篩選**：定義觸發關鍵字如「價格」「怎麼買」「哪裡買」「想要」等
4. **匯入 Workflow**：匯入下方 JSON 並填入相關金鑰
5. **測試**：在 IG 貼文留言測試關鍵字觸發

## Workflow JSON

```json
{
  "name": "IG潛在客戶CRM",
  "nodes": [
    {"name": "排程檢查", "type": "n8n-nodes-base.scheduleTrigger", "params": {"interval": 10}},
    {"name": "取得IG留言", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://graph.facebook.com/v18.0/{{$mediaId}}/comments", "method": "GET", "headers": {"Authorization": "Bearer {{$credentials.igToken}}"}}},
    {"name": "關鍵字篩選", "type": "n8n-nodes-base.filter", "params": {"conditions": [{"field": "text", "regex": "(價格|怎麼買|哪裡買|想要|多少錢|如何訂購)"}]}},
    {"name": "整理客戶資料", "type": "n8n-nodes-base.set", "params": {"username": "{{$json.username}}", "content": "{{$json.text}}", "source": "IG留言", "date": "{{$now}}"}},
    {"name": "寫入CRM", "type": "n8n-nodes-base.googleSheets", "params": {"operation": "append", "sheetId": "{{$credentials.crmSheetId}}"}},
    {"name": "通知負責人", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://notify-api.line.me/api/notify", "message": "🎯 新潛在客戶: {{$json.username}} 詢問: {{$json.content}}"}}
  ],
  "connections": {"排程檢查": [{"node": "取得IG留言"}], "取得IG留言": [{"node": "關鍵字篩選"}], "關鍵字篩選": [{"node": "整理客戶資料"}], "整理客戶資料": [{"node": "寫入CRM"}, {"node": "通知負責人"}]}
}
```

## 自訂建議
- 加入「已跟進」「已成交」等標籤管理客戶狀態
- 設定自動私訊回覆，引導客戶到官網或 LINE
- 每週自動統計新增潛在客戶數量

> 💡 建議每週檢視 CRM 表格，及時跟進高意向客戶
