# workbuddy.md —— 用 WorkBuddy 从 0 到能跑

项目：麦麦周周领（mcd-coupon-keeper）
参赛：1024 程序员节 · 麦当劳程序员创意开发大赛
开发方式：WorkBuddy（阿布节点）本地开发，全程工具操作，未使用其他 IDE 生成代码

## 1. 起点：先摸清接口，不猜

赛制要求基于麦当劳官方 MCP 开发。第一步没有直接写代码，而是**先把 MCP 的真实工具清单拉下来**——用 `initialize` + `tools/list` 打到 `https://mcp.mcd.cn`，拿到 **35 个工具**的完整名称、描述、入参 schema（存为 `tools/mcd_tools.json`）。

这一步的价值：后面所有 Skill 指令都基于**实测 schema**，而不是文档猜测。例如实测发现 `query-my-coupons` 的 `page`/`pageSize` 都是**字符串**类型（不是数字），`campaign-calendar` 的 `specifiedDate` 可空——这类细节只能实测拿到。

## 2. 选题：砍到最小可交付

35 个工具里，券相关的能力最能「立等可用」：领券、看券、看活动、看积分，全是只读或幂等操作，**没有支付风险**。于是选定 5 个工具拼一个 Skill，不做点餐、不做下单。

决策原则：**小而可靠 > 功能多**。第一版只解决「我该领什么、我有什么、哪张先吃」。

## 3. 实测：先跑通，再写文档

在写 Skill 之前，先逐个真实调用验证：

- `query-my-account` → 返回可用积分、累计积分、将过期积分（确认 Token 绑定的是目标账号）
- `available-coupons` → 返回可领券列表，含「已领取 / 可领取」状态
- `tools/list` → 35 个工具全量 schema 落盘

有了真实返回样本，Skill 里的播报模板才敢写死字段名——避免「文档写得漂亮、跑起来对不上」的常见坑。

## 4. 结构：一个 Skill，三层约束

```
SKILL.md        触发条件 + 5 个工具 + 标准流程 + 周报模板 + 6 条硬边界
README.md       给人看的：解决什么、怎么装、安全边界
mcp.example.json 配置样例（Token 用占位符）
.gitignore      排除 .secrets / *.token / *.env
```

关键的 6 条硬边界不是凑数，每条都对应真实风险：不代付、不承诺门店可用性、Token 不进仓库、查不到就说查不到、限频、只读优先。

## 5. 留痕与可复现

- 接口探测脚本、调用脚本都留在 `tools/`（不含 Token）
- Token 只从环境变量 / `.secrets/` 读，仓库里零凭据
- 任何人都能照 README 用**自己的** Token 复现

## 6. 踩过的坑

| 坑 | 现象 | 处理 |
|---|---|---|
| MCP 地址 | WorkBuddy 弹窗预填地址连不上 | 官方地址是 `https://mcp.mcd.cn`，手动改 |
| 返回体是 SSE | 响应是 `data: {...}` 而非纯 JSON | 解析器兼容两种格式 |
| 参数类型 | `pageSize` 传数字被拒 | 实测 schema 是 string，按字符串传 |
| 卡包语义 | 卡包有券就以为能用 | `query-my-coupons` 不做门店校验，必须写进边界 |

## 7. 下一步（不在第一版）

- 接入 `query-store-coupons`，把「手上有券」升级成「这家店这张能用」
- 按历史订单算「最省组合」，给出点单建议（仍需人工付款）
