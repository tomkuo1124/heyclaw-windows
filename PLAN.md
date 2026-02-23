# HeyClaw Windows - 實作計畫

> Windows 版 AI 語音輸入工具，參考 xdite 的 heyclaw (macOS)

## 🎯 專案目標

製作一個 Windows 系統的語音轉文字工具，具備以下功能：
1. **語音聽寫** - 快捷鍵錄音 → 轉文字 → 自動輸入到游標位置
2. **LLM 後處理** - 修正錯字、加標點、口語轉書面語
3. **Clawdbot 整合** - 語音指令送給 AI 助手

---

## 📐 技術架構

### 框架選擇：Tauri v2

| 項目 | 選擇 | 理由 |
|------|------|------|
| 桌面框架 | Tauri v2 | 輕量（~20MB）、Rust 後端、跨平台 |
| 前端 | React + TypeScript | 現代化、生態豐富 |
| 建置工具 | Vite | 快速開發體驗 |

### 核心依賴（Rust）

| Crate | 用途 | 備註 |
|-------|------|------|
| `cpal` | 音訊錄製 | 跨平台音訊 I/O |
| `enigo` | 模擬鍵盤輸入 | 跨平台，Windows 支援良好 |
| `hound` | WAV 編碼 | 將錄音轉為 Whisper 可接受格式 |
| `reqwest` | HTTP 請求 | 呼叫 Groq API |
| `serde` | 序列化 | JSON 處理 |

### Tauri 插件

| 插件 | 用途 |
|------|------|
| `tauri-plugin-global-shortcut` | 全域快捷鍵監聽 |
| `tauri-plugin-shell` | 系統命令執行 |
| `tauri-plugin-notification` | 系統通知 |
| `tauri-plugin-store` | 本地設定儲存 |

### 外部 API

| 服務 | 用途 | 模型 |
|------|------|------|
| Groq Whisper API | 語音轉文字 | whisper-large-v3-turbo |
| Groq LLM API | 後處理修正 | llama-3.3-70b-versatile |
| Clawdbot | AI 助手 | localhost:18789 |

---

## 🏗️ 系統架構圖

```
┌─────────────────────────────────────────────────────────┐
│                      使用者介面                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  系統托盤   │  │ 錄音 Overlay │  │  設定視窗   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    Tauri 後端 (Rust)                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ 快捷鍵監聽  │  │  音訊錄製   │  │  輸入模擬   │     │
│  │ (global-    │  │   (cpal)    │  │  (enigo)    │     │
│  │  shortcut)  │  │             │  │             │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                           │                             │
│                           ▼                             │
│  ┌─────────────────────────────────────────────────┐   │
│  │              API 整合模組 (reqwest)              │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐   │   │
│  │  │  Whisper  │  │  Groq LLM │  │ Clawdbot  │   │   │
│  │  │   API     │  │    API    │  │   API     │   │   │
│  │  └───────────┘  └───────────┘  └───────────┘   │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## 📋 功能規格

### 1. 語音聽寫（核心功能）

**觸發方式：** `Alt + Space`（按住錄音，放開結束）

**流程：**
```
1. 使用者按下 Alt+Space
2. 顯示錄音中 Overlay（小藥丸狀）
3. 開始錄音（cpal 捕捉麥克風）
4. 使用者放開按鍵
5. 停止錄音，轉為 WAV 格式
6. 呼叫 Groq Whisper API 轉文字
7. （可選）呼叫 LLM 後處理
8. 使用 enigo 模擬鍵盤輸入文字
9. 隱藏 Overlay
```

**錄音規格：**
- 格式：WAV (PCM)
- 取樣率：16000 Hz
- 位元深度：16-bit
- 聲道：單聲道 (Mono)

### 2. LLM 後處理（可選）

**功能：**
- 修正語音辨識錯字
- 自動加標點符號
- 口語轉書面語（可選）

**Prompt 範例：**
```
請修正以下語音轉文字的結果，修正錯字並加上適當標點：
"{transcribed_text}"
只輸出修正後的文字，不要解釋。
```

### 3. 語音指揮 Clawdbot

**觸發方式：** `Alt + F1`

**流程：**
```
1. 使用者按下 Alt+F1
2. 錄音 → 轉文字
3. 將文字送到 Clawdbot API (localhost:18789)
4. 顯示聊天視窗
5. 顯示 AI 回應
```

### 4. 聊天視窗

**觸發方式：** `F1` 切換顯示/隱藏

**功能：**
- 浮動視窗，置頂顯示
- 顯示對話歷史
- 可手動輸入文字

---

## 🚀 開發階段

### Phase 1：基礎建設（Week 1）

**預估工時：** 8-12 小時

**目標：** 建立專案骨架，驗證核心技術可行性

- [ ] 初始化 Tauri v2 專案（1h）
- [ ] 設定 React + TypeScript + Vite（1h）
- [ ] 實作系統托盤（2h）
- [ ] 測試全域快捷鍵（2h）
- [ ] 測試 cpal 錄音（3h）
- [ ] 測試 enigo 輸入（2h）

**交付物：**
- 可執行的空殼程式
- 系統托盤圖示
- 快捷鍵可觸發 console log

**💡 參考 Handy 的實作可節省約 30% 時間**

### Phase 2：語音轉文字（Week 2）🎯 MVP

**預估工時：** 10-15 小時

**目標：** 完成核心語音聽寫功能

- [ ] 實作錄音模組（開始/停止/取消）（3h）
- [ ] WAV 編碼（2h）
- [ ] 串接 Groq Whisper API（3h）
- [ ] 實作文字輸入（enigo）（2h）
- [ ] 錄音狀態 Overlay UI（3h）
- [ ] 整合測試（2h）

**交付物：**
- Alt+Space 可錄音並輸入文字
- 顯示錄音中狀態

**驗收標準：**
- 中文辨識準確率 > 90%
- 延遲 < 3 秒（放開按鍵到文字出現）

### Phase 3：後處理與優化（Week 3）

**預估工時：** 8-10 小時

**目標：** 加入 LLM 後處理，優化體驗

- [ ] 串接 Groq LLM API（2h）
- [ ] 後處理開關設定（1h）
- [ ] 錯誤處理與重試（2h）
- [ ] 設定介面（API Key 等）（3h）
- [ ] 語言選擇（zh-TW/zh-CN/en）（1h）

**交付物：**
- 可選的 LLM 後處理
- 基本設定頁面

### Phase 4：Clawdbot 整合（Week 4）

**預估工時：** 10-12 小時

**目標：** 整合 Clawdbot 語音助手

- [ ] Alt+F1 語音指令（2h）
- [ ] 聊天視窗 UI（4h）
- [ ] Clawdbot API 串接（3h）
- [ ] F1 切換視窗（1h）
- [ ] 對話歷史管理（2h）

**交付物：**
- 完整的語音助手功能

### Phase 5：打包與發布（Week 5）

**預估工時：** 6-8 小時

**目標：** 準備發布

- [ ] Windows 安裝檔打包（2h）
- [ ] 自動更新機制（2h）
- [ ] 文件撰寫（2h）
- [ ] GitHub Release（1h）

---

### 📊 總工時估算

| Phase | 工時 |
|-------|------|
| Phase 1 | 8-12h |
| Phase 2 (MVP) | 10-15h |
| Phase 3 | 8-10h |
| Phase 4 | 10-12h |
| Phase 5 | 6-8h |
| **總計** | **42-57h** |

> 💡 參考 Handy 原始碼可縮短 20-30% 時間

---

## 📁 專案結構

```
heyclaw-windows/
├── src/                    # React 前端
│   ├── components/
│   │   ├── Overlay.tsx     # 錄音狀態 overlay
│   │   ├── ChatWindow.tsx  # 聊天視窗
│   │   └── Settings.tsx    # 設定頁面
│   ├── hooks/
│   │   └── useRecording.ts # 錄音狀態管理
│   ├── App.tsx
│   └── main.tsx
├── src-tauri/              # Rust 後端
│   ├── src/
│   │   ├── main.rs
│   │   ├── audio.rs        # 錄音模組
│   │   ├── whisper.rs      # Whisper API
│   │   ├── llm.rs          # LLM 後處理
│   │   ├── input.rs        # 鍵盤輸入模擬
│   │   └── clawdbot.rs     # Clawdbot API
│   ├── Cargo.toml
│   └── tauri.conf.json
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

---

## ⚙️ 設定項目

```json
{
  "groq_api_key": "gsk_...",
  "whisper_model": "whisper-large-v3-turbo",
  "llm_model": "llama-3.3-70b-versatile",
  "llm_enabled": true,
  "language": "zh-TW",
  "shortcuts": {
    "dictation": "Alt+Space",
    "clawdbot": "Alt+F1",
    "toggle_chat": "F1"
  },
  "clawdbot_url": "http://localhost:18789"
}
```

---

## 🔗 參考資源

### 文件
- [Tauri v2 文件](https://v2.tauri.app/)
- [cpal - Rust 音訊](https://github.com/RustAudio/cpal)
- [enigo - 輸入模擬](https://github.com/enigo-rs/enigo)
- [Groq API 文件](https://console.groq.com/docs)

### 開源參考專案
| 專案 | 連結 | 說明 |
|------|------|------|
| **Handy** ⭐ | https://github.com/cjpais/Handy | 開源跨平台語音輸入，Tauri + Rust，支援離線 Whisper |
| heyclaw | (私有 repo by xdite) | macOS 語音輸入工具，整合 clawdbot |
| Typeless | https://typeless.com | 商業 AI 語音輸入工具 |

> **💡 Handy 是最佳參考！** 架構相似、開源、支援 Windows，可直接學習其 cpal + enigo 實作。

---

## 🚀 快速開始

### 環境需求
- Node.js 18+
- Rust 1.70+
- Windows 10/11

### 開發指令
```bash
# 1. Clone 專案
git clone https://github.com/tomkuo1124/heyclaw-windows.git
cd heyclaw-windows

# 2. 安裝前端依賴
yarn install

# 3. 開發模式
yarn tauri dev

# 4. 建構發布版
yarn tauri build
```

---

## ⚠️ 風險評估與備案

| 風險 | 可能性 | 影響 | 備案 |
|------|--------|------|------|
| Groq API 限流/不穩定 | 中 | 高 | 備案 1：本地 Whisper（參考 Handy）<br>備案 2：OpenAI Whisper API |
| enigo Windows 相容性問題 | 低 | 高 | 備案：使用 Windows API `SendInput` 直接呼叫 |
| 全域快捷鍵衝突 | 中 | 中 | 提供自訂快捷鍵設定 |
| 麥克風權限被拒 | 中 | 高 | 顯示清楚的權限引導 UI |
| LLM 後處理延遲 | 中 | 低 | 提供「跳過後處理」選項 |

---

## 🎯 MVP 定義

### Phase 1 結束時（Week 1）可 Demo：
- ✅ 系統托盤圖示顯示
- ✅ 按 Alt+Space 觸發事件（console log）
- ✅ 可錄製麥克風聲音並播放
- ✅ 可模擬輸入文字到記事本

### Phase 2 結束時（Week 2）可 Demo：
- ✅ **完整語音聽寫流程**：Alt+Space → 說話 → 文字出現在游標
- ✅ 錄音中顯示小藥丸 Overlay
- ✅ 支援中/英文

### 最小可用版本 (MVP)：
> Phase 2 完成 = MVP 達成

---

## 🧪 測試策略

### 單元測試
| 模組 | 測試項目 |
|------|----------|
| audio.rs | 錄音開始/停止、WAV 編碼正確性 |
| whisper.rs | API 呼叫格式、錯誤處理 |
| input.rs | 特殊字元輸入、中文輸入 |

### 整合測試
| 場景 | 測試項目 |
|------|----------|
| 快樂路徑 | Alt+Space → 說「你好」→ 游標出現「你好」 |
| 長錄音 | 30 秒以上的語音 |
| 快速連按 | 連續多次 Alt+Space |
| 無網路 | API 失敗時的錯誤處理 |
| 權限問題 | 麥克風權限被拒時的提示 |

### 相容性測試
- Windows 10 (21H2+)
- Windows 11
- 不同麥克風裝置

---

## 📝 待確認事項

1. **Groq API Key** - 需要申請
2. **xdite 的 repo** - 是否公開可參考
3. **Windows 權限** - 全域快捷鍵是否需要 admin
4. **音訊裝置** - 麥克風選擇機制

---

## 👥 分工建議

| 角色 | 負責項目 |
|------|----------|
| 後端開發 | Rust 模組（audio, whisper, input） |
| 前端開發 | React UI（Overlay, ChatWindow, Settings） |
| 整合測試 | 端到端測試、打包發布 |

---

*文件版本：v1.1*
*更新日期：2026-02-23*
*補充：兔兔 🐰 - Handy 參考、Quick Start、風險評估、MVP 定義、測試策略*
