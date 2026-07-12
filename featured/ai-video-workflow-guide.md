---
---
# 🎬 AI 影片自動生成工作流 — 一句話出片指南

> 作者：一人公司研究所（鄭惠文）
> 更新：2026-07-12
> 授權：MIT（免費開源）

---

## 📌 這是什麼？

這是一個**全自動知識類解說影片生成工作流**。你只需要講一句話，AI Agent 就會自動完成：

- ✍️ 腳本撰寫
- 🎨 動畫場景設計（深色科技風卡片）
- 🎙️ 中文語音配音
- 📝 字幕自動燒錄
- 🎬 成片匯出（MP4）

**完全免費**，使用開源工具組合。

---

## 🔧 使用工具

| 工具 | 用途 | 授權 |
|------|------|------|
| **Manim CE** | 數學/科技風動畫場景 | MIT |
| **Edge TTS** | 微軟免費中文語音合成 | MIT |
| **FFmpeg** | 影片合成 + 字幕燒錄 | LGPL |
| **AI Agent** | 腳本生成 + 流程控制 | — |

---

## 🚀 快速開始

### 1. 準備環境

確保你的 AI Agent 環境已安裝：
- Python 3.10+
- FFmpeg
- Manim CE（`pip install manim`）
- Edge TTS（`pip install edge-tts`）

### 2. 下載工作流檔案

將本檔案（`ai-video-workflow-guide.md`）下載到你的電腦。

### 3. 給 AI Agent 下指令

將工作流檔案丟給你的 AI Agent，然後說：

```
按照附件檔案流程，幫我產出 90 秒的「當前主流 AI Agent 優缺點比較」短片
```

AI Agent 會自動：
1. 生成腳本（約 7 個場景）
2. 為每個場景建立 Manim 動畫
3. 用 Edge TTS 產生中文配音
4. 用 FFmpeg 合成最終影片

### 4. 批量出片

調整好模板後，換個 CSV 檔就能批量出片：

```csv
主題,時長(秒),風格
AI Agent比較,90,科技風
Python入門,60,簡約風
股市分析,120,數據風
```

---

## 🎯 工作流程圖

```
一句話指令
    ↓
[AI Agent] 腳本生成（7 場景 × ~8 秒/場景）
    ↓
[Manim CE] 動畫場景渲染（深色科技風卡片）
    ↓
[Edge TTS] 中文語音合成（自然語調）
    ↓
[FFmpeg] 音畫合成 + 字幕燒錄
    ↓
📹 成品 MP4（90 秒知識解說影片）
```

---

## 🎨 模板樣式

- **背景**：深色科技風（#09090b）
- **卡片**：毛玻璃效果 + 綠色強調線
- **字體**：Outfit + Noto Sans TC
- **動畫**：淡入淡出 + 卡片滑動
- **字幕**：白色，底部居中，黑色描邊

> 💡 所有元素都可以再調整！修改 Manim 場景參數就能換風格。

---

## ⚠️ 注意事項

1. **首次渲染較慢** — Manim 需要編譯 Python 場景，約 2-5 分鐘
2. **Edge TTS 需要網路** — 微軟語音合成是線上服務
3. **中文語音推薦** — `zh-TW-HsiaoChenNeural`（女聲）或 `zh-TW-YunJheNeural`（男聲）
4. **影片解析度** — 預設 1080p，可在 Manim 參數中調整
5. **字幕位置** — 預設 `y=h-140+10px`，可微調

---

## 📦 輸出規格

| 項目 | 規格 |
|------|------|
| 解析度 | 1920×1080 (1080p) |
| 幀率 | 30fps |
| 格式 | MP4 (H.264) |
| 音訊 | AAC 128kbps |
| 字幕 | 硬字幕（燒錄） |
| 時長 | 60-120 秒（可調） |

---

## 🔗 相關資源

- Manim CE 官方文件：https://docs.manim.community/
- Edge TTS 語音清單：https://github.com/rany2/edge-tts
- FFmpeg 文件：https://ffmpeg.org/documentation.html
- AI Agent Skill 市集：https://ai-dreams.net/

---

## 💬 常見問題

**Q: 可以用其他語言嗎？**
A: 可以！Edge TTS 支援 100+ 種語言，修改 `--voice` 參數即可。

**Q: 可以加背景音樂嗎？**
A: 可以。在 FFmpeg 合成階段加入 `-i bgm.mp3 -filter_complex amix=inputs=2:duration=first:dropout_transition=3` 即可混音。

**Q: 動畫風格可以換嗎？**
A: 可以。修改 Manim 場景的顏色、字體、動畫參數就能換風格。常見風格：科技風、簡約風、數據風、活潑風。

**Q: 要花多少錢？**
A: 完全免費。所有工具都是開源的，語音合成用的是微軟免費方案。

---

*由 AI Agent Skill 市集（ai-dreams.net）提供 · 一人公司研究所*
