---
---
# FB 廣告成效自動報表 workflow

## 說明
自動收集 Facebook 廣告投放數據，生成每週成效報表並推播通知，包含花費、觸及人數、點擊率、轉換成本等關鍵指標，優化廣告預算配置。

## 前置需求
- Facebook 廣告帳號（管理員權限）
- Facebook Marketing API（App 已開通 ads_read 權限）
- Google Sheets 或 Google Data Studio
- n8n 自動化平台

## 設定步驟

1. **開通 Marketing API**：在 Facebook Developer Console 申請 `ads_read` 和 `ads_management` 權限
2. **取得廣告帳號 ID**：格式如 `act_12345678`，在廣告管理員設定中找到
3. **建立報表模板**：Google Sheets 預設欄位：日期、廣告名稱、花費、觸及、點擊、CTR、CPA
4. **匯入 Workflow**：匯入下方 JSON，填入 Facebook Token 與廣告帳號 ID
5. **設定報表時間**：建議每週一早上 9 點產生上週報表

## Workflow JSON

```json
{
  "name": "FB廣告成效報表",
  "nodes": [
    {"name": "每週觸發", "type": "n8n-nodes-base.scheduleTrigger", "params": {"rule": {"interval": [{"triggerAtHour": 9, "triggerAtDay": "Monday"}]}}},
    {"name": "取得廣告數據", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://graph.facebook.com/v18.0/{{$credentials.adAccountId}}/insights", "method": "GET", "qs": {"fields": "campaign_name,spend,reach,clicks,ctr,cpc,actions", "time_range": "{\"since\":\"{{$now.minus({weeks:1}).format('YYYY-MM-DD')}}\",\"until\":\"{{$now.minus({days:1}).format('YYYY-MM-DD')}}\"}", "access_token": "{{$credentials.fbToken}}"}}},
    {"name": "計算關鍵指標", "type": "n8n-nodes-base.code", "params": {"js": "return $input.all().map(item => ({ campaign: item.json.campaign_name, spend: item.json.spend, reach: item.json.reach, clicks: item.json.clicks, ctr: item.json.ctr, cpa: (item.json.spend / item.json.actions[0].value).toFixed(2) }));"}},
    {"name": "寫入報表", "type": "n8n-nodes-base.googleSheets", "params": {"operation": "append", "sheetId": "{{$credentials.reportSheetId}}"}},
    {"name": "生成摘要", "type": "n8n-nodes-base.code", "params": {"js": "const data = $input.all(); const totalSpend = data.reduce((s,i) => s+parseFloat(i.json.spend), 0); const avgCtr = (data.reduce((s,i) => s+parseFloat(i.json.ctr), 0) / data.length).toFixed(2); return {summary: `📊 上週廣告報表\\n總花費: NT$${totalSpend}\\n平均CTR: ${avgCtr}%\\n廣告組數: ${data.length}`};"}},
    {"name": "推播通知", "type": "n8n-nodes-base.httpRequest", "params": {"url": "https://notify-api.line.me/api/notify", "message": "{{$json.summary}}"}}
  ],
  "connections": {
    "每週觸發": [{"node": "取得廣告數據"}],
    "取得廣告數據": [{"node": "計算關鍵指標"}],
    "計算關鍵指標": [{"node": "寫入報表"}, {"node": "生成摘要"}],
    "生成摘要": [{"node": "推播通知"}]
  }
}
```

## 自訂建議
- 加入 ROAS 計算（廣告投資報酬率）
- 設定花費異常警報（日花費超過預算 20% 自動通知）
- 每月自動比較上月同期數據，標示成長/衰退趨勢
- 整合 Google Data Studio 產生視覺化圖表

> 💡 建議每週檢視報表，及時調整表現不佳的廣告組合
