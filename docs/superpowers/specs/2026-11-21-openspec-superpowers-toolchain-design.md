# OpenSpec + Superpowers 工具链设计（Team31）

日期：2026-11-21
状态：已获批（本轮仅搭工具链，不含赛题功能）

## 意图与原因

这个仓库要成为 AI-Challenge-Team31 的正式开发仓库，目前是零提交的空仓库。团队希望用
OpenSpec 管理「要做什么」、用 Superpowers 管理「怎么可靠地做出来」，两条流水线分工而
不互相打架。本轮只搭工具链，第一个功能留到下一轮再 spec。

## 目标

1. OpenSpec 装入本项目（项目级，而非全局），生成本仓库可提交的 `openspec/` 与
   `.dsh/skills/openspec-*`。
2. `AGENTS.md` 写明两条流水线的分工契约，任何 agent 进仓都能读到。
3. 工具链可验证：CLI 健康检查通过，DSH 能在本项目发现 `openspec-*` 技能。
4. 干净的首次提交基线（`.gitignore` 挡住 IDE 与依赖噪音）。

## 非目标

- 不写赛题功能的 spec / change；不建 `openspec/changes/<name>/` 业务条目。
- 不引入构建系统、语言选型、CI、测试框架（没有代码可测）。
- 不改全局 OpenSpec 配置（`--scope project`）。

## 反目标

- 若最终出现「同一份需求写在两个地方」（如同时存在于 `openspec/changes/*/design.md`
  与 `docs/superpowers/specs/`），本轮设计即失败。
- 若两套技能给出互相矛盾的流程指令，且仓内没有裁判文本，即失败。

## 已决定的分工契约

- **需求真相源 = OpenSpec**：`openspec/specs/` 是系统现状，`openspec/changes/<name>/`
  是一次改动（proposal → design → tasks），完成后 `archive` 回写 specs。
- **执行纪律 = Superpowers**：brainstorm 澄清 → 设计写进该 change 的 `design.md`
  （不再另建 `docs/superpowers/specs/` 副本）→ `writing-plans` 出计划 → TDD + 子代理
  执行 → `verification-before-completion` 通过后才 `archive`。
- **例外**：工具链/基础设施本身的改动（如本轮）不属于产品需求，短设计放
  `docs/superpowers/specs/`，不占用 change 目录。
- Artifacts 语言：中文（`openspec init --language`）。
- 技能作用域：项目级（`.dsh/skills/`），随仓库提交，队友与 CI 一致。

## 考虑过的其他方案

| 方案 | 取舍 | 结论 |
| --- | --- | --- |
| 只用 OpenSpec | 需求可追溯，但执行侧无 TDD/评审/验证纪律 | 否 |
| 只用 Superpowers | 执行纪律强，但变更历史不可查、规格不沉淀 | 否 |
| 两者并行但各自完整流程 | 简单，但双真相源，必然漂移 | 否 |
| **OpenSpec 管需求 + Superpowers 管执行（交界 = design.md 与 archive 前验证）** | 多一层文档纪律，小改动也要走 change 目录；换来单一真相源 | **采用** |

## 交给实施者的自由度

- `AGENTS.md` 的具体措辞与分节。
- `.gitignore` 条目（至少覆盖 `.idea/`、`node_modules/`）。
- 冒烟验证的具体命令组合，只要覆盖：CLI 健康、技能可发现。
