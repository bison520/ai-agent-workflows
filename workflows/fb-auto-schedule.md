---
---
# FB 粉專貼文自動排程 workflow

## 說明
自動排程 Facebook 粉絲專頁貼文，支援一次上傳多則內容，按預設時間表自動發布，適合需要維持每日發文頻率的商家。

## 前置需求
- Facebook 粉絲專頁（管理員權限）
- Facebook Developer App（取得 Page Access Token）
- Google Sheets（作為內容管理表）
- n8n 自動化平台

## 設定步驟

1. **建立 Facebook App**：至 [developers.facebook.com](https://developers.facebook.com/) 建立 App，取得 Page Access Token（需 `pages_manage_posts` 權限）
2. **建立 Google Sheet**：建立表格包含欄位：日期、時間、內容、圖片URL、狀態
3. **匯入 Workflow**：匯入下方 JSON，填入 Token 與 Sheet ID
4. **填入內容**：在 Google Sheet 一次填入一週的貼文內容
5. **啟用排程**：啟用 workflow，每天自動檢查並發布

## Workflow JSON

```json
{
  "name": "FB粉專自動排程",
  "nodes": [
    {"name": "每日觸發", "type": "n8n-nodes-base.scheduleTrigger", "params": {"rule": {"interval": [{"triggerAtHour": 9}]}}},
    {"name": "讀取Google Sheet", "type": "n8n-nodes-base.googleSheets", "params": {"operation": "read", "sheetId": "{{$credentials.sheetId}}", "range": "貼文清單!A:E"}},
    {"name": "篩選今日貼文", "type": "n8n-nodes-base.filter", "params": {"conditions": [{"field": "日期", "value": "{{$now.format('YYYY-MM-DD')}}"}]}},
    {"name": "發佈FB貼文", "type": "n8n-nodes-base.facebookGraph", "params": {"resource": "post", "pageId": "{{$credentials.pageId}}", "message": "{{$json.內容}}", "imageUrl": "{{$json.圖片URL}}"}},
    {"name": "更新狀態", "type": "n8n-nodes-base.googleSheets", "params": {"operation": "update", "row": "{{$json.row}}", "column": "狀態", "value": "已發布"}}
  ],
  "connections": {"每日觸發": [{"node": "讀取Google Sheet"}], "讀取Google Sheet": [{"node": "篩選今日貼文"}], "篩選今日貼文": [{"node": "發佈FB貼文"}], "發佈FB貼文": [{"node": "更新狀態"}]}
}
```

## 自訂建議
- 搭配 AI 自動生成不同風格的貼文文案
- 設定不同時間發布（早上9點、中午12點、晚上8點）
- 加入成效追蹤，自動記錄每篇貼文的互動數

> 💡 最佳發文頻率：每天 1-2 則，避免連續發布造成觸及率下降
