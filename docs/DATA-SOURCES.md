# Data Sources · 数据源与适配器表

> 原则：WHOOP 只是适配器之一；小米健康 **不是** 主数据源。

## 1. Source Matrix

| Source ID | 类型 | 方向 | 主/辅 | 状态 | 说明 |
| --- | --- | --- | --- | --- | --- |
| `whoop` | 可穿戴 MCP | 只读 | **主** | 已落地 | `whoop-personal-vps` 六工具 |
| `notion-nutrition` | Notion DB | 读写 | 主（饮食正本） | 已落地 | 饮食记录库 |
| `notion-training` | Notion DB | 读写 | 主（训练正本） | 已落地 | 训练记录库 |
| `notion-body` | Notion DB | 读写 | 主（身体正本） | 已落地 | 身体数据库 |
| `notion-daily-meals` | Notion Page | 读写 | 派生（打卡视图） | 已落地 | 「每天吃的饭」 |
| `notion-fitness-memory` | Notion Page | 读写 | 规则/偏好 | 已落地 | 不存度量时间序列 |
| `manual-voice` | 用户口述 | 写入 | 辅→正本 | 已落地 | 经确认后入台账 |
| `manual-meal-photo` | 餐照 | 写入 | 辅→正本 | 已落地 | 白底 Ins 风处理后再入库 |
| `watch-oral-or-screenshot` | 手表口述/截图 | 辅交叉核对 | 辅 | 经验可用 | 睡眠有时交叉核对 WHOOP；**非 API 主源** |
| `xiaomi-health-api` | API | — | **不做主源** | 不采用 | 小米/IFTTT 仅叫醒相关 |
| `apple-health` | 导出/API | 只读 | 未来 | 规划 v2 | `AppleHealthAdapter` |
| `alert-ntfy` | 通知 | 写出 | 通道 | 已引用 | 正文禁数值 |

Notion 真实 database/page id 使用环境变量占位，**不写进 Public 文档字面量。**

## 2. Entity Schemas（最小）

### 2.1 MetricSample（可穿戴）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `metric` | enum | `recovery_score` / `hrv_ms` / `rhr_bpm` / `sleep_performance` / `sleep_duration_hrs` / `day_strain` / … |
| `value` | number \| null | null=缺失 |
| `unit` | string | 显式单位 |
| `as_of` | date/datetime | 指标所属日；睡眠用醒来日 |
| `source` | string | `whoop` |
| `fetched_at` | datetime | 拉取时间 |
| `band` | string \| null | 如 recovery 色带（描述性，非处方） |

### 2.2 NutritionEntry

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `date` | date | Asia/Shanghai |
| `meal` | string | 早/午/晚/加餐 |
| `items_text` | string | 描述 |
| `kcal` / macros | number \| null | 可空；`数据来源=估算|实测|标签` |
| `source` | `manual-voice` \| `manual-meal-photo` \| … |
| `source_id` | string | 幂等键组成部分 |
| `confidence` | number \| enum | 低/中/高 |
| `created_by` | `agent` \| `user` |

### 2.3 TrainingEntry

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `date` | date | |
| `sport` | string | |
| `duration_min` | number \| null | |
| `notes` | string | |
| `whoop_workout_ref` | string \| null | 可选对齐 |
| `source` / `source_id` / `created_by` | | 同饮食 |

### 2.4 BodyEntry

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `date` | date | |
| `weight_kg` / `body_fat_pct` / `waist_cm` | number \| null | |
| `method` | `scale` \| `oral_estimate` \| … | 口述估计必须标注 |

## 3. Conflict Priority

主指标（恢复/HRV/睡眠时长等）：

1. `whoop`
2. 口述实测（用户明确读数）
3. 截图转录
4. 估算

饮食能量：标签/秤重 > 估算餐照 > 粗口述。  
**小米健康 API 不参与优先级。**

## 4. Freshness

- 日概览：优先 `as_of == today(Asia/Shanghai)`；否则展示日期并标注非今日。
- 趋势：遵守上游工具最小样本量；不足则 `null` + 说明。

## 5. Synthetic Examples

`examples/synthetic/` 仅放标注 `SYNTHETIC` 的假数据，供 dashboard/契约测试。禁止从生产 raw 拷贝进 Public 树。
