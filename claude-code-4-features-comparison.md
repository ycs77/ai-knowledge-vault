# Claude Code 四大擴充功能比較

Slash Command / MCP Server / Skill / Subagent

---

## 快速總覽

| | Slash Command | MCP Server | Skill | Subagent |
|--|---------------|------------|-------|----------|
| **一句話說明** | 可重複使用的提示詞範本 | 連接外部服務的橋接器 | 自動載入的知識庫 | 獨立運作的專家代理 |
| **觸發方式** | 手動輸入 `/指令` | Claude 自動調用 | Claude 自動載入 | 自動委派或手動指定 |
| **檔案結構** | 單一 `.md` | 外部伺服器 + 設定檔 | 資料夾（`SKILL.md` + 腳本） | 單一 `.md`（含 YAML frontmatter） |
| **存放位置** | `.claude/commands/` | 外部服務 | `.claude/skills/` | `.claude/agents/` |
| **外部整合** | 否 | 是 | 否 | 否 |
| **獨立情境窗** | 否 | N/A | 否 | 是 |
| **需要寫程式** | 否 | 有時需要 | 否 | 否 |

---

## 各功能詳解

### Slash Command — 快捷鍵

最簡單的擴充方式，把常用提示詞存成檔案，隨時以 `/指令名` 呼叫。

```markdown
# .claude/commands/commit-msg.md
根據目前的 git diff 產生 commit message
```

使用：`/commit-msg "修正登入功能"`

---

### MCP Server — 外部系統橋接器

讓 Claude 能存取外部服務（GitHub、資料庫、Slack 等），Claude 會在需要時自動調用。

```bash
# 連接 GitHub
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 連接 PostgreSQL
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://user:pass@localhost:5432/mydb"
```

你說「PR #123 有什麼問題？」→ Claude 自動透過 MCP 查詢 GitHub。

---

### Skill — 知識庫模組

一組檔案（說明 + 腳本 + 參考文件），Claude 根據對話情境自動載入使用。

```
.claude/skills/pdf-processing/
├── SKILL.md          # 主要說明
├── FORMS.md          # 表單處理指南
└── scripts/
    └── fill_form.py
```

你說「幫我從這個 PDF 提取文字」→ Claude 自動發現並使用該 Skill。

---

### Subagent — 專家代理

擁有**獨立情境視窗**的專業 AI 助手，不會污染主對話。可限制工具權限，透過 Git 與團隊共享。

```markdown
---
name: code-reviewer
description: 程式碼審查專家，檢查品質、安全性和可維護性
tools: Read, Grep, Glob, Bash
model: inherit
---

你是資深程式碼審查員。
當被呼叫時，執行 git diff 查看最近變更，依照以下清單審查：
- 程式碼清晰度與命名
- 重複程式碼
- 錯誤處理
- 密鑰洩漏風險
```

內建 Subagent：**general-purpose**（通用）、**plan**（規劃）、**explore**（快速探索）。

使用 `/agents` 可互動式建立、編輯、刪除 subagent。

---

## Skill vs Subagent 怎麼選？

這兩者最容易混淆：

- **Skill** = 給 Claude 一本參考書 → 知識載入主對話，由主 Claude 決策執行
- **Subagent** = 請另一位專家幫忙 → 獨立情境，自己決策和執行

---

## 使用時機

| 情境 | 選擇 |
|------|------|
| 經常重複相同提示詞 | Slash Command |
| 需要連接外部系統（GitHub、DB、Slack） | MCP Server |
| 需要自動載入的標準流程/知識庫 | Skill |
| 需要獨立情境的專業任務處理 | Subagent |

---

## 協同運作範例

以「安全性程式碼審查」為例，四者可串聯使用：

1. `/security-review`（Slash Command）→ 啟動流程
2. `code-security` Skill → 載入安全審查標準
3. `security-auditor` Subagent → 獨立情境中深度分析
4. GitHub MCP Server → 抓取 PR 程式碼與變更歷程

---

## 入門建議

**學習順序**：Slash Command → Subagent → Skill → MCP Server（由簡到繁）

**團隊規模建議**：
- 個人開發者：Slash Command + Subagent
- 小型團隊：加上 Skill
- 中大型團隊：全部整合
