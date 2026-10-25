# mcd-gachapon

麦麦扭蛋机：基于麦当劳中国 MCP 积分抽奖能力的游戏化助手。将 `draw-lottery` / `query-lottery-info` / `query-my-prizes` 包装成「转蛋 → 开盖 → 图鉴收集」的可视化体验，同时持有严格的积分消耗二次确认安全闸。

## When to use

当用户提出以下需求时使用：

- “帮我抽个奖 / 扭一下 / 试试手气 / 用积分抽个奖”
- “麦当劳积分抽奖有什么奖品 / 现在有什么抽奖活动”
- “查我抽到了什么 / 我的奖品 / 抽奖记录”
- “麦麦扭蛋机 / 扭蛋”

## Preconditions

1. WorkBuddy 已配置麦当劳 MCP Server（连接器名 `mcd-order`，URL `https://mcp.mcd.cn`，Streamable HTTP + `Authorization: Bearer <用户Token>`）。
2. 用户已在连接器管理页「信任」`mcd-order` 并重启/刷新会话，且会话可见 `mcp__mcd-order__query-lottery-info` 等工具。
3. 抽奖为**真实消耗积分**操作，必须获得用户明确二次确认后方可调用 `draw-lottery`。

## Workflow

1. **查活动**：调用 `query-lottery-info`，获取活动状态、奖品列表、抽奖消耗规则（每次消耗积分数/可用次数）、用户可用资源（剩余积分、剩余次数）。
2. **展示与确认**：向用户清晰展示：
   - 活动是否进行中、起止时间
   - 奖品池概览（名称、等级、是否还有库存/概率说明）
   - **本次抽奖将消耗的积分数与剩余资源**，并明确提示“抽奖将真实扣除积分，不可撤销”
   - 等待用户明确确认“抽 / 扭一次 / 确认抽奖”后再继续
3. **执行抽奖**：用户确认后调用 `draw-lottery`，返回中奖奖品信息。
4. **展示结果**：以扭蛋卡片展示中奖奖品（名称、等级、描述），并维护「奖品图鉴」（按奖品聚合计数）与「抽奖记录」（时间、等级、奖品）。
5. **事后可查**：用户说“我抽到了什么 / 历史”时调用 `query-my-prizes` 展示全部奖品记录（分页）。

## Capability workflows

- `query-lottery-info`：抽奖前必调，用于判断活动有效性、展示奖品池与消耗规则，避免向用户承诺不确定的奖品。
- `draw-lottery`：仅在高危二次确认后调用。若接口返回次数/积分不足、活动未开始或已结束，停止并如实说明。
- `query-my-prizes`：查询用户历史奖品，按中奖时间倒序展示，支持分页说明。

## Output formats

- 活动信息：展示活动名称、状态（进行中/未开始/已结束）、时间、奖品池、消耗规则、用户剩余资源。
- 抽奖结果：扭蛋卡片（奖品名 + 等级标签 + 描述）+ 图鉴（集齐度）+ 记录列表。
- 等级配色（通过 Design Token 引用，禁止硬编码）：普通=中性灰，稀有=绿，史诗=红。

## Safety rules

- 抽奖真实消耗积分，**必须二次确认**：先展示消耗数量与规则，再等待用户明确确认，绝不自动连抽或默认抽。
- 不在对话中展示、记录、复述用户 Token、手机号等敏感信息。
- 若工具返回 Token 失效/401/403，提示用户重新获取 MCP Token 并按初始化流程更新、信任、刷新。
- 若返回 429 限流，提示稍后重试，不循环重试。
- 接口返回字段缺失时，明确说明“接口未返回该信息”，不补编奖品或概率。

## Pitfalls

- `draw-lottery` 是外部账户扣积分动作，未二次确认即调用视为严重流程缺陷。
- 奖品概率/库存以 `query-lottery-info` 返回为准，不臆测“必中”。
- 当前 MCP 未暴露“撤销抽奖/退回积分”工具，抽奖前务必确认用户已知不可撤销。

## Verification

使用前确认：

1. 会话可见 `mcp__mcd-order__query-lottery-info` / `draw-lottery` / `query-my-prizes`。
2. 抽奖前已向用户展示消耗规则并完成二次确认。
3. 中奖结果、图鉴、记录均基于工具真实返回，未编造。

## 可视化契约（maimai-gachapon.html）

HTML 为「结果可视化层」，由 Agent 注入数据驱动，浏览器不直接调用 MCP（无法持 Token）。

- 注入对象：`window.__MCD_GACHA__`（真实模式）或留空（演示模式用本地 mock 奖品池）。
- 结构：
  ```json
  {
    "real": true,
    "activity": { "name": "积分抽奖", "status": "进行中" },
    "cost": { "points": 10, "remainingPoints": 1280, "remainingTimes": 9 },
    "prizes": [ { "id": "p1", "name": "巨无霸", "tier": "普通", "desc": "...", "color": "#C0392B" } ],
    "lastResult": { "id": "p1", "name": "巨无霸", "tier": "普通", "desc": "...", "color": "#C0392B" },
    "history": [ { "id": "p1", "name": "巨无霸", "tier": "普通", "time": "21:30:01" } ]
  }
  ```
- 真实模式下，用户点击「扭一次」不会本地抽奖，而是提示「请在对话框发送『抽一次』」，由 Skill 调 `draw-lottery` 并将 `lastResult` 注入后重新打开本页，播放开盖动画并刷新图鉴/记录。
- `prizes` 来源于 `query-lottery-info` 的奖品列表；`lastResult`/`history` 来源于 `draw-lottery`/`query-my-prizes`。
