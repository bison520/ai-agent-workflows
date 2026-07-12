---
---
# LINE 群組訊息摘要 workflow

## 說明
自動將 LINE 群組每日對話整理成摘要，提取重要事項、待辦事項和決議事項，適合忙碌的上班族和團隊管理者快速掌握群組動態。

## 前置需求
- LINE Messaging API（群組機器人權限）
- OpenAI API Key（用於摘要生成）
- n8n 自動化平台

## 設定步驟

1. **建立 LINE Bot**：在 LINE Developers 建立 Messaging API，加入目標群組
2. **設定 Webhook**：讓機器人接收群組所有訊息並記錄
3. **準備儲存**：建立臨時資料庫或 Google Sheet 暫存每日訊息
4. **匯入 Workflow**：匯入下方 JSON，填入 LINE Token 與 OpenAI Key
5. **設定摘要時間**：建議每天晚上 9 點產生當日摘要

## Workflow JSON

```json
{
  "name": "LINE群組每日摘要",
  "nodes": [
    {"name": "每日觸發", "type": "n8n-nodes-base.scheduleTrigger", "params": {"rule": {"interval": [{"triggerAtHour": 21}]}}},
    {"name": "讀取今日訊息", "type": "n8n-nodes-base.googleSheets", "params": {"operation": "read", "filter": {"日期": "{{$now.format('YYYY-MM-DD')}}"}}},
    {"name": "AI生成摘要", "type": "n8n-nodes-base.openAi", "params": {"model": "gpt-4o-mini", "system": "你是一個會議摘要助手，請將以下LINE群組對話整理成：1.重要事項 2.待辦事項 3.決議事項。用繁體中文，簡潔明確。", "prompt": "{{$json.messages}}"}},
    {"name": "推播摘要", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://api.line.me/v2/bot/message/push", "method": "POST", "body": {"to": "{{$json.groupId}}", "messages": [{"type": "text", "text": "📋 今日群組摘要\\n\\n{{$json.summary}}"}]}}},
    {"name": "歸檔記錄", "type": "n8n-nodes-base.googleSheets", "params": {"operation": "append", "sheetId": "{{$credentials.archiveSheetId}}"}}
  ],
  "connections": {
    "每日觸發": [{"node": "讀取今日訊息"}],
    "讀取今日訊息": [{"node": "AI生成摘要"}],
    "AI生成摘要": [{"node": "推播摘要"}, {"node": "歸檔記錄"}]
  }
}
```

## 自訂建議
- 設定關鍵字高亮（如「截止日」「重要」「確認」等）
- 加入待辦事項自動指派功能
- 支援週報摘要（每週五產生本週總結）
- 可調整為即時摘要（每 100 則訊息產生一次）

> 💡 建議先取得群組管理員同意再啟用，注意隱私合規
