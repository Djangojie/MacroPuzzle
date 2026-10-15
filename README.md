# MacroPuzzle · 营养拼图

这是 **M0 项目初始化与开发规格**，不是已上线产品，也尚无 React/FastAPI 可运行应用。原始单页原型见 [`prototype/macropuzzle_mvp.html`](prototype/macropuzzle_mvp.html)：可直接用浏览器打开，支持照片本地预览、手动记录、营养拼图和演示食物推荐。它没有照片识别、账户、服务端或经过验证的营养数据库。

## 先看什么

1. [`docs/01-PRD.md`](docs/01-PRD.md)：用户、范围、成功指标、用户故事。
2. [`docs/02-INTERACTION.md`](docs/02-INTERACTION.md)：页面流程和拼图规则。
3. [`docs/03-NUTRITION-SAFETY.md`](docs/03-NUTRITION-SAFETY.md)：数值、数据来源与安全边界。
4. [`docs/04-ARCHITECTURE.md`](docs/04-ARCHITECTURE.md)、[`docs/05-API-DATA.md`](docs/05-API-DATA.md)：架构、数据与 API 合约。
5. [`docs/06-ROADMAP-TEST.md`](docs/06-ROADMAP-TEST.md)：分期、测试和验收。
6. [`docs/09-PUZZLE-ENGINE.md`](docs/09-PUZZLE-ENGINE.md)：纯数据拼图引擎与 SVG 映射。
7. [`docs/10-PHASE1-TASKS.md`](docs/10-PHASE1-TASKS.md)：第一阶段可独立验收的任务卡。
8. [`docs/07-AI-WORKFLOW.md`](docs/07-AI-WORKFLOW.md)：AI 协作建议。
9. [`prompts/first-codex-task.md`](prompts/first-codex-task.md)：第一阶段第一张任务卡。

## 用法

在 Codex 中打开此目录，按 `docs/10-PHASE1-TASKS.md` 顺序完成小任务。第一阶段目标是 React + TypeScript 前端、FastAPI 本机 API、SVG 拼图与可测试的记录闭环。后端只使用本机 SQLite；不调用付费识别或营养服务。拍照仅在浏览器本地预览，手动确认的营养数据才进入 API。`AGENTS.md` 是项目约束；文档里标记 **待验证** 的产品和营养决策不得当成事实。

## 目录

```text
MacroPuzzle/
  AGENTS.md
  README.md
  docs/
    01-PRD.md
    02-INTERACTION.md
    03-NUTRITION-SAFETY.md
    04-ARCHITECTURE.md
    05-API-DATA.md
    06-ROADMAP-TEST.md
    07-AI-WORKFLOW.md
    08-SOURCE-AND-DECISIONS.md
    09-PUZZLE-ENGINE.md
    10-PHASE1-TASKS.md
  prompts/first-codex-task.md
  prototype/macropuzzle_mvp.html
  apps/web/README.md
  apps/api/README.md
  packages/core/README.md
  packages/contracts/README.md
```

## 来源状态

参考包原有来源说明见 [`docs/08-SOURCE-AND-DECISIONS.md`](docs/08-SOURCE-AND-DECISIONS.md)。本轮已检查该包及原型代码；当前交付目录是其工作副本，新增文档与架构修订以本轮用户要求为准。未验证原始对话全文。整理前的工作区及参考目录均非 Git 仓库；此公开快照尚无锁文件、自动测试或浏览器实测记录。
