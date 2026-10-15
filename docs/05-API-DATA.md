# 数据模型与 API 契约 v0.2 · 待实现

本页是 M1 的实现合约，当前没有可调用服务。正式 OpenAPI 文件在任务 P1-02 生成，并以契约测试保证本页与运行接口一致。所有数值是估计/来源记录，`null` 表示未知，`0` 才表示已知为零。

## 核心对象

```ts
type Nutrients = { proteinG: number; carbsG: number; fatG: number; energyKcal: number | null };
type Provenance = {
  kind: 'demo' | 'label' | 'database' | 'manual';
  sourceId: string | null; sourceUrl: string | null;
  sourceVersion: string | null; observedAt: string | null;
  userEdited: boolean;
};
type EntryInput = {
  localDate: string; name: string;
  quantity: number; unit: 'g' | 'serving';
  basisQuantity: number; basisUnit: 'g' | 'serving';
  nutrientsForBasis: Nutrients; provenance: Provenance;
  note: string | null; confirmed: true;
};
type FoodEntry = EntryInput & {
  id: string; createdAt: string; updatedAt: string; revision: number;
  nutrientsConsumed: Nutrients;
};
type DailyGoals = {
  proteinG: number | null; carbsG: number | null; fatG: number | null;
  energyKcal: number | null; origin: 'user' | 'demo'; updatedAt: string;
};
type Candidate = {
  id: string; name: string; basisQuantity: number; basisUnit: 'g' | 'serving';
  nutrientsForBasis: Nutrients; provenance: Provenance;
  allergenStatus: 'unknown' | 'declared'; allergens: string[];
};
type PuzzleMetric = {
  consumed: number | null; goal: number | null; fillRatio: number | null;
  overflow: number | null; state: 'unset' | 'below' | 'met' | 'over' | 'unknown';
};
type DayResponse = {
  date: string; entries: FoodEntry[]; totals: Nutrients; goals: DailyGoals | null;
  puzzle: { protein: PuzzleMetric; carbs: PuzzleMetric; fat: PuzzleMetric;
    energy: PuzzleMetric; completionPercent: number | null };
};
type RecommendationItem = {
  candidate: Candidate; score: number | null; reasons: string[];
  limitations: string[];
};
```

`localDate` 是用户选定的日历日期 `YYYY-MM-DD`，严格校验真实日期；不会随时区变更迁移。时间戳是 UTC ISO 8601。`id` 为 UUID。`quantity/basisQuantity` 只有单位相同才可相除；`serving` 必须显示基准描述，如“每 1 份”，不可假定克重。数值须有限、非负；份量与基准量必须大于 0。能量未知时不以 4/4/9 自动补值。

`confirmed: true` 表示用户在提交前看过并确认该条记录。后端拒绝缺少确认的写入；照片或未来 AI 候选绝不直接写入。客户端不能传 `nutrientsConsumed`、`id` 或时间戳作为新增记录的权威值。编辑任一营养字段或份量时保留原来源字段并设 `userEdited: true`，UI 显示“经用户修正”；不得继续显示为未经修正的包装/数据库值。

## M1 本机 API

统一 JSON；错误体 `{code, message, fieldErrors?: Record<string,string>}`。输入错误 422，日期格式错误 400，不存在 404，编辑版本冲突 409，持久化失败 500。后端异常不返回堆栈或数据库路径。

| 方法与路由 | 请求 | 成功响应 | 说明 |
| --- | --- | --- | --- |
| `GET /v1/health` | 无 | 200 `{status:'ok'}` | 本机开发探针 |
| `GET /v1/days/{date}` | 日期路径 | 200 `{date,entries,totals,goals,puzzle}` | `totals` 各项可为 null；无记录的已知合计为 0 |
| `POST /v1/entries` | `EntryInput`，可选 `Idempotency-Key` | 201 `FoodEntry` | 服务端重算摄入量；重复键返回原结果 |
| `PATCH /v1/entries/{id}` | 可编辑字段 + 必填 `revision` + `confirmed:true` | 200 `FoodEntry` | 原子更新并递增 revision |
| `DELETE /v1/entries/{id}` | 无 | 204 | UI 的短时撤销通过重新提交保留原数据快照，M1 不保证同 ID 恢复 |
| `GET /v1/goals` | 无 | 200 `DailyGoals \| null` | 不自动注入演示目标 |
| `PUT /v1/goals` | 数值与 `origin:'user'` | 200 `DailyGoals` | 至少一个目标；已设值须为正 |
| `GET /v1/recommendations?date=YYYY-MM-DD` | 日期 | 200 `{mode,items,limitations}` | `mode='demo-ranked' \| 'unranked' \| 'empty'` |
| `DELETE /v1/local-data` | `{confirmed:true}` | 204 | 清空本机记录/目标；UI 二次确认 |

M1 不提供照片上传、食物库搜索、账号或公网上传。演示候选作为版本化静态 fixture 随应用发布。若无目标、数据不完整或过敏/忌口无法核查，推荐接口返回 `unranked` 或 `empty`，不能用“适合你”。

`demo-ranked` 只表示按演示数据和用户自行设定目标排序，永远不称“个体营养建议”。评分采用 `02-INTERACTION.md` 的固定公式；得分相同按候选 ID 升序。`unranked` 的每项 `score:null`，且必须列出无法排序的原因。`items` 含候选、营养数值、可理解的理由和限制；空数组时提供可读说明。

## 响应例子

```json
{
  "id": "d298b245-5296-4077-8d4d-d75c78e1687b",
  "localDate": "2026-10-15",
  "createdAt": "2026-10-15T04:00:00Z",
  "updatedAt": "2026-10-15T04:00:00Z",
  "revision": 1,
  "name": "示例食物",
  "quantity": 150,
  "unit": "g",
  "basisQuantity": 100,
  "basisUnit": "g",
  "nutrientsForBasis": {"proteinG": 8, "carbsG": 20, "fatG": 3, "energyKcal": 140},
  "nutrientsConsumed": {"proteinG": 12, "carbsG": 30, "fatG": 4.5, "energyKcal": 210},
  "provenance": {"kind": "demo", "sourceId": null, "sourceUrl": null, "sourceVersion": null, "observedAt": null, "userEdited": false},
  "note": null,
  "confirmed": true
}
```

例子只用于算术与接口测试，不是可采信的食物营养数据。

## SQLite 表（建议）

- `entries`: `id` 主键、`local_date` 索引、名称、份量/基准、各项每基准营养值、来源 JSON/分列、注释、创建/更新 UTC、`revision`。摄入值由服务端计算，可存快照用于审计，但每次写入重算。
- `goals`: M1 单行设置；数值、来源、更新时间。
- `idempotency_keys`: key 主键、请求摘要、响应 ID/状态；同 key 不同请求返回 409。
- `schema_migrations`: 版本号与执行时间。迁移失败回滚，不丢弃旧数据。

旧原型键 `macropuzzle-mvp-v1` 的数据缺少基准份与来源，且跨日会清空；M1 **不自动迁移**。首次运行显示“旧演示记录不会导入”；不删除旧键。导入方案需另设计人工确认。

## 第二阶段预留

云同步、真实营养库、图片候选识别若启动，需要认证、资源隔离、密钥代理、同意/删除期限、来源许可与新版本契约。不得把 M1 本机接口直接部署成生产 API。
