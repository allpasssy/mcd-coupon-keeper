# 麦麦周周领 · mcd-coupon-keeper

> 麦当劳优惠券的「领、盘、省」三件事，一句话办完。基于麦当劳官方 MCP 的只读工具，**不碰支付**。

## 它解决什么

麦当劳 App 里的券散布在三处：麦麦省可领的、卡包已领的、活动页藏着的。要省钱得来回翻三个页面，还要自己算哪张快过期。这个 Skill 把它们收成一份「周报」：**新领了什么、手里有什么、哪张先吃、本月还有什么活动**。

## 用起来

对 WorkBuddy 说：

- 「帮我领麦当劳的券」→ 列可领 + 一键领取 + 给卡包总览
- 「我有哪些券快过期了」→ 卡包按到期排序
- 「这周麦当劳有什么活动」→ 本月活动日历
- 「我麦享会还有多少分」→ 积分余额与将过期积分

## 装法

1. 到 [open.mcd.cn/mcp](https://open.mcd.cn/mcp) 用手机号登录，激活后拿 MCP Token（免费）。
2. 配 MCP（`mcp.example.json` 可参考）：

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": { "Authorization": "Bearer <你的Token>" }
    }
  }
}
```

3. 把 `SKILL.md` 放进 WorkBuddy 的 skills 目录（项目级 `.workbuddy/skills/` 或用户级 `~/.workbuddy/skills/`）。
4. 对 WorkBuddy 说「帮我领券」，看它按模板出周报。

> ⚠️ WorkBuddy 添加 MCP 时弹窗预填的地址不是 `https://mcp.mcd.cn`，请手动改成上面这个。

## 用到的工具（5 个，全部来自官方 MCP）

`available-coupons` · `auto-bind-coupons` · `query-my-coupons` · `query-my-account` · `campaign-calendar`

## 安全边界

- **不代付**：Skill 只建单不付款，付款永远本人扫码。
- **不吹可用性**：卡包有券 ≠ 当前门店能用；要门店可用性得走 `query-store-coupons`。
- **Token 不进仓库**：用环境变量 `MCD_MCP_TOKEN`，`.gitignore` 已排除 `.secrets/`、`*.token`、`*.env`。

## 开发留痕

见 [workbuddy.md](workbuddy.md) —— 记录了这个 Skill 是怎么用 WorkBuddy 从 0 做出来的（赛制要求提交）。

## License

MIT
