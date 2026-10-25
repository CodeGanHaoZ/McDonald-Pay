# mcd-mood-diner

麦麦心情点餐剧场：基于麦当劳中国官方 MCP **真实菜单与营养数据**的「心情驱动点餐」创意 Skill。
把"今天吃什么"变成 **选心情 → 答 3 题小测验 → 转大转盘揭晓 → 一键端盘** 的强交互叙事体验；所有推荐均来自 `query-meals` 真实菜单，价格经 `calculate-price` 官方核验。

> 本 Skill 即大赛示例「麦麦点餐助手」的游戏化创意变体，完全符合主题「基于麦当劳 MCP 能力开发你的创意 Skill」。

## When to use

当用户提出以下需求时使用：

- “今天不知道吃啥 / 帮我点个餐 / 麦麦点餐剧场 / 心情点餐”
- “根据我的心情推荐麦当劳 / 我想玩着点餐”
- “帮我凑个健身餐 / 合家欢套餐 / 控卡点单”

## Preconditions

1. WorkBuddy 已配置麦当劳 MCP Server（连接器名 `mcd-order`，URL `https://mcp.mcd.cn`，Streamable HTTP + `Authorization: Bearer <用户Token>`）。
2. 用户已在连接器管理页「信任」`mcd-order` 并重启/刷新会话，会话可见 `query-nearby-stores` / `query-meals` / `list-nutrition-foods` / `calculate-price`。
3. 本 Skill **仅做查询与算价推荐，绝不自动下单 / 扣费**；若用户要求真实下单，转麦乐送下单流程并执行二次确认。

## Workflow

1. **定位门店**：调用 `query-nearby-stores`（searchType=2 + 用户城市/位置）→ 取 `storeCode`。
2. **拉真实菜单**：调用 `query-meals`（storeCode、orderType=1、beType=1）→ 商品名 / 价格 / 图片。
3. **拉营养**：调用 `list-nutrition-foods` → 热量 / 蛋白 / 脂肪 / 碳水，按名称匹配到菜单项。
4. **互动引导**：在对话中请用户选心情（累瘫了 / 饿狠了 / 有点丧 / 嗨起来 / 嘴馋了 / 想暖点）+ 答 3 题（跟谁吃 / 钱包状态 / 热量管控）；或调起 `maimai-combo.html` 让用户直接在前端玩。
5. **生成推荐**：按心情偏向 + 测验答案，从真实菜单挑出套餐（独食 / 双人 / 合家欢 / 开黑、省着点 / 犒劳自己、轻负担 / 硬核健身），计算小计。
6. **官方算价核验**：调用 `calculate-price`（items）→ 得到官方总价与取餐方式，回填页面。
7. **交付**：生成 / 打开「麦麦心情点餐剧场」交互页（注入真实数据），或在对话直接给出推荐 + 算价 + 营养小结。

## Capability workflows

- `query-nearby-stores`：定位取餐门店，缺省用用户所在城市就近门店。
- `query-meals` / `list-nutrition-foods`：取数基础；任一项缺失则如实告知用户并降级使用内联缓存数据。
- `calculate-price`：推荐后必调，保证页面总价与官方一致，并返回可取餐方式（堂食 / 外带）。

## Output formats

- 推荐套餐卡：场景叙事 + 人设标签 + 商品清单 + 立省金额 + 总热量 / 蛋白 + 取餐方式。
- 可视化：`maimai-combo.html`（心情选择 / 测验 / 大转盘 / 拼盘 / 小票），由 Agent 注入真实数据驱动。

## Safety rules

- 仅查询与算价，绝不自动下单 / 扣费；涉及真实交易必须二次确认。
- 不在对话中展示、记录、复述用户 Token、手机号等敏感信息。
- 营养数据以 `list-nutrition-foods` 为准；套餐类返回 `null` 时如实标注「暂无营养数据」，不臆造。

## Pitfalls

- `calculate-price` 仅算价不下单，切勿与下单接口混淆。
- 推荐必须来自真实菜单，不得编造不存在的商品或价格。

## Verification

使用前确认：

1. 会话可见 `query-nearby-stores` / `query-meals` / `list-nutrition-foods` / `calculate-price`。
2. 推荐均来自真实菜单，价格经 `calculate-price` 核验。
3. 未触发任何下单 / 扣费动作。

## 可视化契约（maimai-combo.html）

HTML 为「交互可视化层」，由 Agent 将真实数据注入 `window.__MCD_MOOD__` 驱动（真实模式）；留空则用内联缓存数据（演示模式）。浏览器不直接调用 MCP（无法持 Token）。
