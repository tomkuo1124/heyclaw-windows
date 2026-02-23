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

**目標：** 建立專案骨架，驗證核心技術可行性

- [ ] 初始化 Tauri v2 專案
- [ ] 設定 React + TypeScript + Vite
- [ ] 實作系統托盤
- [ ] 測試全域快捷鍵
- [ ] 測試 cpal 錄音
- [ ] 測試 enigo 輸入

**交付物：**
- 可執行的空殼程式
- 系統托盤圖示
- 快捷鍵可觸發 console log

### Phase 2：語音轉文字（Week 2）

**目標：** 完成核心語音聽寫功能

- [ ] 實作錄音模組（開始/停止/取消）
- [ ] WAV 編碼
- [ ] 串接 Groq Whisper API
- [ ] 實作文字輸入（enigo）
- [ ] 錄音狀態 Overlay UI

**交付物：**
- Alt+Space 可錄音並輸入文字
- 顯示錄音中狀態

### Phase 3：後處理與優化（Week 3）

**目標：** 加入 LLM 後處理，優化體驗

- [ ] 串接 Groq LLM API
- [ ] 後處理開關設定
- [ ] 錯誤處理與重試
- [ ] 設定介面（API Key 等）
- [ ] 語言選擇（zh-TW/zh-CN/en）

**交付物：**
- 可選的 LLM 後處理
- 基本設定頁面

### Phase 4：Clawdbot 整合（Week 4）

**目標：** 整合 Clawdbot 語音助手

- [ ] Alt+F1 語音指令
- [ ] 聊天視窗 UI
- [ ] Clawdbot API 串接
- [ ] F1 切換視窗

**交付物：**
- 完整的語音助手功能

### Phase 5：打包與發布（Week 5）

**目標：** 準備發布

- [ ] Windows 安裝檔打包
- [ ] 自動更新機制
- [ ] 文件撰寫
- [ ] GitHub Release

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

- [Tauri v2 文件](https://v2.tauri.app/)
- [cpal - Rust 音訊](https://github.com/RustAudio/cpal)
- [enigo - 輸入模擬](https://github.com/enigo-rs/enigo)
- [Groq API 文件](https://console.groq.com/docs)
- [heyclaw (原版 macOS)](https://github.com/xdite/heyclaw) ← 待確認是否公開

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

*文件版本：v1.0*
*更新日期：2026-02-23*
