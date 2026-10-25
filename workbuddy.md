# WorkBuddy 开发对话上下文（workbuddy.md）

> 本文件用于核验本作品是否使用 WorkBuddy 开发、是否符合「WorkBuddy 专项奖励」条件。
> 开发全程在 WorkBuddy（AI 原生开发环境，Agent 模式）中完成，完整对话已留存于本会话。以下为开发脉络摘要。

## 开发环境

- **工具**：WorkBuddy（对话式 AI 开发，Agent 模式，可直接读写文件、运行命令、调 MCP）
- **关键能力**：
  - 接入麦当劳官方 MCP（`mcd-order` 连接器，endpoint `https://mcp.mcd.cn`，Bearer Token 由用户提供并写入 `~/.workbuddy/mcp.json`）
  - 调用 `query-nearby-stores` / `query-meals` / `list-nutrition-foods` / `calculate-price` 拉取真实数据
  - 用 Chrome 无头截图 + 视觉校验循环迭代 UI（顶栏/转盘/剧本栅格/任务进度条/小票印章/响应式破版修复）

## 开发脉络（关键节点）

1. **调研与方案**：读取官方 MCP 仓库（github.com/M-China/mcd-mcp-server，v1.0.9，37 工具 / 9 场景），产出 4 个开发方案，主推「麦省宝」。
2. **强交互转向**：用户反馈原方案偏问答、易撞车，改为强交互专题（扭蛋机 / 套餐工坊 / 连连看 / 配送追踪 / 派对策划台）。
3. **接入 MCP**：用户提供 Token，写入连接器并信任；验证 `query-lottery-info`（麦麦积分抽奖 24 积分/次）、`query-my-prizes`、`list-nutrition-foods`、`query-nearby-stores`、`query-meals` 全部打通。
4. **UI 风格迭代**：
   - 初版套餐工坊被批"太 AI 味、无品牌特色、像通用网页"；
   - 改为"借鉴麦当劳设计范式（红黄撞色 / 金拱门 M / 圆角卡 / 拟人食物表情 / 贴纸），但做成正常**网页**而非仿微信小程序手机壳"；
   - 再做"强交互 + 叙事"转向：用**心情选择 → 3 题测验 → 大转盘**驱动点餐，而非自由拖菜单。
5. **破版修复**：用户截图反馈横向溢出、贴纸飞出卡片、角色气泡遮挡侧栏。定位为响应式缺失（仅 880px 断点、固定 380px 侧栏、长店名胶囊不换行、mascot 固定宽遮挡），已通过 `flex-wrap`、断点下移到 980/760/560、`minmax(0,1fr)` 侧栏、贴纸/角标收进卡内、窄屏隐藏气泡等方式修复，并在 480px 与 1100px 双宽度截图验证。

## 交付物

- `maimai-combo.html`：主交付，麦麦今日剧本·心情点餐剧场（单文件，浏览器直接打开）
- `SKILL.md`：WorkBuddy Skill 编排（含 `draw-lottery` 二次确认安全闸）
- `maimai-gachapon.html` / `maimai-gachapon-real.html`：早期备选原型（扭蛋机）
- `README.md` / `CONTEST_DECLARATION.md` / `MCP_INTEGRATION.md`：参赛必备文件

## 合规与安全

- 仓库不含任何真实 Token / 密钥 / 手机号；MCP 认证信息仅留在本地 `~/.workbuddy/mcp.json`，未入库。
- 涉及资源消耗的动作（抽奖）均设计二次确认，绝不自动执行。
- 价格、营养数据均来自官方 MCP 实时返回，前端仅做展示与本地累加（并可用 `calculate-price` 兜底核验）。
