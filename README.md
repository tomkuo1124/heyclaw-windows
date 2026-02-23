# HeyClaw Windows

> Windows 版 AI 語音輸入工具

🎤 按下快捷鍵，說話，文字自動輸入到游標位置。

## ✨ 功能

- **語音聽寫** - Alt+Space 錄音 → 轉文字 → 打到游標位置
- **LLM 後處理** - 修正錯字、加標點、口語轉書面語
- **Clawdbot 整合** - Alt+F1 語音指令送給 AI 助手

## 🛠️ 技術架構

- **Tauri v2** + React + TypeScript
- **Groq Whisper API** - 語音轉文字
- **Groq LLM API** - 後處理修正
- **cpal** - 音訊錄製
- **enigo** - 模擬鍵盤輸入

## 📋 實作計畫

詳見 [PLAN.md](./PLAN.md)

## 🙏 致謝

參考 [xdite](https://github.com/xdite) 的 heyclaw (macOS 版)

## 📝 License

MIT
