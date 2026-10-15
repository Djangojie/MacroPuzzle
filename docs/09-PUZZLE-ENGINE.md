# 拼图引擎规则 v0.1

本规则把“每日正方形”定义为**记录进度的视觉映射**，不表示健康程度或每日应吃到 100%。同样输入应产生同样的数值输出；SVG/颜色/动画不能改变计算。

## 输入与输出

输入：指定 `localDate` 的已确认记录、可空的 `DailyGoals`。先按 `docs/03-NUTRITION-SAFETY.md` 求当日原值合计，得到 P/C/F/能量。输出：每项 `{consumed, goal|null, fillRatio|null, overflow|null, state}`；`fillRatio` 为 0 到 1 的数，`overflow=max(consumed-goal,0)`；`state='unset'|'below'|'met'|'over'|'unknown'`。`unknown` 用于合计缺失（如能量未知），`unset` 用于无目标。空记录的已知 P/C/F 为 0。

对正目标 `g` 和已知摄入 `x`：`fillRatio=min(max(x/g,0),1)`。`x<g` 为 below，`x=g` 为 met，`x>g` 为 over。缺目标或 `g<=0` 不计算比率，显示“目标未设置/无效”。显示舍入后的相等不改变原值比较结果。

原型 30%/45%/25% 的 P/C/F 区域高度仅是视觉布局。综合填充进度用三项**目标均有效时**的比率平均 `round(100*(rP+rC+rF)/3)`；任何一项目标缺失时显示“进度待设置”，不从剩余目标推一个看似完整的百分比。能量单独展示，不进入综合进度。目标超额的原值与 `+X g` 文本必须同时保留。

## SVG 映射

- 固定 `viewBox="0 0 300 300"`；P 区 `y=0,h=90`，C 区 `y=90,h=135`，F 区 `y=225,h=75`。各区填充矩形从底部向上：`height=zoneHeight*fillRatio`、`y=zoneTop+zoneHeight-height`。
- 未设置/未知值用空白或斜线纹理，不用 0% 冒充。超额由区域外的文字徽标表达，不再延伸矩形。`clipPath` 限制图形在正方形内。
- SVG 有整体 `role="img"` 和简短标题；另提供 DOM 文本列表含已摄入、目标、超额与“估算值”标签。颜色不可作为唯一信息。减少动态效果适配 `prefers-reduced-motion`。
- 同一组数值在 320px、375px 与桌面宽度下不改变语义；图形允许缩放，文本不能因图形裁切而丢失。

## 确定性示例（仅测试）

目标 P=100g、C=200g、F=50g；摄入 P=50g、C=250g、F=0g：比率为 0.5、1、0；综合进度 50%；碳水超出 50g。SVG 高度分别 45、135、0。展示摄入碳水仍为 250g，不能写成 200g。

目标缺失、0、负数、非数、记录跨日、浮点边界、摄入超目标以及能量未知均必须纳入自动测试。具体测试分工见 `docs/10-PHASE1-TASKS.md`。
