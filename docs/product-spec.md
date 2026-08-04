# Bonded Agent P0 产品规则与合约验收规范

**状态：** 已冻结，作为合约、前端、README 和 Demo 的共同事实来源。  
**负责人：** Research / 产品统筹。  
**版本：** v0.1，2026-08-05。

## 1. P0 的唯一承诺

Bonded Agent 不保证利润，也不保证 Agent 在任何任务中正确。它只对一笔特定的 Monad Testnet Swap 作出一个可测量承诺：在保障范围内，用户最终收到的 `tUSDC` 不低于计划中写明的保证输出。

P0 固定场景：`MON -> tUSDC`。只支持一个团队控制的 Agent 运营方和一个测试市场组件 `MockDex`。

## 2. 计划字段与含义

| 字段 | 含义 | 前端要求 |
| --- | --- | --- |
| `inputAmount` | 用户投入的 MON 数量 | 人类可读格式，例如 `1 MON` |
| `expectedOutput` | 报价时模拟得到的输出 | 标记为“预期”，不是保证 |
| `guaranteedOutput` | 保障范围内用户最终至少收到的 tUSDC | 高亮展示 |
| `maxCompensation` | 运营方为本计划承担的最大差额责任 | 必须与锁定保证金一致或更小 |
| `failureCompensation` | 无法在保障范围内执行时的固定补偿 | 单独展示 |
| `deadline` | 用户可执行该计划的截止时间 | 显示倒计时与绝对时间 |
| `target` | 允许调用的目标合约 | 在详情/开发者信息中可查看 |
| `calldataHash` | 绑定的调用数据哈希 | 不在主卡片占空间，但必须可验证 |
| `nonce` | 防止计划重复执行的编号 | 不在主卡片占空间，但必须可验证 |

所有金额在合约内使用最小单位；前端必须读取对应代币的 `decimals()` 后格式化。不得把 `950000000000000000` 直接显示给用户，也不得假定所有代币都是 18 位小数。

## 3. 样例计划与计算规则

当前演示计划使用：

```text
inputAmount          = 1 MON
expectedOutput       = 0.9500 tUSDC
guaranteedOutput     = 0.9405 tUSDC
maxCompensation      = 0.1000 tUSDC
failureCompensation  = 0.0200 tUSDC
```

定义：

```text
coverageFloor = guaranteedOutput - maxCompensation
              = 0.8405 tUSDC
```

| 实际输出 `actualOutput` | 结算结果 | 用户可读状态 |
| --- | --- | --- |
| `actualOutput >= guaranteedOutput` | 不赔付，释放保证金 | 正常履约 |
| `coverageFloor <= actualOutput < guaranteedOutput` | 赔付 `guaranteedOutput - actualOutput` | 已自动补足差额 |
| `actualOutput < coverageFloor` | 不允许以不足额结果完成；安全失败、退回输入，并支付 `failureCompensation` | 未在可保障范围内完成，已退款并补偿 |
| 目标调用失败 | 安全失败、退回输入，并支付 `failureCompensation` | Swap 调用失败，已退款并补偿 |

这条规则解决一个必要约束：当最大赔付只有 `0.1 tUSDC` 时，产品不能无条件声称“保证 0.9405 tUSDC”。超出保障范围时必须走安全失败路径，而不是完成一笔无法补足的 Swap。

## 4. 合约不可违反的条件

- 没有足额保证金，不能创建或激活计划。
- 计划必须绑定用户、目标合约、calldata 哈希、截止时间和 nonce。
- 非指定用户、过期计划、已使用计划、被替换的 calldata 都不能执行。
- 赔付金额不得大于 `maxCompensation`，累计结算不得超过锁定保证金。
- `actualOutput` 必须来自用户输出代币余额变化，不能由前端、LLM 或管理员提交。
- 外部 Swap 调用失败时，退款与失败补偿必须仍可结算，不能被整体回滚吞掉。
- 任何人都不能重复领取同一笔赔付、退款或保证金释放。
- `MockDex` 的可配置汇率是模拟市场能力，必须在产品中明确披露。

## 5. 需要产生的链上事件

| 事件 | 最少包含的信息 | 前端用途 |
| --- | --- | --- |
| `PlanOpened` | plan id、operator、user、保证输出、保证金、deadline | 显示“保证金已锁定” |
| `PlanExecuted` | plan id、实际输出、执行交易 | 显示正常履约 |
| `ShortfallPaid` | plan id、实际输出、赔付金额 | 显示自动补足 |
| `PlanFailed` | plan id、失败原因类别、退款、失败补偿 | 显示安全失败 |
| `BondReleased` | plan id、返还给运营方的保证金 | 显示结算完成 |

在 P0 尚未接入真实事件前，界面必须使用 `演示计划 / 未上链`，不得显示“运营方已锁定保证金”。

## 6. Dev 验收测试矩阵

| Case | MockDex 输出 | 预期事件 | 预期用户结果 |
| --- | --- | --- | --- |
| 正常履约 | `0.9500` | `PlanExecuted`、`BondReleased` | 收到 `0.9500 tUSDC`，无赔付 |
| 自动赔付 | `0.9000` | `PlanExecuted`、`ShortfallPaid`、`BondReleased` | 收到 `0.9405 tUSDC`，赔付 `0.0405` |
| 边界赔付 | `0.8405` | `PlanExecuted`、`ShortfallPaid` | 收到 `0.9405 tUSDC`，赔付 `0.1000` |
| 超出保障范围 | `0.8000` | `PlanFailed` | MON 退回，收到 `0.0200 tUSDC` 失败补偿 |
| 过期计划 | 任意 | 无执行事件 | 拒绝执行 |
| 重放 / 改 calldata / 非指定用户 | 任意 | 无执行事件 | 拒绝执行 |

## 7. 联调完成定义

P0 只有同时满足以下条件才可称为“真实 Testnet Demo”：

- 上表中正常履约、自动赔付和安全失败至少各有一组 Monad Testnet 交易 Hash。
- 前端从合约事件读取状态和数值，不使用硬编码成功页。
- 每一个状态都能打开对应区块浏览器链接。
- README 和页面都清晰区分 Testnet 合约、MockDex 和尚未完成的 Moss / 真实 DEX 集成。
