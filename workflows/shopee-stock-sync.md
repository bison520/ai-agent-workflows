---
---
# 蝦皮商品庫存同步 workflow

## 說明
自動同步蝦皮商品庫存數量，當庫存低於設定門檻時即時通知，並支援多平台庫存統一管理，避免超賣情況發生。

## 前置需求
- 蝦皮賣家帳號（API 權限）
- Google Sheets 或 Airtable（作為庫存主表）
- n8n 自動化平台

## 設定步驟

1. **取得蝦皮 API**：蝦皮賣家中心 → 開發者設定，取得商品管理 API 權限
2. **建立庫存主表**：在 Google Sheets 建立商品清單：SKU、商品名稱、庫存數量、安全庫存門檻
3. **設定同步頻率**：建議每 30 分鐘同步一次
4. **匯入 Workflow**：匯入下方 JSON，填入 API 金鑰與 Sheet ID
5. **設定門檻**：在庫存表設定每項商品的安全庫存門檻

## Workflow JSON

```json
{
  "name": "蝦皮庫存同步",
  "nodes": [
    {"name": "定時觸發", "type": "n8n-nodes-base.scheduleTrigger", "params": {"interval": 30}},
    {"name": "讀取庫存主表", "type": "n8n-nodes-base.googleSheets", "params": {"operation": "read", "sheetId": "{{$credentials.stockSheetId}}"}},
    {"name": "比對蝦皮庫存", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://partner.shopeemobile.com/api/v1/items", "method": "GET", "qs": {"item_sku": "{{$json.SKU}}"}}},
    {"name": "檢查差異", "type": "n8n-nodes-base.if", "params": {"condition": "{{$json.localStock}} !== {{$json.shopeeStock}}"}},
    {"name": "更新蝦皮庫存", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://partner.shopeemobile.com/api/v1/item/update_stock", "method": "POST", "body": {"item_id": "{{$json.itemId}}", "stock": "{{$json.localStock}}"}}},
    {"name": "低庫存警報", "type": "n8n-nodes-base.if", "params": {"condition": "{{$json.localStock}} < {{$json.safetyStock}}"}},
    {"name": "發送通知", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://notify-api.line.me/api/notify", "message": "⚠️ 低庫存警報！{{$json.itemName}} 剩 {{$json.localStock}} 件"}}
  ],
  "connections": {
    "定時觸發": [{"node": "讀取庫存主表"}],
    "讀取庫存主表": [{"node": "比對蝦皮庫存"}],
    "比對蝦皮庫存": [{"node": "檢查差異"}],
    "檢查差異": [{"true": "更新蝦皮庫存"}],
    "更新蝦皮庫存": [{"node": "低庫存警報"}],
    "低庫存警報": [{"true": "發送通知"}]
  }
}
```

## 自訂建議
- 支援多蝦皮帳號同步
- 加入進貨後自動更新庫存功能
- 整合供應商自動下單（庫存低於門檻自動發採購單）

> 💡 建議設定安全庫存為 10-20 件，避免熱銷商品來不及補貨
