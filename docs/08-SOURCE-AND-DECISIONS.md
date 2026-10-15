# 来源、现状与决策记录

## 2026-10-15 本轮 M0 审查与决策

- 整理前的工作区与参考包均不是 Git 仓库；参考包有单页 HTML 原型和 v0.1 文档，没有 React/FastAPI 源码、锁文件或自动测试。本轮建立的是该包的工作副本，并保留原型原文件作为交互参考。
- 用户明确指定第一阶段优先 React + TypeScript、FastAPI、SVG，因此把旧文档的“前端本地存储、后端以后再定”改为本机 FastAPI + SQLite；仍不做外部部署。API 数值是保存后权威，前端 core 用于即时预览和 SVG 视图模型，跨语言 fixture 防止规则分叉。
- 旧原型的六个预设与三色布局可作界面/测试素材；其营养值没有逐项可核来源，只可标“演示数据”。原型跨日会清空记录，不作为新存储规则。
- 尚未用真实用户验证拍照意愿、拼图理解、推荐解释或长期留存。未进行浏览器视觉 QA、代码构建与营养准确性测试。

## 已核实（2026-10-15）

- 原对话中可读到用户提出“面向大众/薄肌”、拍照记录、营养拼图、食物推荐，并要继续在 Codex 建项目。旧助手回答在接口中只有内容引用标记，未取得正文，不能当作功能事实。
- 本机下载目录中的 `macropuzzle_mvp.html` 已复制到 `prototype/`。它是 18,028 字节的单文件 HTML 原型；本项目选用无编号文件。
- 原型可读代码中：六个食物预设、P/C/F 三色区域、用户可改目标、`localStorage` 存当天记录、照片只用 `FileReader` 预览、按缺口减超额的启发式推荐。没有照片识别、后端或账户。跨日时把旧食物清空。

## 当前建议与原因

| 决策 | 状态 | 原因 / 后续验证 |
| --- | --- | --- |
| 第一轮本地单机，先修记录闭环 | 建议 | 先验证交互价值；观察是否需要云同步 |
| 三色固定区域并单列能量 | 暂定 | 沿用原型易比较；测试用户是否误以为健康评分 |
| 推荐只用演示候选、展示原因 | 暂定 | 数据源未审；先验证解释能否被理解 |
| 不从照片直接估营养 | 边界 | 单图份量与做法不可靠，须用户确认 |
| 成人中文用户先试用 | 假设 | 需要确认地域、食物库、目标人群 |

## 外部依据

- [OpenAI：AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
- [OpenAI：用计划推进长任务（已归档示例）](https://developers.openai.com/cookbook/articles/codex_exec_plans)
- [GitHub：spec-driven/Spec Kit](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
- [GitHub：复核 AI 生成代码](https://docs.github.com/en/copilot/tutorials/review-ai-generated-code)
- [USDA FoodData Central FAQ/API](https://fdc.nal.usda.gov/faq/)、[API 指南](https://fdc.nal.usda.gov/api-guide/)
- [FDA：营养标签热量换算](https://www.fda.gov/media/81606/download)
- [WHO：健康饮食](https://www.who.int/health-topics/healthy-diet)
- [食物图片份量估计研究综述](https://pmc.ncbi.nlm.nih.gov/articles/PMC11607557/)

## 决策变更模板

`日期 / 决策 / 用户证据或技术证据 / 替代方案 / 影响文件 / 下一次验证`。没有证据时写“假设”，不要补写成已验证。
