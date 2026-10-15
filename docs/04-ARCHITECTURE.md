# 技术架构 v0.2 · M1 实施方案

## 目标与边界

M1 在开发机运行 React + TypeScript + Vite 前端、FastAPI API 与 SQLite。没有账户和生产部署：API 只绑定 `127.0.0.1`，任何同机进程均可访问，不能把这套配置暴露到公网或局域网。数据只存本机。浏览器图片用 object URL 预览并及时释放；API 不接收图片。所有记录的营养、份量和来源由用户确认后发送。依赖版本在实现时核对官方资料并锁定。

```text
相机/相册 → 浏览器本地预览 ── 不进入 API/数据库
用户输入/确认 → React 表单 → HTTP /v1 → FastAPI/Pydantic → SQLite
                            ↘ types + calculation contract ↗
GET /v1/days/{date} → 纯数值结果 → SVG 视图模型 → <svg> + 文本列表
GET /v1/recommendations → 演示候选排序 + 解释 → 用户再确认记录
```

## 责任划分

| 模块 | 职责 | 禁止 |
| --- | --- | --- |
| `apps/web` | 交互、表单、图片预览、HTTP 适配、SVG 绘制、可访问文本 | 自己计算第二套营养合计或排序；上传照片 |
| `packages/core` | TypeScript 类型、纯函数：校验、缩放、合计、拼图视图模型、推荐排序 | React、DOM、存储、网络、时钟依赖 |
| `apps/api` | FastAPI、Pydantic 输入校验、SQLite 持久化、日期查询；与 core 相同规则的 Python 计算实现 | 从照片推断营养、默认外联、硬编码个人目标 |
| `packages/contracts` | OpenAPI schema、JSON fixture 和跨语言契约测试 | 让两个实现静默分叉 |

M1 的**数值权威是后端**。前端 core 可用于即时表单预览和 SVG 视图模型，但保存后的总量与推荐以 API 响应为准。相同输入必须在 TypeScript 与 Python 的契约 fixture 中产生相同未舍入结果；若维护双实现的成本过高，后续可改为 API 单一计算源，先记录决策再改。SVG 仅消费 `PuzzleViewModel` 的 `fillRatio` 等数值，不能由像素反推克数。

## 建议目录

```text
apps/web/src/{pages,components,api,styles}/
apps/api/app/{main.py,models.py,schemas.py,repository.py,nutrition.py,recommendation.py}/
apps/api/tests/
packages/core/src/{types.ts,validation.ts,nutrition.ts,puzzle.ts,recommendation.ts}/
packages/core/src/*.test.ts
packages/contracts/{openapi.yaml,fixtures/}
prototype/macropuzzle_mvp.html
```

## 数据和运行路径

1. 用户确认记录前，UI 展示日期、份量基准、P/C/F、能量和来源；确认后才 `POST`。
2. FastAPI 对 `quantity`、同单位基准和每基准营养值校验，重新计算 `nutrientsConsumed`，不接受客户端自报合计作为权威。
3. SQLite 事务保存记录；同日读取按记录日期聚合，日期以客户端明确传入的 `YYYY-MM-DD` 为准，不用服务器 UTC 日期替代。
4. API 输出未经显示舍入的数值；前端统一展示格式。刷新、编辑、删除后重新取当天数据。
5. 推荐只读经确认记录与演示候选，包含来源、解释和限制；不作为健康或过敏安全结论。

## 故障与安全

- API 不可用：前端保留未提交表单，不显示“已保存”；给重试入口。
- SQLite 写失败：返回明确错误并回滚；数据库文件及其 WAL 不提交 Git。备份/导出能力列入后续任务。
- 浏览器刷新：已保存文字记录从 API 恢复；照片预览消失，界面说明该行为。
- 跨域：开发期仅允许明确的本机 Vite origin，正式环境重新设计 CORS、认证、HTTPS 和数据隔离。
- 本地演示数据与真实营养源分离；未核准许可证和来源前不接入外部食品库。

## 待验证技术假设

- 小规模单机演示时 SQLite 足够；并发与同步需求尚未验证。
- 后端计算与前端即时预览的跨语言一致性可由共享 fixture 管住；实际维护成本尚未验证。
- 浏览器相机输入、SVG 在目标手机浏览器上的兼容与无障碍表现尚未测试。
