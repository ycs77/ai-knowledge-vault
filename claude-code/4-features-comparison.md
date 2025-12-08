# Claude Code 功能比較指南

Slash Command / MCP Server / Skill / Subagent 完整比較

---

## 🎯 四大功能總覽

### 1️⃣ **Slash Command（斜線指令）**
**比喻**：像是「快捷鍵」或「巨集指令」

- **觸發方式**：您手動輸入 `/指令名稱`
- **檔案結構**：單一 `.md` 檔案
- **存放位置**：`.claude/commands/`
- **複雜度**：★☆☆☆☆（最簡單）

**範例**：
```markdown
# .claude/commands/commit-msg.md
根據目前的 git diff 產生 commit message
```
使用：`/commit-msg "修正登入功能"`

---

### 2️⃣ **MCP Server（模型情境協定伺服器）**
**比喻**：像是「外掛程式」或「API 橋接器」

- **觸發方式**：Claude 自動使用（透過工具調用）
- **檔案結構**：外部伺服器 + 設定檔
- **存放位置**：外部服務（本機或遠端）
- **複雜度**：★★★☆☆

**功能**：連接外部系統，例如：
- 📊 資料庫查詢（PostgreSQL、MongoDB）
- 🐛 錯誤監控（Sentry）
- 💬 GitHub、Slack 整合
- 📁 檔案儲存系統

**設定範例**：
```bash
# 連接 GitHub
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 連接 PostgreSQL 資料庫
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://user:pass@localhost:5432/mydb"
```

**使用情境**：
```
您：「GitHub PR #123 有什麼問題？」
→ Claude 自動透過 MCP 伺服器查詢 GitHub
```

---

### 3️⃣ **Skill（技能）**
**比喻**：像是「專業助手包」或「知識庫模組」

- **觸發方式**：Claude 自動判斷使用
- **檔案結構**：資料夾 + 多個檔案（`SKILL.md` + 腳本 + 文件）
- **存放位置**：`.claude/skills/`
- **複雜度**：★★★☆☆

**結構範例**：
```
.claude/skills/pdf-processing/
├── SKILL.md          (主要說明)
├── FORMS.md          (表單處理指南)
├── REFERENCE.md      (API 參考文件)
└── scripts/
    ├── fill_form.py
    └── validate.py
```

**使用情境**：
```
您：「幫我從這個 PDF 提取文字」
→ Claude 自動發現並使用 pdf-processing Skill
```

---

### 4️⃣ **Subagent（子代理）**
**比喻**：像是「專業領域助手」或「委派專家」

- **觸發方式**：Claude 自動委派或您明確指定
- **檔案結構**：`.md` 檔案（包含 YAML frontmatter + 提示詞）
- **存放位置**：`.claude/agents/`
- **複雜度**：★★★☆☆

**關鍵特性**：
- ✅ **獨立情境視窗**：不會污染主對話
- ✅ **專業化指令**：針對特定領域設計
- ✅ **工具權限控制**：可限制使用的工具
- ✅ **團隊共享**：透過 Git 共享

**檔案範例**：
```markdown
---
name: code-reviewer
description: 程式碼審查專家，主動檢查程式碼品質、安全性和可維護性
tools: Read, Grep, Glob, Bash
model: inherit
permissionMode: default
---

你是資深程式碼審查員，確保高標準的程式碼品質和安全性。

當被呼叫時：
1. 執行 git diff 查看最近的變更
2. 專注於修改過的檔案
3. 立即開始審查

審查清單：
- 程式碼是否清晰易讀
- 函數和變數命名是否恰當
- 沒有重複的程式碼
- 適當的錯誤處理
- 沒有暴露的密鑰或 API keys
```

**使用 `/agents` 指令管理**：
```bash
/agents
```
這個指令可以讓您：
- 📋 查看所有可用的 subagent
- ➕ 互動式建立新的 subagent
- ✏️ 編輯現有的 subagent
- 🗑️ 刪除自訂 subagent

**使用情境**：
```
您：「請 code-reviewer subagent 檢查我的最新變更」
→ Claude 啟動獨立的程式碼審查專家來分析

或者：
您：「這段程式碼有錯誤」
→ Claude 自動委派給 debugger subagent 處理
```

**內建的 Subagent**：
- 🤖 **general-purpose**：通用型，使用 Sonnet，所有工具
- 📋 **plan**：規劃模式，優化研究程式碼庫
- 🔍 **explore**：探索型，使用 Haiku（快速），唯讀模式

---

## 📊 完整功能對照表

| 項目 | Slash Command | MCP Server | Skill | Subagent |
|------|---------------|------------|-------|----------|
| **觸發方式** | 手動 `/指令` | 自動（工具） | 自動（情境） | 自動委派/手動 |
| **複雜度** | ★☆☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ |
| **建置時間** | 幾分鐘 | 幾小時 | 15-30 分鐘 | 10-20 分鐘 |
| **檔案結構** | 單一 .md | 外部伺服器 | 資料夾 | 單一 .md（含 YAML） |
| **外部整合** | ❌ 受限 | ✅ 強大 | ❌ 無 | ❌ 無 |
| **獨立情境** | ❌ 否 | N/A | ❌ 否 | ✅ 是 |
| **自動委派** | ❌ 否 | ✅ 是 | ✅ 是 | ✅ 是 |
| **需要寫程式** | ❌ 否 | 🟡 有時 | ❌ 否 | ❌ 否 |
| **團隊共享** | Git | 設定檔 | Git | Git |
| **工具權限** | `allowed-tools` | 精確匹配 | `allowed-tools` | `tools` 欄位 |

---

## 💡 白話比喻

想像您在經營一間咖啡店：

1. **Slash Command** = **菜單上的快捷按鈕**
   - 按「美式咖啡」按鈕 → 立即製作美式咖啡
   - 簡單、快速、手動操作

2. **MCP Server** = **供應商配送系統**
   - 自動連接牛奶供應商、咖啡豆供應商
   - 即時查詢庫存、訂購原料
   - 整合外部系統

3. **Skill** = **員工訓練手冊**
   - 店員看到客人說「我要拿鐵」會自動知道怎麼做
   - 包含完整流程、注意事項、品質檢查清單
   - 自動應用專業知識

4. **Subagent** = **特定領域的專業師傅**
   - 咖啡師傅（咖啡專家）、甜點師傅（甜點專家）
   - 各自有獨立的工作區域和專業知識
   - 店長可以委派特定任務給特定師傅
   - 不會干擾到主要營運流程

---

## 🎯 使用時機決策指南

### 選擇 **Slash Command** 當：
✅ 經常重複使用相同提示詞
✅ 想要完全控制執行時機
✅ 任務簡單，單一檔案就夠
✅ 個人生產力工具

**例子**：`/review`、`/explain`、`/optimize`

---

### 選擇 **MCP Server** 當：
✅ 需要連接外部系統（GitHub、資料庫、Slack）
✅ 需要即時資料
✅ 團隊共用基礎設施整合
✅ 跨多個外部工具協作

**例子**：
- 查詢生產環境資料庫
- 監控 Sentry 錯誤
- 自動分析 GitHub PR

---

### 選擇 **Skill** 當：
✅ 希望 Claude 自動判斷使用時機
✅ 需要多個檔案和腳本
✅ 建立團隊標準化流程
✅ 複雜工作流程需要驗證步驟
✅ 不需要外部整合

**例子**：
- PDF 處理專家
- 程式碼審查標準
- 文件撰寫範本

---

### 選擇 **Subagent** 當：
✅ 需要專業化的 AI 助手處理特定任務
✅ 想保持主對話情境清晰（獨立情境視窗）
✅ 需要限制特定任務的工具權限
✅ 建立可重複使用的專家角色
✅ 團隊需要共享專業化助手

**例子**：
- code-reviewer（程式碼審查專家）
- debugger（除錯專家）
- data-scientist（資料科學家）
- security-auditor（安全稽核員）

---

## 🔄 協同運作範例

這四個功能可以一起使用！

### 情境：安全性程式碼審查工作流程

1. **您輸入**：`/security-review`（Slash Command）
   - 啟動安全審查流程

2. **Skill 載入**：`code-security` 技能
   - 自動載入安全審查標準、檢查清單

3. **Subagent 委派**：`security-auditor` 子代理
   - 在獨立情境中進行深度安全分析
   - 不會污染主對話的情境

4. **MCP Server 查詢**：GitHub MCP
   - 自動抓取 PR 程式碼、變更歷程

---

### 情境：資料分析任務

1. **您輸入**：`/analyze-sales`（Slash Command）
2. **Subagent 委派**：`data-scientist` 子代理處理分析
3. **Skill**：提供分析範本和最佳實踐
4. **MCP Server**：連接 PostgreSQL 資料庫取得資料

---

## 🆚 Subagent vs Skill 的關鍵差異

這兩個最容易混淆，讓我特別說明：

| 特性 | Skill | Subagent |
|------|-------|----------|
| **性質** | 知識庫/能力包 | 獨立 AI 代理 |
| **情境** | 載入到主對話 | 獨立情境視窗 |
| **執行者** | 主 Claude | 獨立的 Claude |
| **決策能力** | 提供知識，主 Claude 決策 | 自己決策和執行 |
| **適合** | 提供參考資料、範本 | 執行完整任務 |

**白話說明**：
- **Skill** 像是「給 Claude 一本參考書」
- **Subagent** 像是「請另一位專家來幫忙」

---

## 🚀 實際建議

**新手入門順序**：
1. 先學 **Slash Command**（最簡單，立即上手）
2. 再試 **Subagent**（建立專業助手）
3. 探索 **Skill**（加強知識庫）
4. 最後 **MCP Server**（連接外部系統）

**日常使用組合**：
- **個人開發者**：Slash Command + Subagent
- **小型團隊**：Subagent + Skill
- **中大型團隊**：全部整合（Slash + MCP + Skill + Subagent）

**建立 Subagent 小技巧**：
```bash
# 使用互動式介面建立
/agents

# 讓 Claude 幫你生成，然後用 /agents 微調
您：「幫我建立一個專門審查 React 元件的 subagent」
```

---

## 📚 相關文件連結

- Slash Commands: `code.claude.com/docs/en/slash-commands.md`
- MCP Servers: `code.claude.com/docs/en/mcp.md`
- Skills: `code.claude.com/docs/en/skills.md`
- Subagents: `code.claude.com/docs/en/sub-agents.md`

---

## 📝 總結

這四個功能各有特色，選擇合適的工具能大幅提升開發效率：

- **Slash Command**：快速重複的簡單任務
- **MCP Server**：外部系統整合
- **Skill**：團隊知識庫與標準流程
- **Subagent**：專業化的獨立 AI 助手

最佳實踐是組合使用這些功能，建立符合您團隊需求的工作流程！
