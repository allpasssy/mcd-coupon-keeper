---
name: mcd-weekly-coupon
description: 麦麦周周领 —— 一键领齐麦当劳优惠券、看清券包快过期、盘出本周能省多少。基于麦当劳官方 MCP（mcp.mcd.cn）只读工具拼装，全程不碰支付。当用户问「有什么券可以领」「帮我领券」「我有哪些券」「这周能省多少」「最近有什么活动」时使用。
---

# 麦麦周周领（mcd-weekly-coupon）

一句话：**帮你把麦当劳能省的券全领了，把快到期的先吃了，把本周活动算成钱。**

## 何时用

- 「帮我领券 / 一键领券 / 有什么羊毛」
- 「我卡包里有什么券 / 哪些快过期」
- 「这周麦当劳有什么活动 / 能省多少」
- 「我还有多少麦享会积分」

## 依赖的 MCP 工具（麦当劳官方 mcp.mcd.cn，实测 35 个工具中取 5 个）

| 工具 | 作用 | 入参 |
|---|---|---|
| `available-coupons` | 麦麦省当前**可领取**的券列表 | 无 |
| `auto-bind-coupons` | **一键领取**全部可领券（幂等，重复调用无副作用） | 无 |
| `query-my-coupons` | 卡包内**已拥有**的券（不含门店可用性校验） | `page`、`pageSize`（都是字符串，页最多 5、每页最多 200） |
| `query-my-account` | 麦享会积分余额/累计/将过期 | 无 |
| `campaign-calendar` | 当月营销活动日历（进行中/往期/未来） | `specifiedDate`（yyyy-MM-dd，可选） |

## 标准流程

1. **先看**：`available-coupons` 列出可领券，`query-my-coupons` 列出已有券 —— 两个都调，才能区分「没领」和「已领」。
2. **再领**：用户确认后调 `auto-bind-coupons`（一次即可，勿循环）。返回里按「本次领取 / 已在卡包」分组播报。
3. **后盘**：`query-my-coupons` 复查卡包，按到期时间排序；`query-my-account` 取积分余额与将过期积分；`campaign-calendar` 取本月活动。
4. **出账**：按下面的「周报模板」输出，最后给一句行动建议。

## 周报模板（默认输出）

```
🍟 麦麦周周领 · {YYYY-MM-DD}

【本次新领】N 张
· {券名} —— {面额/条件}
（无新增则写「无新增，卡包已领齐」）

【卡包 N 张，按到期临近】
1. {券名}｜{条件}｜{到期日}（剩 X 天）
2. …

【积分】可用 {availablePoint}，累计 {accumulativePoint}，本月将过期 {currentMouthExpirePoint}

【本月活动】{进行中的活动名 + 时间}

【一句建议】挑 1-2 张最划算的先用掉，别等过期。
```

## 硬边界（不可越过）

1. **只读优先**：除 `auto-bind-coupons`（领券，无成本）外，**不下单、不支付、不核销**。
2. **支付永远人工**：任何 `create-order` / `mall-create-order` 类工具只在用户明确要求时调用，且必须提醒「下单后需本人扫码付款，Agent 只建单」。
3. **不承诺可用性**：`query-my-coupons` 只说明「你拥有」，不保证「当前门店可用」；要判门店可用性必须走 `query-store-coupons`。播报时必须写明这一点。
4. **Token 不进仓库**：KEY 只从本机 `.secrets/mcd.json` 或环境变量 `MCD_MCP_TOKEN` 读取，严禁写入仓库、Issue、日志、截图。
5. **查不到就说查不到**：MCP 返回空或报错时如实说明，禁止编造券名、面额、到期日。
6. **限频**：Token 限 600 次/分钟，单次会话调用不超过 10 次。

## 配置

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": { "Authorization": "Bearer ${MCD_MCP_TOKEN}" }
    }
  }
}
```

> WorkBuddy 弹窗里的默认 MCP 地址是错的，以上面这个为准。
