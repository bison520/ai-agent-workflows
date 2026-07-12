---
---
# 台灣發票自動整理 + 記帳 workflow

## 說明
自動從電子郵件擷取電子發票，解析發票內容並記錄到 Google Sheets，支援財政部發票對獎，讓每月記帳輕鬆完成。

## 前置需求
- Gmail 或企業信箱（接收電子發票）
- Google Sheets（記帳表）
- n8n 自動化平台

## 設定步驟

1. **整理發票信箱**：建立專門收發票的信箱，或設定 Gmail 標籤過濾發票郵件
2. **建立記帳表格**：Google Sheets 欄位：日期、店家、品項、金額、稅額、統一編號、發票號碼
3. **匯入 Workflow**：匯入下方 JSON，填入信箱與 Sheet 設定
4. **設定排程**：每天檢查一次新發票
5. **手動核對**：每月月底檢查一次自動記錄是否正確

## Workflow JSON

```json
{
  "name": "台灣發票自動記帳",
  "nodes": [
    {"name": "每日觸發", "type": "n8n-nodes-base.scheduleTrigger", "params": {"rule": {"interval": [{"triggerAtHour": 8}]}}},
    {"name": "讀取發票郵件", "type": "n8n-nodes-base.gmail", "params": {"operation": "getAll", "filter": {"label": "發票", "after": "{{$now.minus({days:1})}}"}}},
    {"name": "解析發票內容", "type": "n8n-nodes-base.code", "params": {"js": "const body = $input.first().json.body; const amount = body.match(/總計[：:]\\s*(\\d+)/)?.[1]; const store = body.match(/店名[：:]\\s*(.+)/)?.[1]; const invoiceNo = body.match(/([A-Z]{2}-\\d{8})/)?.[1]; return {store, amount, invoiceNo};"}},
    {"name": "寫入記帳表", "type": "n8n-nodes-base.googleSheets", "params": {"operation": "append", "sheetId": "{{$credentials.accountingSheetId}}", "columns": {"日期": "{{$now.format('YYYY-MM-DD')}}", "店家": "{{$json.store}}", "金額": "{{$json.amount}}", "發票號碼": "{{$json.invoiceNo}}"}}},
    {"name": "月結統計", "type": "n8n-nodes-base.aggregate", "params": {"field": "金額", "operation": "sum", "groupBy": "月份"}}
  ],
  "connections": {
    "每日觸發": [{"node": "讀取發票郵件"}],
    "讀取發票郵件": [{"node": "解析發票內容"}],
    "解析發票內容": [{"node": "寫入記帳表"}]
  }
}
```

## 自訂建議
- 加入統一編號自動帶入功能（公司報帳用）
- 整合財政部發票對獎 API，自動對獎
- 每月自動產生費用報表 PDF
- 加入分類標籤（餐飲、交通、辦公用品等）

> 💡 建議搭配載具歸戶，發票自動匯入信箱更方便
