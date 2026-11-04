# Adapters · 适配器约定

## 目标

把「可穿戴 / 台账 / 手动录入 / 未来系统」统一成可测试的边界，使 **AI 编排** 不绑定某一厂商。

## 目录建议（实现阶段）

```text
adapters/
  README.md          ← 本文件
  contract.md        ← 可选：JSON Schema 外置
  whoop/             ← 调用 MCP 工具的薄封装（无密钥）
  notion_ledger/     ← 饮食/训练/身体读写
  manual_ingest/     ← 口述与餐照草稿
  apple_health/      ← v2 占位
```

MVP 可先只有约定与伪代码；运行时继续由 Grok Bot 直接调 MCP。

## 公共约定

1. **不编造**：上游 null / 缺行 → 下游 null + `missing[]`。
2. **元数据**：每次成功 fetch 必须带 `source`、`fetched_at`、`timezone`。
3. **错误类型**：`AuthError` / `TimeoutError` / `UpstreamUnavailable` / `ValidationError`。
4. **无密钥**：适配器代码只读环境变量名；示例用 `<REDACTED>`。
5. **幂等写**：`write` 必须接受 `source_id`；重复提交不双写。

## WhoopAdapter

- 输入：工具名 + 参数（days/sport…）
- 输出：归一化 `MetricSample[]` 或 overview DTO
- 禁止：在适配器内做诊断结论或训练处方
- 连接器逻辑名：`whoop-personal-vps`

## NotionLedgerAdapter

- 读：按日期窗口查询 nutrition / training / body
- 写：仅在 Owner 确认后的草稿上调用
- 记忆页：只更新偏好/规则段落，不写时间序列度量

## ManualIngestAdapter

### 口述
- ASR/对话文本 → 结构化草稿 → 确认 → 写库

### 餐照
1. 原图保留为估算依据（本地或 Notion 附件策略由 ops 定；Public 不存真人餐照）
2. AI 生成桌面 **白底 Ins 风** 展示图
3. 营养字段标记 `数据来源=估算`，confidence 偏低时触发追问

## AppleHealthAdapter（v2 占位）

- 输入：导出 XML/桥接 API（待定）
- 映射到同一 `MetricSample` 契约
- 冲突优先级见 `docs/DATA-SOURCES.md`

## 契约测试（v1）

- 合成 JSON 夹具 → 归一化快照测试
- 非法值（负心率、缺时间戳）→ `ValidationError`
- 超时模拟 → 编排层降级文案（适配器抛 `TimeoutError`）
