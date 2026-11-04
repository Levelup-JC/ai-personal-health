# Architecture · 组件与接口

> 与 [`SOLUTION.md`](./SOLUTION.md) 配套。Public 仓使用逻辑名；运行时 URL/Token 仅来自环境变量或仓外 ops。

## 1. Component Map

| 组件 | 职责 | 运行位置（逻辑） |
| --- | --- | --- |
| Grok Bot + Agent「健身管理」 | 编排、对话、工具调用、写前确认 | Owner 的 Bot 工作区 |
| MCP `whoop-personal-vps` | WHOOP 六工具只读 wellness | 自建 `whoop-mcp` ← HTTP MCP 连接器 |
| Notion MCP / API | 台账查询与写入 | Notion SaaS |
| Manual ingest | 口述结构化、餐照预处理 | Bot 桌面/视觉能力 |
| Dashboard builder（可选） | `raw/*.json` → `data.json` → `index.html` | 本地/box 脚本 |
| Alert channel | 叫醒/报表通知（无数值正文） | 自建 ntfy |
| Ops binder | 隧道 `public_url`、Bearer、VPS | **仓外私册** |

## 2. WHOOP MCP Tool Surface

只读；返回中 `null` 表示不可用，不得改写为 0。

| Tool | 典型用途 | 默认窗口 |
| --- | --- | --- |
| `whoop_get_today_overview` | 日概览圆环与当日指标 | 当日 |
| `whoop_get_recovery_trend` | 恢复趋势 | days=30 |
| `whoop_get_hrv_trend` | HRV 趋势 / CV | days=30 |
| `whoop_get_sleep_trend` | 睡眠趋势（醒来日） | days=14 |
| `whoop_get_training_load` | Day Strain 7/28 描述比 | days=42 |
| `whoop_get_workouts` | 训练列表与区间 | days=14 |

接口语义以上游 MCP 描述为准：wellness only，非诊断、非运动清关。

## 3. Adapter Interface（逻辑）

```text
interface HealthAdapter {
  id: string                  // e.g. "whoop" | "notion-ledger" | "manual"
  capabilities: Cap[]         // read_metrics | write_ledger | ingest_media
  fetch(query): AdapterResult // 归一化；失败抛 AdapterError
  write?(draft): WriteResult  // 仅 ledger 类；须幂等
}

AdapterResult {
  source: string
  fetched_at: ISO-8601        // 建议带偏移或标注 Asia/Shanghai
  timezone: "Asia/Shanghai"
  entities: Entity[]          // 见 DATA-SOURCES
  missing: string[]           // 字段路径列表
  raw_ref?: string            // 可选本地 raw 文件名，不含密钥
}
```

## 4. Orchestration Flows

### 4.1 Daily Overview
1. 并行：WHOOP overview + 可选 trends；Notion 当日 nutrition/training/body。
2. 任一源失败 → 该块降级文案，其它块继续。
3. 组装对话报告；每值标注来源；附非医疗声明。
4. 可选：写 raw 并 build dashboard。
5. 可选：ntfy「报表已生成」（无数值）。

### 4.2 Ledger Ingest（口述 / 餐照）
1. 解析为草稿（餐照：风格化仅用于展示；营养估算基于约定流程）。
2. 展示草稿 + confidence；Owner 确认。
3. 按幂等键写入对应 Notion 库。
4. 更新打卡页状态（派生），不把度量写入「记忆」页。

### 4.3 Red Flag
1. 命中红旗关键词/意图 → 固定就医文案。
2. 跳过建议与训练处方式语言。
3. 不写入「诊断」类字段。

## 5. Failure Modes

| 失败 | 用户可见 | 禁止 |
| --- | --- | --- |
| MCP 超时 / 隧道失效 | 「可穿戴数据暂时不可用，请稍后再试或检查连接器。」 | 编造 Recovery/HRV |
| Notion 失败 | 「台账暂时不可用；口头要点仍可先记在对话，稍后再入库。」 | 静默丢写入 |
| 部分字段 null | 显示「数据缺失」并最多追问 1 次 | 用 0 或昨日值冒充 |
| 餐照模型失败 | 回退口述录入 | 假装已估算营养 |

## 6. Security Boundaries

- Bot 聊天中禁止粘贴 Bearer、`.env`、隧道完整 URL。
- 健身 Agent 的 Notion token **不得** 授权家庭库。
- Dashboard 开源树只含合成 `examples/synthetic/`。
- 连接器凭证轮换流程写在仓外 ops，不在本文件展开。

## 7. Relation to Grok Bot

本仓是 **方案与约定 + 可选脚本骨架**，不是替代 Grok Bot。运行时编排默认发生在 Owner 的 Grok Bot / Agent 内，通过已安装 MCP 连接器访问 WHOOP 与 Notion。
