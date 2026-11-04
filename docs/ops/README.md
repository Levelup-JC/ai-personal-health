# Ops · 运维指针（Public）

本目录 **故意不含** VPS 部署手册、主机地址、隧道 URL、Bearer 或 ntfy 实参。

## 仓外私册应覆盖

1. 自建 `whoop-mcp` 部署与升级
2. Cloudflare Quick Tunnel / 固定隧道与 `public_url` 核对步骤
3. Grok Bot 连接器 `whoop-personal-vps` 的 URL/鉴权配置位置
4. Notion integration 与各库 id 映射
5. ntfy 基址、topic、鉴权（**推送正文禁止健康数值**）
6. 密钥轮换 runbook
7. 例程「每晚健康报表」的启用/暂停操作（须 Owner 拍板后再动）

建议私有仓或加密笔记命名：`ai-personal-health-ops`（由 Owner 决定）。

## 本仓依赖清单（逻辑）

| 依赖 | 用途 |
| --- | --- |
| Grok Bot + MCP 连接器 | 运行时编排 |
| Self-hosted whoop-mcp | WHOOP 只读工具 |
| Notion | 台账正本 |
| ntfy (self-hosted) | 叫醒/报表通知 |
| 可选 Magpie / LLM 网关 | 文档评审或离线 worker |

## 明确不做

- 不在 Public Issue/PR 粘贴隧道 URL 或 token
- 不在本仓 resume 任何已暂停例程
- 不修改现网 ntfy 配置
