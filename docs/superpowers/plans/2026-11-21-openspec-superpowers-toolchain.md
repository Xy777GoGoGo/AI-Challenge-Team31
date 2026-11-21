# OpenSpec + Superpowers 工具链搭建 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use subagent-driven-development (recommended) or executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在空仓库 `AI-Challenge-Team31` 中装好 OpenSpec（项目级、DSH 目标、中文 artifacts），并用 `AGENTS.md` 固化 OpenSpec（需求真相源）与 Superpowers（执行纪律）的分工，产出干净的首次提交。

**Architecture:** 用 `openspec init --tools dsh --scope project` 生成 `openspec/` 与 `.dsh/skills/openspec-*`（DSH 自动从 `<project>/.dsh/skills` 发现技能，无需胶水）。仓根 `AGENTS.md` 只写分工契约与命令，不复制 OpenSpec 技能内部指令。最后以 CLI 健康检查与「DSH 能加载 `openspec-propose`」两项实测收尾。

**Tech Stack:** OpenSpec CLI 2.1.0（`@fission-ai/openspec`，全局安装于 `D:\nodeStore\v26.13.1`）、DeepSeek Harness skill 发现机制、git。

**Spec:** `docs/superpowers/specs/2026-11-21-openspec-superpowers-toolchain-design.md`

## Global Constraints

- OpenSpec CLI 版本已装：`2.1.0`，不得升级或重装。
- 技能作用域：项目级（不得使用 `--scope global`）；不得修改 `~/.dsh` 或其他仓库。
- artifacts 语言：中文（`--language` 取值为中文名，写入 `openspec/config.yaml` 的 context）。
- 不创建任何 `openspec/changes/<name>/` 业务条目；本轮只允许工具链产物。
- 不修改 `.idea/` 内任何文件；不执行 `git push`。
- 提交信息用中文祈使句，形如 `chore: 初始化 OpenSpec 与 Superpowers 工具链`。

## Review Focus

- **重复真相源**：`AGENTS.md` 里是否又抄了一份 OpenSpec 流程细则，导致它与 `.dsh/skills/openspec-*/SKILL.md` 日后不一致 —— 期望 `AGENTS.md` 只给分工与命令，细则指向技能。
- **技能作用域错位**：产物若落在家目录而非仓库内，队友拉取后没有技能 —— 期望 `git status` 中出现 `.dsh/skills/openspec-*`。
- **中文语言未生效**：`openspec/config.yaml` 的 `context` 缺少语言声明，后续 artifact 全英文 —— 期望 context 内有 `All artifacts must be written in ...`。
- **IDE 噪音入库**：`.idea/workspace.xml` 等被提交 —— 期望 `.gitignore` 生效且 `git status` 干净。
- **技能未被 DSH 发现**：`openspec-propose` 加载失败 —— 期望 `skill` 工具能返回其内容。

---

### Task 1: 初始化 OpenSpec（项目级、DSH、中文）

**Files:**
- Create: `openspec/`（由 CLI 生成：`config.yaml`、`specs/`、`changes/` 等）
- Create: `.dsh/skills/openspec-*/SKILL.md`（由 CLI 生成）

**Interfaces:**
- Consumes: 无
- Produces: `openspec/config.yaml`（含中文 context）；`.dsh/skills/openspec-propose/SKILL.md` 等技能目录，供 Task 4 验证、供后续所有 change 使用。

- [ ] **Step 1: 确认仓库为空且 CLI 可用**

Run: `openspec --version` 与 `git status --short`
Expected: 版本输出 `2.1.0`；`git status --short` 仅显示 `?? .idea/`（说明无既有 openspec 产物，可安全初始化）。

- [ ] **Step 2: 执行初始化（非交互）**

Run:
```powershell
openspec init . --tools dsh --scope project --language "Chinese (Simplified)" --no-animation --no-copilot-cloud
```
Expected: 输出成功摘要，提到 `.dsh/skills` 与 `openspec/`；无交互提问停留。

- [ ] **Step 3: 验证产物存在且为中文配置**

Run: `Get-ChildItem .dsh\skills | Select-Object Name` 与 `Get-Content openspec\config.yaml`
Expected: 至少列出 `openspec-propose`、`openspec-apply-change`、`openspec-archive-change`；`config.yaml` 的 `context` 含 `All artifacts must be written in Chinese (Simplified)`。

- [ ] **Step 4: 验证 OpenSpec 自身健康**

Run: `openspec doctor`
Expected: 无 error 级别问题（warning 可接受，需在对话中说明）。

- [ ] **Step 5: 提交**

```bash
git add openspec .dsh
git commit -m "chore: 初始化 OpenSpec（项目级 dsh 技能、中文 artifacts）"
```
Expected: 提交成功；`git log --oneline -1` 显示该提交。

---

### Task 2: 建立首次提交基线与忽略规则

**Files:**
- Create: `.gitignore`
- Modify: 无

**Interfaces:**
- Consumes: Task 1 已提交的 `openspec/`、`.dsh/skills/`
- Produces: 干净的 `git status` 基线，Task 3 的 `AGENTS.md` 得以单独成提交。

- [ ] **Step 1: 写 `.gitignore`**

内容至少包含：
```gitignore
# IDE
.idea/
*.iml
# 依赖
node_modules/
# 系统噪音
.DS_Store
Thumbs.db
```

- [ ] **Step 2: 验证忽略生效**

Run: `git status --short`
Expected: 不再出现 `.idea/`；仅显示 `?? .gitignore`（若 Task 1 已提交）。

- [ ] **Step 3: 提交**

```bash
git add .gitignore
git commit -m "chore: 添加 .gitignore（IDE 与依赖噪音）"
```
Expected: 提交成功，`git status --short` 输出为空。

---

### Task 3: 写 AGENTS.md 分工契约

**Files:**
- Create: `AGENTS.md`
- Test: 人工核对（无测试框架；见 Step 3 的自检命令）

**Interfaces:**
- Consumes: Task 1 的技能名（`openspec-propose` / `openspec-apply-change` / `openspec-archive-change`）与 Task 1 的目录布局
- Produces: 仓根 `AGENTS.md`，后续每个会话由 agent 自动读取的分工契约。

- [ ] **Step 1: 起草 `AGENTS.md`**

必须包含且仅包含这五部分：
1. 一句话分工：OpenSpec = 需求真相源（`openspec/specs/` 现状，`openspec/changes/<name>/` 一次改动）；Superpowers = 执行纪律（brainstorm → writing-plans → TDD → verification）。
2. 一次改动的标准路径，逐步给出命令：
   `openspec new change <name>` → 写 `proposal.md`/`design.md`/`tasks.md` → 实现（TDD）→ `openspec validate <name>` → `openspec archive <name>`。
3. 禁止重复真相源：产品级设计只写 `openspec/changes/<name>/design.md`；`docs/superpowers/specs/` 仅存工具链/基础设施等非产品设计。
4. 完成定义：`verification-before-completion` 通过（有证据的命令输出）才允许 `openspec archive`。
5. 常用命令速查（`openspec list`、`openspec show <name>`、`openspec view`、`openspec validate --all`）。

- [ ] **Step 2: 自检无细则抄写**

Run: `Select-String -Path AGENTS.md -Pattern "SKILL|skill" `
Expected: 只出现指向技能名的引用（如 `openspec-propose`），不出现流程细则段落；`AGENTS.md` 行数 < 80。

- [ ] **Step 3: 提交**

```bash
git add AGENTS.md
git commit -m "docs: 固化 OpenSpec 与 Superpowers 分工契约"
```
Expected: 提交成功。

---

### Task 4: 端到端冒烟验证

**Files:**
- Modify: `AGENTS.md`（仅在验证发现契约与真实 CLI 行为不符时修正）
- Test: 本节命令即验收

**Interfaces:**
- Consumes: Task 1–3 的全部产物
- Produces: 一份可复述的验收结论（命令 + 输出），满足完成定义。

- [ ] **Step 1: 验证清单自动检查**

Run:
```powershell
openspec doctor
openspec list
openspec validate --all
openspec show --help | Select-Object -First 3
```
Expected: `doctor` 无 error；`list` 与 `validate --all` 正常返回（无 change 时应为空或明确「无变更」而非报错）。

- [ ] **Step 2: 验证 DSH 能在本项目发现 OpenSpec 技能**

Run: 调用 `skill` 工具，`name` 传 `openspec-propose`
Expected: 返回该技能的完整指令内容（而非「未知技能」）。此步证明 `.dsh/skills` 已被 DSH 发现。

- [ ] **Step 3: 确认工作区干净**

Run: `git status --short` 与 `git log --oneline`
Expected: `git status --short` 为空；`git log --oneline` 至少有 3 条本轮提交。
