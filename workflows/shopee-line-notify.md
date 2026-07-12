# 蝦皮訂單自動通知 workflow

## 說明
自動監控蝦皮新訂單，即時透過 LINE 推播通知賣家，包含訂單編號、商品名稱、買家資訊與金額，讓你隨時掌握出貨進度。

## 前置需求
- 蝦皮賣家帳號（已開通 API 或使用蝦皮後台 Webhook）
- LINE Notify 權杖（至 [LINE Notify](https://notify-bot.line.me/) 綁定）
- n8n 自動化平台（自架或使用 n8n Cloud）

## 設定步驟

1. **取得蝦皮 API 金鑰**：進入蝦皮賣家中心 → 服務市場 → 開發者設定，取得 Partner ID 與 Secret Key
2. **申請 LINE Notify Token**：登入 LINE Notify，選擇「個人聊天室」產生權杖
3. **匯入 Workflow**：在 n8n 中匯入下方 JSON，填入你的金鑰
4. **設定排程**：建議每 5 分鐘執行一次檢查新訂單
5. **測試**：建立一筆測試訂單確認通知正常

## Workflow JSON

```json
{
  "name": "蝦皮訂單LINE通知",
  "nodes": [
    {"name": "排程觸發", "type": "n8n-nodes-base.scheduleTrigger", "params": {"interval": 5}},
    {"name": "取得蝦皮訂單", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://partner.shopeemobile.com/api/v1/orders", "method": "GET", "headers": {"Authorization": "Bearer {{$credentials.shopeeToken}}"}}},
    {"name": "篩選新訂單", "type": "n8n-nodes-base.filter", "params": {"conditions": [{"field": "status", "value": "UNPAID"}]}},
    {"name": "格式化訊息", "type": "n8n-nodes-base.set", "params": {"message": "🛒 新訂單！\\n訂單: {{$json.order_sn}}\\n商品: {{$json.item_name}}\\n金額: ${{$json.total_amount}}"}},
    {"name": "LINE推播", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://notify-api.line.me/api/notify", "method": "POST", "headers": {"Authorization": "Bearer {{$credentials.lineNotifyToken}}"}}}
  ],
  "connections": {"排程觸發": [{"node": "取得蝦皮訂單"}], "取得蝦皮訂單": [{"node": "篩選新訂單"}], "篩選新訂單": [{"node": "格式化訊息"}], "格式化訊息": [{"node": "LINE推播"}]}
}
```

## 自訂建議
- 可增加「已出貨」「已完成」等不同狀態的通知
- 搭配 Google Sheets 記錄所有訂單資料
- 加入金額門檻，僅通知大額訂單

> 💡 適合蝦皮中小型賣家，每天處理 10-100 筆訂單的店家最實用
