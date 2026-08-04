# Research / 产品统筹工作板

## 你的唯一目标

让 Bonded Agent 的产品承诺在四个地方保持一致：合约、前端、README 和 Demo。你的工作不是替 Dev 写合约，而是定义什么必须被证明、什么绝不能被夸大。

## 每日节奏

### Day 1：冻结规则

- 与 Dev 1 对照 [产品规则](product-spec.md)，确认 `guaranteedOutput`、`maxCompensation`、`failureCompensation`、`deadline` 的准确语义。
- 与 Dev 2 修复金额最小单位展示，并冻结演示/真实状态的 badge 文案。
- 在群里确认：低于 `coverageFloor` 时采用安全失败、退款与失败补偿，不执行无法覆盖的 Swap。

### Day 2：把规则变成验收项

- 用 [界面状态文案](interface-state-copy.md) 逐屏检查首页、报价、计划、结算与历史页。
- 用 [验证方案](validation-and-user-test.md) 给 Dev 1 六个测试场景，给 Dev 2 七种状态。
- 收集需要写进 README 的真实/模拟边界。

### Day 3：证据联调

- 向 Dev 1 收集地址、ABI、事件定义和三类 Testnet 交易 Hash。
- 向 Dev 2 检查 Hash、Explorer 链接和状态是否来自真实事件。
- 填写链上证据登记表；没有证据的条目保持“待验证”。

### Day 4：用户测试与修改

- 找 3 名非团队成员，按用户测试方案观察，不先解释产品。
- 汇总误解，选出对理解影响最大的 3 项修改。
- 让设计和 Dev 2 只修这 3 项；不要在 P0 未稳定时新增功能。

### Day 5：提交材料与 Pitch

- 核验 README、截图、Hash、Explorer 链接与视频中的陈述一致。
- 准备评委问题：为什么不是 `minOutput`？为什么需要保证金？MockDex 为什么仍有价值？Moss 现在接入了吗？
- 在所有材料中保持同一句边界：保证最低到账，不保证盈利；超出保障范围时安全失败；当前是 Testnet + MockDex。

## 每天向 Dev 同步的三句话

1. 今天要验证的状态是什么？需要哪个事件或交易 Hash 证明？
2. 前端此刻显示的句子，合约真的已经做到了吗？
3. 哪个未完成部分必须标成演示或 Mock？

## 交接格式

### 从 Dev 1 到 Research

```text
合约地址：
函数 / 事件：
测试场景：
交易 Hash：
已知限制：
```

### 从 Research 到 Dev 2 / 设计

```text
页面状态：
链上事实：
用户可读文案：
必须显示的数字：
演示或真实：
验收条件：
```

## 你可以直接发到群里的今日消息

> 我已经把 Bonded Agent 的 P0 规则、状态文案、链上证据表和用户测试方案放进仓库 docs。今天请 Dev 1 确认保障范围 / 退款逻辑 / 事件字段，Dev 2 修复金额最小单位展示并区分“演示计划”和“真实上链计划”。我会按文档验收，不会把 Mock 数据写成真实完成。
