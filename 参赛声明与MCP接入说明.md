# 参赛声明与 MCP 接入说明

## 一、参赛声明

- **参赛作品**：麦麦今日剧本 · 心情点餐剧场（麦当劳 × WorkBuddy 创意开发大赛）
- **赛事**：2026 麦当劳程序员创意开发大赛
- **作者 / 仓库**：CodeGanHaoZ（GitHub: https://github.com/CodeGanHaoZ/McDonald-Pay）
- **开发工具**：WorkBuddy（AI 原生开发环境），完整开发对话已按赛事要求以 MD 文档格式留存。
- **能力来源**：基于麦当劳官方 MCP Server（`mcp.mcd.cn`，v1.0.9）提供的真实菜单、营养、门店、算价等能力开发，非模拟数据。
- **声明**：本作品仅用于赛事演示，所有点餐/抽奖动作均为只读或二次确认后执行，未产生真实交易；价格、营养数据均来自官方 MCP 实时返回。

## 二、MCP 接入说明

### 1. 连接器配置（WorkBuddy `~/.workbuddy/mcp.json`）

```json
{
  "mcpServers": {
    "mcd-order": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer <你的 MCP Token>"
      }
    }
  }
}
```

> Token 获取：登录 https://open.mcd.cn/mcp/doc → 右上角「控制台」→「激活」→ 复制 Token。
> 配置后需在 WorkBuddy 连接器管理页「信任」`mcd-order` 并重启会话。

### 2. 本作品实际调用的工具

| 工具 | 用途 |
|------|------|
| `query-nearby-stores` | 定位门店（广州海珠广场餐厅，storeCode `1400067`） |
| `query-meals` | 拉取该门店真实菜单（130+ 商品，含现价/原价/图片） |
| `list-nutrition-foods` | 拉取真实营养数据（热量/蛋白/脂肪/碳水等） |
| `calculate-price` | 官方算价核验（含取餐方式） |

### 3. 安全说明

涉及资源消耗的动作（如 `draw-lottery` 抽奖消耗积分）均设计为**展示消耗规则 → 用户二次确认 → 再调用**，绝不自动执行，符合官方 MCP 使用规范。

## 三、运行方式

直接用浏览器打开 `maimai-combo.html` 即可体验完整交互（心情选择 → 3 题测验 → 大转盘揭晓 → 一键端盘 → 微调 → 出戏小票）。配合 WorkBuddy + 已信任的麦当劳 MCP，可由 Skill（`SKILL.md`）驱动真实数据。
